# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-recovery.spec.ts >> provider transport failure mid-creation: interrupted guidance, resume, library intact
- Location: e2e/course-generation-recovery.spec.ts:224:1

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
  - paragraph: Personal budget and reserve
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
    - paragraph: Foundations of Personal budget and reserve
    - paragraph: Прошло 0:02
    - paragraph: Можно вернуться в библиотеку. Создание продолжится, пока приложение работает
```

# Test source

```ts
  153 |  * The stale digest must never be consented: at least one confirmation was sent, every
  154 |  * accepted (2xx) one carries the fresh digest and never the stale one, and every request
  155 |  * carrying the stale digest was refused (409).
  156 |  */
  157 | function assertOnlyFreshPlanConsented(
  158 |   confirmations: readonly PlanConfirmation[],
  159 |   staleDigest: string,
  160 |   freshDigest: string,
  161 | ): void {
  162 |   expect(freshDigest, 'tab A observed a fresh digest').not.toBe('');
  163 |   expect(freshDigest, 'fresh digest differs from the stale one').not.toBe(staleDigest);
  164 |   expect(confirmations.length, 'tab B sent at least one plan confirmation').toBeGreaterThan(0);
  165 |   const accepted = confirmations.filter((c) => c.status >= 200 && c.status < 300);
  166 |   expect(accepted.length, 'tab B consent was accepted').toBeGreaterThan(0);
  167 |   for (const confirmation of accepted) {
  168 |     expect(confirmation.blueprintDigest).not.toBe(staleDigest);
  169 |     expect(confirmation.blueprintDigest).toBe(freshDigest);
  170 |   }
  171 |   for (const confirmation of confirmations.filter((c) => c.blueprintDigest === staleDigest)) {
  172 |     expect(confirmation.status, 'a stale-digest confirmation is refused').toBe(409);
  173 |   }
  174 | }
  175 | 
  176 | async function assertDeterministicMaterializationOrder(): Promise<void> {
  177 |   const log = await generationCalls();
  178 |   const mats = log.calls.filter((call) => call.kind === 'materialize_unit');
  179 |   expect(log.counts['propose_learning_map'] ?? 0).toBe(0);
  180 |   expect(mats.map((call) => call.unit_id)).toEqual([PRACTICE_UNIT_ID, TRANSFER_UNIT_ID]);
  181 | }
  182 | 
  183 | test.describe.configure({ mode: 'serial' });
  184 | 
  185 | // Scope drain at every boundary (CG21-121 flake fix): one global server shares the
  186 | // authoring queue and the fake's fault arming across tests, so a leaked non-terminal
  187 | // operation from a prior test would either steal worker cycles (FIFO scope queue) or
  188 | // consume this test's armed fault. The hooks keep every test on a quiet scope.
  189 | test.beforeEach(async ({ page }) => {
  190 |   await drainAuthoringQueue(page);
  191 | });
  192 | test.afterEach(async ({ page }) => {
  193 |   await drainAuthoringQueue(page);
  194 | });
  195 | 
  196 | test('capabilities read failure: library notice, wizard guidance, library intact', async ({
  197 |   page,
  198 | }) => {
  199 |   await resetGenerationFake();
  200 |   // Bounded read-failure injection on the capabilities projection only (never a faked
  201 |   // success): the app must treat the unknown state conservatively.
  202 |   await page.route('**/api/v1/authoring/capabilities', (route) => route.abort());
  203 |   await gotoLearnerApp(page);
  204 |   await expect(
  205 |     page.getByRole('heading', { name: 'AI для создания курсов пока не настроен' }),
  206 |   ).toBeVisible({ timeout: 30_000 });
  207 |   // The library stays intact: existing courses and templates remain listed.
  208 |   await expect(page.getByRole('button', { name: 'Создать курс' })).toBeVisible();
  209 | 
  210 |   await page.getByRole('button', { name: 'Создать курс' }).click();
  211 |   await expect(page.getByText('AI для создания курсов пока не настроен')).toBeVisible();
  212 |   // The goal CTA stays disabled: no creation is promised without a working provider.
  213 |   await page.fill('#wizard-goal', 'Anything at all');
  214 |   await expect(page.getByRole('button', { name: 'Продолжить' })).toBeDisabled();
  215 | 
  216 |   await page.unroute('**/api/v1/authoring/capabilities');
  217 |   await page.reload();
  218 |   await gotoLearnerApp(page);
  219 |   await expect(
  220 |     page.getByRole('heading', { name: 'AI для создания курсов пока не настроен' }),
  221 |   ).toHaveCount(0);
  222 | });
  223 | 
  224 | test('provider transport failure mid-creation: interrupted guidance, resume, library intact', async ({
  225 |   page,
  226 | }) => {
  227 |   await resetGenerationFake();
  228 |   await fakeControl('fail-next/materialize_unit');
  229 |   const subject = 'finance' as const;
  230 |   const planWatcher = watchSessionPlanDigest(page);
  231 |   await driveToConfirmedPlan(page, subject);
  232 |   await expect.poll(() => planWatcher.digest(), { timeout: 90_000 }).not.toBe('');
  233 |   const blueprintDigest = planWatcher.digest();
  234 |   await confirmPlanStep(page);
  235 |   // The first materialize call failed with a transport error: the wizard shows the
  236 |   // interrupted recovery with the no-new-consent note and the continue action.
  237 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  238 |     timeout: 60_000,
  239 |   });
  240 |   await expect(page.getByText('Подтверждать прежнюю программу повторно не нужно')).toBeVisible();
  241 | 
  242 |   // The library stays intact while the wizard sits in the failure phase.
  243 |   await page.getByRole('button', { name: 'В библиотеку' }).click();
  244 |   await expect(page.getByRole('heading', { name: 'Ваше обучение' })).toBeVisible();
  245 |   await gotoLearnerApp(page);
  246 |   // Resume through the draft shelf: the failure state is durable server-side.
  247 |   await openActiveDraft(page, subject, blueprintDigest);
  248 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  249 |     timeout: 60_000,
  250 |   });
  251 | 
  252 |   await page.getByRole('button', { name: 'Продолжить создание' }).click();
> 253 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
      |                                                                       ^ Error: expect(locator).toBeVisible() failed
  254 |     timeout: 180_000,
  255 |   });
  256 | 
  257 |   // The failed unit was re-materialized once; the second unit ran exactly once.
  258 |   expect(await countCalls('materialize_unit', PRACTICE_UNIT_ID)).toBe(2);
  259 |   expect(await countCalls('materialize_unit', TRANSFER_UNIT_ID)).toBe(1);
  260 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  261 |   expect(await countCalls('propose_learning_map')).toBe(0);
  262 | });
  263 | 
  264 | test('interrupted mid-flight: completed checkpoints are not regenerated', async ({ page }) => {
  265 |   await resetGenerationFake();
  266 |   // Practice materializes fine; the transfer provider call is interrupted mid-flight.
  267 |   await fakeControl(`drop-next/materialize_unit/${TRANSFER_UNIT_ID}`);
  268 |   await driveToConfirmedPlan(page, 'management');
  269 |   await confirmPlanStep(page);
  270 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  271 |     timeout: 60_000,
  272 |   });
  273 | 
  274 |   await page.getByRole('button', { name: 'Продолжить создание' }).click();
  275 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  276 |     timeout: 180_000,
  277 |   });
  278 | 
  279 |   // Checkpoint retention: practice ran once; only the interrupted transfer ran again.
  280 |   expect(await countCalls('materialize_unit', PRACTICE_UNIT_ID)).toBe(1);
  281 |   expect(await countCalls('materialize_unit', TRANSFER_UNIT_ID)).toBe(2);
  282 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  283 |   expect(await countCalls('propose_learning_map')).toBe(0);
  284 | });
  285 | 
  286 | test('two tabs: stale plan is never materialized', async ({ page }) => {
  287 |   await resetGenerationFake();
  288 |   const description = subjectDescription('python');
  289 |   await openGenerator(page);
  290 |   await submitGoal(page, description);
  291 |   await confirmProfileStep(page, description.split('\n')[0]);
  292 |   // Tab A stays at the plan preview WITHOUT confirming: the first consent must come
  293 |   // from Tab B, so a stale attempt cannot race an already-running materialize.
  294 |   const tabAPlan = watchSessionPlanDigest(page);
  295 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  296 |     timeout: 90_000,
  297 |   });
  298 |   await expect.poll(() => tabAPlan.digest(), { timeout: 90_000 }).not.toBe('');
  299 |   const staleDigest = tabAPlan.digest();
  300 |   // Tab B attaches to the same durable draft. Its 4s poll may absorb A's revision
  301 |   // before confirm; both outcomes are valid as long as the old plan is not generated.
  302 |   const tabB = await page.context().newPage();
  303 |   const tabBPlan = watchSessionPlanDigest(tabB);
  304 |   const tabBConfirmations = watchPlanConfirmations(tabB);
  305 |   try {
  306 |     await gotoLearnerApp(tabB);
  307 |     await openPlanDraftByDigest(tabB, description.split('\n')[0], staleDigest);
  308 |     const confirmB = tabB.getByRole('button', { name: 'Подтвердить и создать курс' });
  309 |     await expect(confirmB).toBeEnabled({ timeout: 90_000 });
  310 |     // The session ref the opened wizard actually reads (a re-minted public ref of the
  311 |     // SAME durable draft tab A opened; matching the compiled digest disambiguates
  312 |     // earlier retained python plan_review sessions without relying on shelf order.
  313 |     const tabBRef = tabBPlan.seenRef();
  314 |     if (tabBRef === '') throw new Error('tab B never fetched its authoring session');
  315 |     const tabBDigest = async () => {
  316 |       const session = await apiGet(tabB, `/api/v1/authoring/sessions/${tabBRef}`);
  317 |       const digest = (
  318 |         session['plan'] as { summary?: { blueprint_digest?: string } } | null
  319 |       )?.summary?.blueprint_digest;
  320 |       return typeof digest === 'string' ? digest : '';
  321 |     };
  322 |     // Two independent proofs of the same stale plan: the in-page digest watcher
  323 |     // (the visible compiled program) AND the direct API read both agree.
  324 |     await expect.poll(() => tabBPlan.digest(), { timeout: 90_000 }).toBe(staleDigest);
  325 |     await expect.poll(() => tabBDigest(), { timeout: 90_000 }).toBe(staleDigest);
  326 | 
  327 |     // One revision only. Weekly +30 may not change «N минут» (fake caps units at 360);
  328 |     // the digest is the consent identity.
  329 |     await page.getByRole('button', { name: 'Изменить', exact: true }).click();
  330 |     // The revise wire now carries whole hours 1..40 (M24-004 from server snapshot);
  331 |     // pick a changed in-range value instead of the old +30-minute delta.
  332 |     const hoursInput = page.locator('input[type="number"]').first();
  333 |     const currentHours = Number(await hoursInput.inputValue());
  334 |     const nextHours = currentHours < 5 ? currentHours + 1 : currentHours - 1;
  335 |     await hoursInput.fill(String(Math.max(1, nextHours)));
  336 |     await page.getByRole('button', { name: 'Применить изменения' }).click();
  337 |     await expect(
  338 |       page.getByRole('button', { name: 'Подтвердить и создать курс' }),
  339 |     ).toBeEnabled({ timeout: 120_000 });
  340 |     await expect.poll(() => tabAPlan.digest(), { timeout: 60_000 }).not.toBe(staleDigest);
  341 |     const freshDigest = tabAPlan.digest();
  342 |     // A profile revision drafts against the new spec, then deterministic mapping
  343 |     // prepares the plan without an external map-provider call.
  344 |     expect(await countCalls('propose_plan_draft')).toBe(2);
  345 |     expect(await countCalls('propose_learning_map')).toBe(0);
  346 |     expect(await countCalls('materialize_unit')).toBe(0);
  347 | 
  348 |     // Click may wait if B is mid re-read (confirm unmounts while the new map compiles).
  349 |     await confirmB.click({ timeout: 120_000 });
  350 |     const outcome = await waitForTwoTabConfirmOutcome(tabB);
  351 |     if (outcome === 'stale_refused') {
  352 |       expect(await countCalls('materialize_unit')).toBe(0);
  353 |       await expect.poll(() => tabBDigest(), { timeout: 60_000 }).not.toBe(staleDigest);
```