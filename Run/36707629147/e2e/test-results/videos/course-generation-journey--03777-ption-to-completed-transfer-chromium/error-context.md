# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-journey.spec.ts >> python: full Q2 journey from description to completed transfer
- Location: e2e/course-generation-journey.spec.ts:104:1

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
    - listitem: 3 Программа
    - listitem: 4 Создание
  - paragraph: Python functions checked by integer cases
  - text: Программа Готовим занятия
  - region "Проверьте программу":
    - paragraph: Программа
    - heading "Проверьте программу" [level=1]
    - text: Черновик v11
    - paragraph: Черновик готов. Уточните детали или создайте курс по этой программе.
    - heading "Python functions checked by integer cases draft" [level=2]
    - paragraph: Write and verify small pure functions against published integer cases.
    - article:
      - paragraph: Настроим курс
      - paragraph: Необязательно
      - heading "Should the draft add one guided practice unit?" [level=3]
      - paragraph: "Почему спрашиваем: The draft depends on this bounded personalization signal."
      - paragraph: "Что изменится: The chosen answer rewrites the affected plan modules."
      - group "Should the draft add one guided practice unit?":
        - radio "Keep No extra guided unit."
        - text: Keep No extra guided unit.
        - radio "Add one Add one guided unit."
        - text: Add one Add one guided unit.
      - button "Оставить как в черновике"
      - button "Доверьте выбор программе"
    - article:
      - paragraph: Настроим курс
      - paragraph: Необязательно
      - heading "Should the draft favor a deeper core or a shorter course?" [level=3]
      - paragraph: "Почему спрашиваем: The module depth follows this bounded personalization signal."
      - paragraph: "Что изменится: The chosen answer rewrites the affected plan modules."
      - status:
        - paragraph: Ответ применён
        - paragraph: Keep depth
        - button "Изменить выбор"
    - region "Модули программы":
      - heading "Модули программы" [level=3]
      - group: 01 Foundations of Python functions checked by integer cases 720 мин Core ideas of Python functions checked by integer cases → Keep depth (module_pace)
    - paragraph: Принятые допущения
    - list:
      - listitem: Modules follow a foundations-first order picked by the program, not a stated learner preference.
    - region "Модули по этапам":
      - heading "Модули по этапам" [level=3]
      - list:
        - listitem: "Этап 1: Foundations of Python functions checked by integer cases"
    - paragraph: Write and verify small pure functions against published integer cases.
    - paragraph: Как проверим
    - list:
      - listitem: "lf158-group-01-practice: implementation evidence via published criteria"
      - listitem: "lf158-group-01-transfer: transfer evidence via published criteria"
    - paragraph: Что не проверяем
    - list:
      - listitem: Free-form explanations are not graded as proof of solution quality.
      - listitem: No income, investment or financial outcomes are promised or checked.
      - listitem: Transfer is checked in one neighboring context only.
    - status:
      - paragraph: "Отмечено карточек: 0"
      - paragraph: Отметьте ответ, пропуск или выбор программы хотя бы в одной карточке.
    - button "Применить ответы" [disabled]
    - list:
      - listitem: "Программа обновлена: m1: added revision topic 'Keep depth (module_pace)' (module_pace)."
    - button "Изменить"
    - button "Назад"
    - complementary:
      - heading "Параметры курса" [level=2]
      - button "Изменить параметры"
      - term: Ваш опыт
      - definition: Начинаю с нуля
      - term: Время в неделю, часов
      - definition: "15"
      - term: На сколько недель?
      - definition: "26"
      - term: Язык курса
      - definition: Русский
    - button "Подтвердить и создать курс"
```

# Test source

```ts
  195 | /** The one batch endpoint (EPIC-25 WP2); per-card answer/skip/delegate routes no longer exist. */
  196 | const DRAFT_ANSWERS_PATH = /\/api\/v1\/authoring\/sessions\/[^/]+\/draft-answers$/;
  197 | 
  198 | /** Count and capture the `draft-answers` POST bodies this page sends (the one-batch proof). */
  199 | export function watchDraftAnswerPosts(page: Page): {
  200 |   count: () => number;
  201 |   bodies: () => Array<{ items?: Array<Record<string, unknown>> }>;
  202 | } {
  203 |   const bodies: Array<{ items?: Array<Record<string, unknown>> }> = [];
  204 |   page.on('request', (request) => {
  205 |     if (request.method() === 'POST' && DRAFT_ANSWERS_PATH.test(new URL(request.url()).pathname)) {
  206 |       bodies.push(request.postDataJSON() as { items?: Array<Record<string, unknown>> });
  207 |     }
  208 |   });
  209 |   return { count: () => bodies.length, bodies: () => bodies };
  210 | }
  211 | 
  212 | /** Open (unanswered) clarification cards, in render order; decided read-only cards are excluded. */
  213 | export function openQuestionCards(page: Page): Locator {
  214 |   return page
  215 |     .locator('article[data-testid^="question-"]')
  216 |     .filter({ hasNot: page.locator('[data-testid^="question-decision-"]') });
  217 | }
  218 | 
  219 | /** Decided (answered/skipped/delegated) read-only cards carried after an applied batch (E-12). */
  220 | export function decidedQuestionCards(page: Page): Locator {
  221 |   return page
  222 |     .locator('article[data-testid^="question-"]')
  223 |     .filter({ has: page.locator('[data-testid^="question-decision-"]') });
  224 | }
  225 | 
  226 | /** Locally mark a card with its first option (no network call). Returns the option id. */
  227 | export async function markFirstOption(card: Locator): Promise<string> {
  228 |   const radio = card.locator('label[data-testid^="question-option-"] input[type="radio"]').first();
  229 |   const optionId = await radio.inputValue();
  230 |   await radio.check();
  231 |   await expect(radio).toBeChecked();
  232 |   return optionId;
  233 | }
  234 | 
  235 | /** Click «Применить ответы» once the bar enables it; resolves with the 202 draft-answers response. */
  236 | export async function applyDraftAnswers(page: Page): Promise<void> {
  237 |   const bar = page.getByTestId('draft-answers-bar');
  238 |   const apply = bar.getByRole('button', { name: 'Применить ответы' });
  239 |   await expect(apply).toBeEnabled({ timeout: 30_000 });
  240 |   const [response] = await Promise.all([
  241 |     page.waitForResponse(
  242 |       (candidate) =>
  243 |         candidate.request().method() === 'POST' &&
  244 |         DRAFT_ANSWERS_PATH.test(new URL(candidate.url()).pathname),
  245 |     ),
  246 |     apply.click(),
  247 |   ]);
  248 |   expect(response.status(), 'draft-answers accepted').toBe(202);
  249 | }
  250 | 
  251 | /**
  252 |  * Answer ONE draft clarification card through the batch contract: mark the first option of
  253 |  * the first open card locally, then send it with the single «Применить ответы» submit. The
  254 |  * accepted batch re-runs the plan draft (`propose_plan_draft` >= 2) once and the gate rebuilds
  255 |  * the map for the moved spec without a provider call.
  256 |  */
  257 | export async function answerFirstDraftQuestion(page: Page): Promise<void> {
  258 |   // Wait for the settled plan window first: between the draft commit and the
  259 |   // compiled program the controller auto-compiles the gated map, and that
  260 |   // draft→compiled branch change REMOUNTS the question cards and clears any
  261 |   // local selection (LF-146 slice B). Interacting before the settle races the
  262 |   // remount; after the CTA is enabled the card state is stable.
  263 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  264 |     timeout: 120_000,
  265 |   });
  266 |   const card = openQuestionCards(page).first();
  267 |   await expect(card).toBeVisible({ timeout: 90_000 });
  268 |   await markFirstOption(card);
  269 |   await applyDraftAnswers(page);
  270 |   // The accepted batch re-runs the plan draft for the moved spec.
  271 |   await waitForCalls(page, 'propose_plan_draft', 2);
  272 | }
  273 | 
  274 | /** Wait for the compiled plan and confirm it once (the single consent point). */
  275 | export async function confirmPlanStep(page: Page): Promise<string> {
  276 |   const cta = page.getByRole('button', { name: 'Подтвердить и создать курс' });
  277 |   await expect(cta).toBeEnabled({ timeout: 120_000 });
  278 |   const [request] = await Promise.all([
  279 |     page.waitForRequest(
  280 |       (candidate) =>
  281 |         candidate.method() === 'POST' &&
  282 |         /\/api\/v1\/authoring\/sessions\/[^/]+\/plan-confirmation$/.test(
  283 |           new URL(candidate.url()).pathname,
  284 |         ),
  285 |     ),
  286 |     cta.click(),
  287 |   ]);
  288 |   const body = request.postDataJSON() as { blueprint_digest?: string };
  289 |   expect(typeof body.blueprint_digest, 'consent carries the compiled plan digest').toBe('string');
  290 |   return body.blueprint_digest as string;
  291 | }
  292 | 
  293 | /** Wait for the wizard to reach the saved-course screen. */
  294 | export async function waitForSavedCourse(page: Page): Promise<void> {
> 295 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
      |                                                                       ^ Error: expect(locator).toBeVisible() failed
  296 |     timeout: 180_000,
  297 |   });
  298 | }
  299 | 
  300 | /**
  301 |  * Wait for the saved-course heading after consent, failing on a visible recovery alert.
  302 |  *
  303 |  * Used when a 180s heading timeout would hide the actual failure: the worker call log
  304 |  * must first show materialization started, then the heading or an alert decides.
  305 |  */
  306 | export async function waitForSavedCourseOrAlert(page: Page, timeoutMs = 60_000): Promise<void> {
  307 |   const saved = page.getByRole('heading', { name: 'Ваш курс готов' });
  308 |   try {
  309 |     await expect(saved).toBeVisible({ timeout: timeoutMs });
  310 |   } catch (error) {
  311 |     const alerts = await page.getByRole('alert').allTextContents();
  312 |     const detail = alerts.map((text) => text.trim()).filter((text) => text.length > 0);
  313 |     throw new Error(
  314 |       `saved heading missing after ${timeoutMs}ms; alerts: ${detail.join(' | ') || 'none'}`,
  315 |       { cause: error },
  316 |     );
  317 |   }
  318 | }
  319 | 
  320 | /** Drive one full creation from the library to the saved screen; returns the course title. */
  321 | export async function driveCreation(
  322 |   page: Page,
  323 |   subject: 'python' | 'finance' | 'management',
  324 | ): Promise<string> {
  325 |   const title = await driveToConfirmedPlan(page, subject);
  326 |   await confirmPlanStep(page);
  327 |   await waitForSavedCourse(page);
  328 |   return title;
  329 | }
  330 | 
  331 | /** Drive one creation up to the confirmed plan (before materialization starts). */
  332 | export async function driveToConfirmedPlan(
  333 |   page: Page,
  334 |   subject: 'python' | 'finance' | 'management',
  335 | ): Promise<string> {
  336 |   const description = subjectDescription(subject);
  337 |   const title = subjectCourseTitle(subject);
  338 |   await openGenerator(page);
  339 |   await submitGoal(page, description);
  340 |   await confirmProfileStep(page, description.split('\n')[0]);
  341 |   return title;
  342 | }
  343 | 
  344 | /** Start the just-saved course and land in its workspace (trust granted when asked). */
  345 | export async function startSavedCourse(page: Page): Promise<void> {
  346 |   await page.getByRole('button', { name: 'Начать обучение' }).click();
  347 |   await grantTrustIfPresent(page);
  348 |   await waitForWorkspace(page);
  349 | }
  350 | 
  351 | // -- workspace driving --------------------------------------------------------
  352 | 
  353 | /** Open one curriculum unit by title prefix and order when source titles repeat. */
  354 | export async function openUnitByTitle(
  355 |   page: Page,
  356 |   titlePrefix: string,
  357 |   occurrence = 0,
  358 | ): Promise<void> {
  359 |   await page.getByRole('button', { name: new RegExp(`^${titlePrefix}`) }).nth(occurrence).click();
  360 | }
  361 | 
  362 | /** Switch to the Code tab of the active unit. */
  363 | export async function openCodeTab(page: Page): Promise<void> {
  364 |   await page.getByRole('tab', { name: 'Code' }).click();
  365 | }
  366 | 
  367 | /**
  368 |  * Replace the whole CodeMirror buffer by pasting. DOM-level typing (`keyboard.type`)
  369 |  * corrupts whitespace inside CodeMirror's contenteditable (newlines collapse into one
  370 |  * line, spaces become U+00A0), which saved REAL wrong code and produced genuine UNMET
  371 |  * verdicts for right answers (CG21-121 python journey flake). Dispatch a paste event
  372 |  * with plain text instead; the editor's own event path renders and autosaves it.
  373 |  */
  374 | export async function fillEditor(page: Page, content: string): Promise<void> {
  375 |   const editor = page.locator('.cm-content');
  376 |   await editor.click();
  377 |   await page.keyboard.press('ControlOrMeta+a');
  378 |   await editor.evaluate((element, text) => {
  379 |     const dataTransfer = new DataTransfer();
  380 |     dataTransfer.setData('text/plain', text);
  381 |     element.dispatchEvent(
  382 |       new ClipboardEvent('paste', { clipboardData: dataTransfer, bubbles: true, cancelable: true }),
  383 |     );
  384 |   }, content);
  385 | }
  386 | /** Run Check work and wait for a terminal rubric pill (passed or unmet). */
  387 | export async function checkWork(page: Page): Promise<'PASSED' | 'UNMET CRITERIA'> {
  388 |   const pill = page.getByText(/^(PASSED|UNMET CRITERIA)$/);
  389 |   const checking = page.getByRole('button', { name: 'Checking...' });
  390 |   await page.getByRole('button', { name: 'Check work' }).click();
  391 |   // A stale terminal pill from the PREVIOUS evaluation may still be mounted while the fresh
  392 |   // run executes (CG21-121 flake fix): wait for this run's disabled «Checking...» marker,
  393 |   // then for that marker to leave the tree (the button returns to «Check work», or the
  394 |   // last unit's actions unmount) before reading the pill. Reading while Checking... is
  395 |   // still up would accept the previous verdict.
```