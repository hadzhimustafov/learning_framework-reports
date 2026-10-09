# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-epic25-recovery.spec.ts >> skip optional question keeps the compiled plan and consents with its current digest
- Location: e2e/course-generation-epic25-recovery.spec.ts:442:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByRole('heading', { name: 'Ваш курс готов' })
Expected: visible
Timeout: 180000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 180000ms
  - waiting for getByRole('heading', { name: 'Ваш курс готов' })

```

```yaml
- banner:
  - button "Меню ученика"
- main:
  - button "В библиотеку"
  - list "Этапы создания":
    - listitem: ✓ Цель
    - listitem: ✓ Параметры
    - listitem: ✓ Программа
    - listitem: 4 Создание
  - paragraph: Team capacity planning
  - text: Создание Создание прервано. Готовые занятия сохранены
  - alert:
    - paragraph: Создание прервано. Готовые занятия сохранены
    - paragraph: Можно продолжить с незавершённого занятия. Подтверждать прежнюю программу повторно не нужно.
    - button "Продолжить создание"
    - button "Изменить программу"
  - region "Готовим ваш курс":
    - heading "Готовим ваш курс" [level=1]
    - paragraph: Создание остановлено
    - list:
      - listitem: Уточняем цель Готово
      - listitem: Составляем программу Готово
      - listitem: Готовим занятия выполняется …
      - listitem: Проверяем курс ждёт
      - listitem: Сохраняем курс ждёт
    - paragraph: Готово 0 из 2 занятий
    - progressbar "Готовые занятия"
    - paragraph: Foundations of Team capacity planning
    - paragraph: Прошло 0:02
    - paragraph: Можно вернуться в библиотеку. Создание продолжится, пока приложение работает
```

# Test source

```ts
  263 |     .toBe('drained');
  264 | }
  265 | 
  266 | // -- wizard driving -----------------------------------------------------------
  267 | 
  268 | /** Open the generator from the library and wait for the goal step. */
  269 | export async function openGenerator(page: Page): Promise<void> {
  270 |   await gotoLearnerApp(page);
  271 |   const create = page.getByRole('button', { name: 'Создать курс' });
  272 |   await expect(create).toBeVisible({ timeout: 30_000 });
  273 |   await create.click();
  274 |   await expect(page.getByRole('heading', { name: 'Чему хотите научиться?' })).toBeVisible({
  275 |     timeout: 30_000,
  276 |   });
  277 | }
  278 | 
  279 | /** Type the subject description and continue to the profile step. */
  280 | export async function submitGoal(page: Page, description: string): Promise<void> {
  281 |   await page.fill('#wizard-goal', description);
  282 |   await page.getByRole('button', { name: 'Продолжить' }).click();
  283 | }
  284 | 
  285 | /**
  286 |  * Wait until the extracted profile is complete and confirm it unchanged (the digest path).
  287 |  * A01: the extracted skill and outcome are shown for confirmation; nothing else is asked.
  288 |  *
  289 |  * LF-141 flip: the confirmation itself enqueues the `propose_plan_draft`, the worker runs
  290 |  * it and the post-draft gate builds the map deterministically, so after «Показать
  291 |  * программу» the wizard waits in the EPIC-25 draft program window
  292 |  * (`draft-version-badge`) until the program is compiled — the consent CTA is the NEW
  293 |  * step, not the confirm output.
  294 |  */
  295 | export async function confirmProfileStep(page: Page, skill: string): Promise<void> {
  296 |   const cta = page.getByRole('button', { name: 'Показать программу' });
  297 |   await expect(page.locator('#wizard-skill')).toHaveValue(skill, { timeout: 60_000 });
  298 |   const language = page.locator('#wizard-language');
  299 |   if ((await language.inputValue()) === '' && !(await cta.isEnabled())) {
  300 |     await language.selectOption({ label: 'Русский' });
  301 |   }
  302 |   await expect(cta).toBeEnabled({ timeout: 60_000 });
  303 |   await cta.click();
  304 |   // The flip-enqueued draft ran: its clarification cards render while the server builds
  305 |   // the current-draft map without a generation-provider request.
  306 |   await expect(page.getByTestId('draft-version-badge')).toBeVisible({ timeout: 120_000 });
  307 |   await waitForCalls(page, 'propose_plan_draft', 1);
  308 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  309 |     timeout: 120_000,
  310 |   });
  311 | }
  312 | 
  313 | /**
  314 |  * Answer ONE draft clarification card through the real card contract: select the first
  315 |  * option of the first open card and Apply it exactly once. The 202 flow re-runs the plan
  316 |  * draft (`propose_plan_draft` >= 2) and the gate rebuilds the map for the moved spec
  317 |  * without a provider call.
  318 |  */
  319 | export async function answerFirstDraftQuestion(page: Page): Promise<void> {
  320 |   // Wait for the settled plan window first: between the draft commit and the
  321 |   // compiled program the controller auto-compiles the gated map, and that
  322 |   // draft→compiled branch change REMOUNTS the question cards and clears any
  323 |   // local selection (LF-146 slice B). Interacting before the settle races the
  324 |   // remount; after the CTA is enabled the card state is stable.
  325 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  326 |     timeout: 120_000,
  327 |   });
  328 |   const card = page.locator('article[data-testid^="question-"]').first();
  329 |   await expect(card).toBeVisible({ timeout: 90_000 });
  330 |   await card
  331 |     .locator('label[data-testid^="question-option-"] input[type="radio"]')
  332 |     .first()
  333 |     .check();
  334 |   // The local dirty lock enables Apply only after the selection is registered.
  335 |   const apply = card.getByRole('button', { name: 'Применить ответ' });
  336 |   await expect(apply).toBeEnabled({ timeout: 30_000 });
  337 |   await apply.click();
  338 |   // The accepted answer re-runs the plan draft for the moved spec.
  339 |   await waitForCalls(page, 'propose_plan_draft', 2);
  340 | }
  341 | 
  342 | /** Wait for the compiled plan and confirm it once (the single consent point). */
  343 | export async function confirmPlanStep(page: Page): Promise<string> {
  344 |   const cta = page.getByRole('button', { name: 'Подтвердить и создать курс' });
  345 |   await expect(cta).toBeEnabled({ timeout: 120_000 });
  346 |   const [request] = await Promise.all([
  347 |     page.waitForRequest(
  348 |       (candidate) =>
  349 |         candidate.method() === 'POST' &&
  350 |         /\/api\/v1\/authoring\/sessions\/[^/]+\/plan-confirmation$/.test(
  351 |           new URL(candidate.url()).pathname,
  352 |         ),
  353 |     ),
  354 |     cta.click(),
  355 |   ]);
  356 |   const body = request.postDataJSON() as { blueprint_digest?: string };
  357 |   expect(typeof body.blueprint_digest, 'consent carries the compiled plan digest').toBe('string');
  358 |   return body.blueprint_digest as string;
  359 | }
  360 | 
  361 | /** Wait for the wizard to reach the saved-course screen. */
  362 | export async function waitForSavedCourse(page: Page): Promise<void> {
> 363 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
      |                                                                       ^ Error: expect(locator).toBeVisible() failed
  364 |     timeout: 180_000,
  365 |   });
  366 | }
  367 | 
  368 | /**
  369 |  * Wait for the saved-course heading after consent, failing on a visible recovery alert.
  370 |  *
  371 |  * Used when a 180s heading timeout would hide the actual failure: the worker call log
  372 |  * must first show materialization started, then the heading or an alert decides.
  373 |  */
  374 | export async function waitForSavedCourseOrAlert(page: Page, timeoutMs = 60_000): Promise<void> {
  375 |   const saved = page.getByRole('heading', { name: 'Ваш курс готов' });
  376 |   try {
  377 |     await expect(saved).toBeVisible({ timeout: timeoutMs });
  378 |   } catch (error) {
  379 |     const alerts = await page.getByRole('alert').allTextContents();
  380 |     const detail = alerts.map((text) => text.trim()).filter((text) => text.length > 0);
  381 |     throw new Error(
  382 |       `saved heading missing after ${timeoutMs}ms; alerts: ${detail.join(' | ') || 'none'}`,
  383 |       { cause: error },
  384 |     );
  385 |   }
  386 | }
  387 | 
  388 | /** Drive one full creation from the library to the saved screen; returns the course title. */
  389 | export async function driveCreation(
  390 |   page: Page,
  391 |   subject: 'python' | 'finance' | 'management',
  392 | ): Promise<string> {
  393 |   const title = await driveToConfirmedPlan(page, subject);
  394 |   await confirmPlanStep(page);
  395 |   await waitForSavedCourse(page);
  396 |   return title;
  397 | }
  398 | 
  399 | /** Drive one creation up to the confirmed plan (before materialization starts). */
  400 | export async function driveToConfirmedPlan(
  401 |   page: Page,
  402 |   subject: 'python' | 'finance' | 'management',
  403 | ): Promise<string> {
  404 |   const description = subjectDescription(subject);
  405 |   const title = subjectCourseTitle(subject);
  406 |   await openGenerator(page);
  407 |   await submitGoal(page, description);
  408 |   await confirmProfileStep(page, description.split('\n')[0]);
  409 |   return title;
  410 | }
  411 | 
  412 | /** Start the just-saved course and land in its workspace (trust granted when asked). */
  413 | export async function startSavedCourse(page: Page): Promise<void> {
  414 |   await page.getByRole('button', { name: 'Начать обучение' }).click();
  415 |   await grantTrustIfPresent(page);
  416 |   await waitForWorkspace(page);
  417 | }
  418 | 
  419 | // -- workspace driving --------------------------------------------------------
  420 | 
  421 | /** Open one curriculum unit by title prefix and order when source titles repeat. */
  422 | export async function openUnitByTitle(
  423 |   page: Page,
  424 |   titlePrefix: string,
  425 |   occurrence = 0,
  426 | ): Promise<void> {
  427 |   await page.getByRole('button', { name: new RegExp(`^${titlePrefix}`) }).nth(occurrence).click();
  428 | }
  429 | 
  430 | /** Switch to the Code tab of the active unit. */
  431 | export async function openCodeTab(page: Page): Promise<void> {
  432 |   await page.getByRole('tab', { name: 'Code' }).click();
  433 | }
  434 | 
  435 | /**
  436 |  * Replace the whole CodeMirror buffer by pasting. DOM-level typing (`keyboard.type`)
  437 |  * corrupts whitespace inside CodeMirror's contenteditable (newlines collapse into one
  438 |  * line, spaces become U+00A0), which saved REAL wrong code and produced genuine UNMET
  439 |  * verdicts for right answers (CG21-121 python journey flake). Dispatch a paste event
  440 |  * with plain text instead; the editor's own event path renders and autosaves it.
  441 |  */
  442 | export async function fillEditor(page: Page, content: string): Promise<void> {
  443 |   const editor = page.locator('.cm-content');
  444 |   await editor.click();
  445 |   await page.keyboard.press('ControlOrMeta+a');
  446 |   await editor.evaluate((element, text) => {
  447 |     const dataTransfer = new DataTransfer();
  448 |     dataTransfer.setData('text/plain', text);
  449 |     element.dispatchEvent(
  450 |       new ClipboardEvent('paste', { clipboardData: dataTransfer, bubbles: true, cancelable: true }),
  451 |     );
  452 |   }, content);
  453 | }
  454 | /** Run Check work and wait for a terminal rubric pill (passed or unmet). */
  455 | export async function checkWork(page: Page): Promise<'PASSED' | 'UNMET CRITERIA'> {
  456 |   const pill = page.getByText(/^(PASSED|UNMET CRITERIA)$/);
  457 |   const checking = page.getByRole('button', { name: 'Checking...' });
  458 |   await page.getByRole('button', { name: 'Check work' }).click();
  459 |   // A stale terminal pill from the PREVIOUS evaluation may still be mounted while the fresh
  460 |   // run executes (CG21-121 flake fix): wait for this run's disabled «Checking...» marker,
  461 |   // then for that marker to leave the tree (the button returns to «Check work», or the
  462 |   // last unit's actions unmount) before reading the pill. Reading while Checking... is
  463 |   // still up would accept the previous verdict.
```