# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-recovery.spec.ts >> provider transport failure mid-creation: interrupted guidance, resume, library intact
- Location: e2e/course-generation-recovery.spec.ts:224:1

# Error details

```
Error: expect(locator).toHaveValue(expected) failed

Locator:  locator('#wizard-skill')
Expected: "Personal budget and reserve"
Received: ""
Timeout:  60000ms

Call log:
  - Expect "toHaveValue" with timeout 60000ms
  - waiting for locator('#wizard-skill')
    116 × locator resolved to <input value="" type="text" id="wizard-skill" aria-invalid="false" aria-describedby="wizard-skill-short" class="min-h-[48px] w-full rounded-xl border border-slate-300 px-4 py-3 aria-invalid:border-rose-600"/>
        - unexpected value ""

```

```yaml
- textbox "Навык"
```

# Test source

```ts
  79  | /** Count provider calls of one kind, optionally restricted to one unit id. */
  80  | export async function countCalls(kind: string, unitId?: string): Promise<number> {
  81  |   const log = await generationCalls();
  82  |   return log.calls.filter(
  83  |     (call) => call.kind === kind && (unitId === undefined || call.unit_id === unitId),
  84  |   ).length;
  85  | }
  86  | 
  87  | /**
  88  |  * Boundedly wait until the fake-provider call log reaches a minimum count of one kind.
  89  |  * Provider calls run behind the async queue worker, so a counter read immediately after
  90  |  * an accepted mutation legitimately lags the queue.
  91  |  */
  92  | export async function waitForCalls(
  93  |   page: Page,
  94  |   kind: string,
  95  |   minLength: number,
  96  |   timeoutMs = 60_000,
  97  | ): Promise<void> {
  98  |   void page;
  99  |   try {
  100 |     await expect
  101 |       .poll(() => countCalls(kind), { timeout: timeoutMs, intervals: [200, 500, 1_000] })
  102 |       .toBeGreaterThanOrEqual(minLength);
  103 |   } catch (error) {
  104 |     const log = await generationCalls();
  105 |     throw new Error(
  106 |       `waitForCalls(${kind} >= ${minLength}) timed out; counts=${JSON.stringify(log.counts)}`,
  107 |       { cause: error },
  108 |     );
  109 |   }
  110 | }
  111 | 
  112 | /**
  113 |  * Wait until every authoring session in the shared catalog scope holds NO non-terminal
  114 |  * operation (test isolation, CG21-121 flake fix).
  115 |  *
  116 |  * The e2e server is one global process with one authoring database for the whole run:
  117 |  * the operation queue is scope-wide FIFO (`peek_queued_operation` picks the OLDEST
  118 |  * queued row, the singleton scope-owner lease guards a claimed one), so a
  119 |  * claimed-or-queued operation leaked by a previous test would run against the NEXT
  120 |  * test's worker cycles and could also consume the shared fake's armed
  121 |  * `fail-next`/`drop-next` fault. Draining the scope at test boundaries removes that
  122 |  * cross-test interference without restarting the server: the poll reads the real
  123 |  * projection (`active_operation_ref` is null exactly when the operation is terminal)
  124 |  * and each state change is driven by the real worker.
  125 |  */
  126 | export async function drainAuthoringQueue(page: Page, timeoutMs = 60_000): Promise<void> {
  127 |   await expect
  128 |     .poll(
  129 |       async () => {
  130 |         const shelf = await apiGet(page, '/api/v1/authoring/sessions?page_size=50');
  131 |         const entries = (shelf['sessions'] ?? []) as Array<{ session_ref?: string }>;
  132 |         const refs = entries
  133 |           .map((entry) => entry['session_ref'])
  134 |           .filter((ref): ref is string => typeof ref === 'string');
  135 |         for (const ref of refs) {
  136 |           const view = await apiGet(page, `/api/v1/authoring/sessions/${ref}`);
  137 |           if (view['active_operation_ref'] != null) {
  138 |             return 'active';
  139 |           }
  140 |         }
  141 |         return 'drained';
  142 |       },
  143 |       { timeout: timeoutMs, intervals: [200, 500, 1_000] },
  144 |     )
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
> 179 |   await expect(page.locator('#wizard-skill')).toHaveValue(skill, { timeout: 60_000 });
      |                                               ^ Error: expect(locator).toHaveValue(expected) failed
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
```