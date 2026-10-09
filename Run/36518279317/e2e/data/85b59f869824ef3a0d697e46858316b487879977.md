# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-journey.spec.ts >> python: full Q2 journey from description to completed transfer
- Location: e2e/course-generation-journey.spec.ts:81:1

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
  - paragraph: Python functions checked by integer cases
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
    - paragraph: Foundations of Python functions checked by integer cases
    - paragraph: Прошло 0:00
    - paragraph: Можно вернуться в библиотеку. Создание продолжится, пока приложение работает
```

# Test source

```ts
  145 |     .toBe('drained');
  146 | }
  147 | 
  148 | // -- wizard driving -----------------------------------------------------------
  149 | 
  150 | /** Open the generator from the library and wait for the goal step. */
  151 | export async function openGenerator(page: Page): Promise<void> {
  152 |   await gotoLearnerApp(page);
  153 |   const create = page.getByRole('button', { name: 'Создать курс' });
  154 |   await expect(create).toBeVisible({ timeout: 30_000 });
  155 |   await create.click();
  156 |   await expect(page.getByRole('heading', { name: 'Чему хотите научиться?' })).toBeVisible({
  157 |     timeout: 30_000,
  158 |   });
  159 | }
  160 | 
  161 | /** Type the subject description and continue to the profile step. */
  162 | export async function submitGoal(page: Page, description: string): Promise<void> {
  163 |   await page.fill('#wizard-goal', description);
  164 |   await page.getByRole('button', { name: 'Продолжить' }).click();
  165 | }
  166 | 
  167 | /**
  168 |  * Wait until the extracted profile is complete and confirm it unchanged (the digest path).
  169 |  * A01: the extracted skill and outcome are shown for confirmation; nothing else is asked.
  170 |  *
  171 |  * LF-141 flip: the confirmation itself enqueues the `propose_plan_draft`, the worker runs
  172 |  * it and the post-draft gate builds the map deterministically, so after «Показать
  173 |  * программу» the wizard waits in the EPIC-25 draft program window
  174 |  * (`draft-version-badge`) until the program is compiled — the consent CTA is the NEW
  175 |  * step, not the confirm output.
  176 |  */
  177 | export async function confirmProfileStep(page: Page, skill: string): Promise<void> {
  178 |   const cta = page.getByRole('button', { name: 'Показать программу' });
  179 |   await expect(page.locator('#wizard-skill')).toHaveValue(skill, { timeout: 60_000 });
  180 |   const language = page.locator('#wizard-language');
  181 |   if ((await language.inputValue()) === '' && !(await cta.isEnabled())) {
  182 |     await language.selectOption({ label: 'Русский' });
  183 |   }
  184 |   await expect(cta).toBeEnabled({ timeout: 60_000 });
  185 |   await cta.click();
  186 |   // The flip-enqueued draft ran: its clarification cards render while the server builds
  187 |   // the current-draft map without a generation-provider request.
  188 |   await expect(page.getByTestId('draft-version-badge')).toBeVisible({ timeout: 120_000 });
  189 |   await waitForCalls(page, 'propose_plan_draft', 1);
  190 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  191 |     timeout: 120_000,
  192 |   });
  193 | }
  194 | 
  195 | /**
  196 |  * Answer ONE draft clarification card through the real card contract: select the first
  197 |  * option of the first open card and Apply it exactly once. The 202 flow re-runs the plan
  198 |  * draft (`propose_plan_draft` >= 2) and the gate rebuilds the map for the moved spec
  199 |  * without a provider call.
  200 |  */
  201 | export async function answerFirstDraftQuestion(page: Page): Promise<void> {
  202 |   // Wait for the settled plan window first: between the draft commit and the
  203 |   // compiled program the controller auto-compiles the gated map, and that
  204 |   // draft→compiled branch change REMOUNTS the question cards and clears any
  205 |   // local selection (LF-146 slice B). Interacting before the settle races the
  206 |   // remount; after the CTA is enabled the card state is stable.
  207 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  208 |     timeout: 120_000,
  209 |   });
  210 |   const card = page.locator('article[data-testid^="question-"]').first();
  211 |   await expect(card).toBeVisible({ timeout: 90_000 });
  212 |   await card
  213 |     .locator('label[data-testid^="question-option-"] input[type="radio"]')
  214 |     .first()
  215 |     .check();
  216 |   // The local dirty lock enables Apply only after the selection is registered.
  217 |   const apply = card.getByRole('button', { name: 'Применить ответ' });
  218 |   await expect(apply).toBeEnabled({ timeout: 30_000 });
  219 |   await apply.click();
  220 |   // The accepted answer re-runs the plan draft for the moved spec.
  221 |   await waitForCalls(page, 'propose_plan_draft', 2);
  222 | }
  223 | 
  224 | /** Wait for the compiled plan and confirm it once (the single consent point). */
  225 | export async function confirmPlanStep(page: Page): Promise<string> {
  226 |   const cta = page.getByRole('button', { name: 'Подтвердить и создать курс' });
  227 |   await expect(cta).toBeEnabled({ timeout: 120_000 });
  228 |   const [request] = await Promise.all([
  229 |     page.waitForRequest(
  230 |       (candidate) =>
  231 |         candidate.method() === 'POST' &&
  232 |         /\/api\/v1\/authoring\/sessions\/[^/]+\/plan-confirmation$/.test(
  233 |           new URL(candidate.url()).pathname,
  234 |         ),
  235 |     ),
  236 |     cta.click(),
  237 |   ]);
  238 |   const body = request.postDataJSON() as { blueprint_digest?: string };
  239 |   expect(typeof body.blueprint_digest, 'consent carries the compiled plan digest').toBe('string');
  240 |   return body.blueprint_digest as string;
  241 | }
  242 | 
  243 | /** Wait for the wizard to reach the saved-course screen. */
  244 | export async function waitForSavedCourse(page: Page): Promise<void> {
> 245 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
      |                                                                       ^ Error: expect(locator).toBeVisible() failed
  246 |     timeout: 180_000,
  247 |   });
  248 | }
  249 | 
  250 | /**
  251 |  * Wait for the saved-course heading after consent, failing on a visible recovery alert.
  252 |  *
  253 |  * Used when a 180s heading timeout would hide the actual failure: the worker call log
  254 |  * must first show materialization started, then the heading or an alert decides.
  255 |  */
  256 | export async function waitForSavedCourseOrAlert(page: Page, timeoutMs = 60_000): Promise<void> {
  257 |   const saved = page.getByRole('heading', { name: 'Ваш курс готов' });
  258 |   try {
  259 |     await expect(saved).toBeVisible({ timeout: timeoutMs });
  260 |   } catch (error) {
  261 |     const alerts = await page.getByRole('alert').allTextContents();
  262 |     const detail = alerts.map((text) => text.trim()).filter((text) => text.length > 0);
  263 |     throw new Error(
  264 |       `saved heading missing after ${timeoutMs}ms; alerts: ${detail.join(' | ') || 'none'}`,
  265 |       { cause: error },
  266 |     );
  267 |   }
  268 | }
  269 | 
  270 | /** Drive one full creation from the library to the saved screen; returns the course title. */
  271 | export async function driveCreation(
  272 |   page: Page,
  273 |   subject: 'python' | 'finance' | 'management',
  274 | ): Promise<string> {
  275 |   const title = await driveToConfirmedPlan(page, subject);
  276 |   await confirmPlanStep(page);
  277 |   await waitForSavedCourse(page);
  278 |   return title;
  279 | }
  280 | 
  281 | /** Drive one creation up to the confirmed plan (before materialization starts). */
  282 | export async function driveToConfirmedPlan(
  283 |   page: Page,
  284 |   subject: 'python' | 'finance' | 'management',
  285 | ): Promise<string> {
  286 |   const description = subjectDescription(subject);
  287 |   const title = subjectCourseTitle(subject);
  288 |   await openGenerator(page);
  289 |   await submitGoal(page, description);
  290 |   await confirmProfileStep(page, description.split('\n')[0]);
  291 |   return title;
  292 | }
  293 | 
  294 | /** Start the just-saved course and land in its workspace (trust granted when asked). */
  295 | export async function startSavedCourse(page: Page): Promise<void> {
  296 |   await page.getByRole('button', { name: 'Начать обучение' }).click();
  297 |   await grantTrustIfPresent(page);
  298 |   await waitForWorkspace(page);
  299 | }
  300 | 
  301 | // -- workspace driving --------------------------------------------------------
  302 | 
  303 | /** Open one curriculum unit by title prefix and order when source titles repeat. */
  304 | export async function openUnitByTitle(
  305 |   page: Page,
  306 |   titlePrefix: string,
  307 |   occurrence = 0,
  308 | ): Promise<void> {
  309 |   await page.getByRole('button', { name: new RegExp(`^${titlePrefix}`) }).nth(occurrence).click();
  310 | }
  311 | 
  312 | /** Switch to the Code tab of the active unit. */
  313 | export async function openCodeTab(page: Page): Promise<void> {
  314 |   await page.getByRole('tab', { name: 'Code' }).click();
  315 | }
  316 | 
  317 | /**
  318 |  * Replace the whole CodeMirror buffer by pasting. DOM-level typing (`keyboard.type`)
  319 |  * corrupts whitespace inside CodeMirror's contenteditable (newlines collapse into one
  320 |  * line, spaces become U+00A0), which saved REAL wrong code and produced genuine UNMET
  321 |  * verdicts for right answers (CG21-121 python journey flake). Dispatch a paste event
  322 |  * with plain text instead; the editor's own event path renders and autosaves it.
  323 |  */
  324 | export async function fillEditor(page: Page, content: string): Promise<void> {
  325 |   const editor = page.locator('.cm-content');
  326 |   await editor.click();
  327 |   await page.keyboard.press('ControlOrMeta+a');
  328 |   await editor.evaluate((element, text) => {
  329 |     const dataTransfer = new DataTransfer();
  330 |     dataTransfer.setData('text/plain', text);
  331 |     element.dispatchEvent(
  332 |       new ClipboardEvent('paste', { clipboardData: dataTransfer, bubbles: true, cancelable: true }),
  333 |     );
  334 |   }, content);
  335 | }
  336 | /** Run Check work and wait for a terminal rubric pill (passed or unmet). */
  337 | export async function checkWork(page: Page): Promise<'PASSED' | 'UNMET CRITERIA'> {
  338 |   const pill = page.getByText(/^(PASSED|UNMET CRITERIA)$/);
  339 |   const checking = page.getByRole('button', { name: 'Checking...' });
  340 |   await page.getByRole('button', { name: 'Check work' }).click();
  341 |   // A stale terminal pill from the PREVIOUS evaluation may still be mounted while the fresh
  342 |   // run executes (CG21-121 flake fix): wait for this run's disabled «Checking...» marker,
  343 |   // then for that marker to leave the tree (the button returns to «Check work», or the
  344 |   // last unit's actions unmount) before reading the pill. Reading while Checking... is
  345 |   // still up would accept the previous verdict.
```