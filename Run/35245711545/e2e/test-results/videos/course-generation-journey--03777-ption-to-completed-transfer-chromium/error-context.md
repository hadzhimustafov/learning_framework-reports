# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-journey.spec.ts >> python: full Q2 journey from description to completed transfer
- Location: e2e/course-generation-journey.spec.ts:79:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByRole('button', { name: 'Создать курс' })
Expected: visible
Timeout: 30000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 30000ms
  - waiting for getByRole('button', { name: 'Создать курс' })

```

```yaml
- main:
  - alert: A newer build of the learner workspace is available. Please reload the page.
```

# Test source

```ts
  175 | export async function resetGenerationFake(): Promise<void> {
  176 |   await fakeControl('reset');
  177 | }
  178 | 
  179 | export interface FakeCall {
  180 |   kind: string;
  181 |   operation_id: string;
  182 |   unit_id: string;
  183 |   at: number;
  184 | }
  185 | 
  186 | export interface FakeCallLog {
  187 |   calls: FakeCall[];
  188 |   counts: Record<string, number>;
  189 | }
  190 | 
  191 | /** Read the provider call log the fake records for every generation operation. */
  192 | export async function generationCalls(): Promise<FakeCallLog> {
  193 |   const response = await fetch(`${GENERATION_FAKE_URL}/__calls`);
  194 |   if (!response.ok) {
  195 |     throw new Error(`generation fake /__calls failed with ${response.status}`);
  196 |   }
  197 |   return (await response.json()) as FakeCallLog;
  198 | }
  199 | 
  200 | /** Count provider calls of one kind, optionally restricted to one unit id. */
  201 | export async function countCalls(kind: string, unitId?: string): Promise<number> {
  202 |   const log = await generationCalls();
  203 |   return log.calls.filter(
  204 |     (call) => call.kind === kind && (unitId === undefined || call.unit_id === unitId),
  205 |   ).length;
  206 | }
  207 | 
  208 | /**
  209 |  * Boundedly wait until the fake-provider call log reaches a minimum count of one kind.
  210 |  * Provider calls run behind the async queue worker, so a counter read immediately after
  211 |  * an accepted mutation legitimately lags the queue.
  212 |  */
  213 | export async function waitForCalls(
  214 |   page: Page,
  215 |   kind: string,
  216 |   minLength: number,
  217 |   timeoutMs = 60_000,
  218 | ): Promise<void> {
  219 |   void page;
  220 |   try {
  221 |     await expect
  222 |       .poll(() => countCalls(kind), { timeout: timeoutMs, intervals: [200, 500, 1_000] })
  223 |       .toBeGreaterThanOrEqual(minLength);
  224 |   } catch (error) {
  225 |     const log = await generationCalls();
  226 |     throw new Error(
  227 |       `waitForCalls(${kind} >= ${minLength}) timed out; counts=${JSON.stringify(log.counts)}`,
  228 |       { cause: error },
  229 |     );
  230 |   }
  231 | }
  232 | 
  233 | /**
  234 |  * Wait until every authoring session in the shared catalog scope holds NO non-terminal
  235 |  * operation (test isolation, CG21-121 flake fix).
  236 |  *
  237 |  * The e2e server is one global process with one authoring database for the whole run:
  238 |  * the operation queue is scope-wide FIFO (`peek_queued_operation` picks the OLDEST
  239 |  * queued row, the singleton scope-owner lease guards a claimed one), so a
  240 |  * claimed-or-queued operation leaked by a previous test would run against the NEXT
  241 |  * test's worker cycles and could also consume the shared fake's armed
  242 |  * `fail-next`/`drop-next` fault. Draining the scope at test boundaries removes that
  243 |  * cross-test interference without restarting the server: the poll reads the real
  244 |  * projection (`active_operation_ref` is null exactly when the operation is terminal)
  245 |  * and each state change is driven by the real worker.
  246 |  */
  247 | export async function drainAuthoringQueue(page: Page, timeoutMs = 60_000): Promise<void> {
  248 |   await expect
  249 |     .poll(
  250 |       async () => {
  251 |         const shelf = await apiGet(page, '/api/v1/authoring/sessions?page_size=50');
  252 |         const entries = (shelf['sessions'] ?? []) as Array<{ session_ref?: string }>;
  253 |         const refs = entries
  254 |           .map((entry) => entry['session_ref'])
  255 |           .filter((ref): ref is string => typeof ref === 'string');
  256 |         for (const ref of refs) {
  257 |           const view = await apiGet(page, `/api/v1/authoring/sessions/${ref}`);
  258 |           if (view['active_operation_ref'] != null) {
  259 |             return 'active';
  260 |           }
  261 |         }
  262 |         return 'drained';
  263 |       },
  264 |       { timeout: timeoutMs, intervals: [200, 500, 1_000] },
  265 |     )
  266 |     .toBe('drained');
  267 | }
  268 | 
  269 | // -- wizard driving -----------------------------------------------------------
  270 | 
  271 | /** Open the generator from the library and wait for the goal step. */
  272 | export async function openGenerator(page: Page): Promise<void> {
  273 |   await gotoLearnerApp(page);
  274 |   const create = page.getByRole('button', { name: 'Создать курс' });
> 275 |   await expect(create).toBeVisible({ timeout: 30_000 });
      |                        ^ Error: expect(locator).toBeVisible() failed
  276 |   await create.click();
  277 |   await expect(page.getByRole('heading', { name: 'Чему хотите научиться?' })).toBeVisible({
  278 |     timeout: 30_000,
  279 |   });
  280 | }
  281 | 
  282 | /** Type the subject description and continue to the profile step. */
  283 | export async function submitGoal(page: Page, description: string): Promise<void> {
  284 |   await page.fill('#wizard-goal', description);
  285 |   await page.getByRole('button', { name: 'Продолжить' }).click();
  286 | }
  287 | 
  288 | /**
  289 |  * Wait until the extracted profile is complete and confirm it unchanged (the digest path).
  290 |  * A01: the extracted skill and outcome are shown for confirmation; nothing else is asked.
  291 |  */
  292 | export async function confirmProfileStep(page: Page, skill: string): Promise<void> {
  293 |   const cta = page.getByRole('button', { name: 'Показать программу' });
  294 |   await expect(cta).toBeEnabled({ timeout: 60_000 });
  295 |   await expect(page.locator('#wizard-skill')).toHaveValue(skill, { timeout: 60_000 });
  296 |   await cta.click();
  297 | }
  298 | 
  299 | /** Wait for the compiled plan and confirm it once (the single consent point). */
  300 | export async function confirmPlanStep(page: Page): Promise<void> {
  301 |   const cta = page.getByRole('button', { name: 'Подтвердить и создать курс' });
  302 |   await expect(cta).toBeEnabled({ timeout: 90_000 });
  303 |   await cta.click();
  304 | }
  305 | 
  306 | /** Wait for the wizard to reach the saved-course screen. */
  307 | export async function waitForSavedCourse(page: Page): Promise<void> {
  308 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  309 |     timeout: 180_000,
  310 |   });
  311 | }
  312 | 
  313 | /**
  314 |  * Wait for the saved-course heading after consent, failing on a visible recovery alert.
  315 |  *
  316 |  * Used when a 180s heading timeout would hide the actual failure: the worker call log
  317 |  * must first show materialization started, then the heading or an alert decides.
  318 |  */
  319 | export async function waitForSavedCourseOrAlert(page: Page, timeoutMs = 60_000): Promise<void> {
  320 |   const saved = page.getByRole('heading', { name: 'Ваш курс готов' });
  321 |   try {
  322 |     await expect(saved).toBeVisible({ timeout: timeoutMs });
  323 |   } catch (error) {
  324 |     const alerts = await page.getByRole('alert').allTextContents();
  325 |     const detail = alerts.map((text) => text.trim()).filter((text) => text.length > 0);
  326 |     throw new Error(
  327 |       `saved heading missing after ${timeoutMs}ms; alerts: ${detail.join(' | ') || 'none'}`,
  328 |       { cause: error },
  329 |     );
  330 |   }
  331 | }
  332 | 
  333 | /** Drive one full creation from the library to the saved screen; returns the course title. */
  334 | export async function driveCreation(
  335 |   page: Page,
  336 |   subject: 'python' | 'finance' | 'management',
  337 | ): Promise<string> {
  338 |   const title = await driveToConfirmedPlan(page, subject);
  339 |   await confirmPlanStep(page);
  340 |   await waitForSavedCourse(page);
  341 |   return title;
  342 | }
  343 | 
  344 | /** Drive one creation up to the confirmed plan (before materialization starts). */
  345 | export async function driveToConfirmedPlan(
  346 |   page: Page,
  347 |   subject: 'python' | 'finance' | 'management',
  348 | ): Promise<string> {
  349 |   const description = subjectDescription(subject);
  350 |   const title = subjectCourseTitle(subject);
  351 |   await openGenerator(page);
  352 |   await submitGoal(page, description);
  353 |   await confirmProfileStep(page, description.split('\n')[0]);
  354 |   return title;
  355 | }
  356 | 
  357 | /** Start the just-saved course and land in its workspace (trust granted when asked). */
  358 | export async function startSavedCourse(page: Page): Promise<void> {
  359 |   await page.getByRole('button', { name: 'Начать обучение' }).click();
  360 |   await grantTrustIfPresent(page);
  361 |   await waitForWorkspace(page);
  362 | }
  363 | 
  364 | // -- workspace driving --------------------------------------------------------
  365 | 
  366 | /** Open one curriculum unit by its title prefix (practice/transfer). */
  367 | export async function openUnitByTitle(page: Page, titlePrefix: string): Promise<void> {
  368 |   await page.getByRole('button', { name: new RegExp(`^${titlePrefix}`) }).first().click();
  369 | }
  370 | 
  371 | /** Switch to the Code tab of the active unit. */
  372 | export async function openCodeTab(page: Page): Promise<void> {
  373 |   await page.getByRole('tab', { name: 'Code' }).click();
  374 | }
  375 | 
```