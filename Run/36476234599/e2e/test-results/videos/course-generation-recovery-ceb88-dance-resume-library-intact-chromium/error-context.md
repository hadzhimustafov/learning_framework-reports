# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-recovery.spec.ts >> provider transport failure mid-creation: interrupted guidance, resume, library intact
- Location: e2e/course-generation-recovery.spec.ts:175:1

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
  104 |   tabB: Page,
  105 | ): Promise<'stale_refused' | 'fresh_started'> {
  106 |   await expect
  107 |     .poll(
  108 |       async () => {
  109 |         const noticed =
  110 |           (await tabB.getByRole('alert').filter({ hasText: STALE_PLAN_COPY }).count()) > 0;
  111 |         const started = await twoTabConfirmOutcome();
  112 |         if (started === 'fresh_started') return 'fresh_started';
  113 |         if (noticed && (await countCalls('materialize_unit')) === 0) return 'stale_refused';
  114 |         return 'pending';
  115 |       },
  116 |       { timeout: 90_000, intervals: [200, 500, 1_000] },
  117 |     )
  118 |     .toMatch(/^(stale_refused|fresh_started)$/);
  119 |   const noticed =
  120 |     (await tabB.getByRole('alert').filter({ hasText: STALE_PLAN_COPY }).count()) > 0;
  121 |   const started = await twoTabConfirmOutcome();
  122 |   if (started === 'fresh_started') return 'fresh_started';
  123 |   if (noticed && (await countCalls('materialize_unit')) === 0) return 'stale_refused';
  124 |   throw new Error('two-tab confirm settled then reverted to pending');
  125 | }
  126 | 
  127 | async function assertDeterministicMaterializationOrder(): Promise<void> {
  128 |   const log = await generationCalls();
  129 |   const mats = log.calls.filter((call) => call.kind === 'materialize_unit');
  130 |   expect(log.counts['propose_learning_map'] ?? 0).toBe(0);
  131 |   expect(mats.map((call) => call.unit_id)).toEqual([PRACTICE_UNIT_ID, TRANSFER_UNIT_ID]);
  132 | }
  133 | 
  134 | test.describe.configure({ mode: 'serial' });
  135 | 
  136 | // Scope drain at every boundary (CG21-121 flake fix): one global server shares the
  137 | // authoring queue and the fake's fault arming across tests, so a leaked non-terminal
  138 | // operation from a prior test would either steal worker cycles (FIFO scope queue) or
  139 | // consume this test's armed fault. The hooks keep every test on a quiet scope.
  140 | test.beforeEach(async ({ page }) => {
  141 |   await drainAuthoringQueue(page);
  142 | });
  143 | test.afterEach(async ({ page }) => {
  144 |   await drainAuthoringQueue(page);
  145 | });
  146 | 
  147 | test('capabilities read failure: library notice, wizard guidance, library intact', async ({
  148 |   page,
  149 | }) => {
  150 |   await resetGenerationFake();
  151 |   // Bounded read-failure injection on the capabilities projection only (never a faked
  152 |   // success): the app must treat the unknown state conservatively.
  153 |   await page.route('**/api/v1/authoring/capabilities', (route) => route.abort());
  154 |   await gotoLearnerApp(page);
  155 |   await expect(
  156 |     page.getByRole('heading', { name: 'AI для создания курсов пока не настроен' }),
  157 |   ).toBeVisible({ timeout: 30_000 });
  158 |   // The library stays intact: existing courses and templates remain listed.
  159 |   await expect(page.getByRole('button', { name: 'Создать курс' })).toBeVisible();
  160 | 
  161 |   await page.getByRole('button', { name: 'Создать курс' }).click();
  162 |   await expect(page.getByText('AI для создания курсов пока не настроен')).toBeVisible();
  163 |   // The goal CTA stays disabled: no creation is promised without a working provider.
  164 |   await page.fill('#wizard-goal', 'Anything at all');
  165 |   await expect(page.getByRole('button', { name: 'Продолжить' })).toBeDisabled();
  166 | 
  167 |   await page.unroute('**/api/v1/authoring/capabilities');
  168 |   await page.reload();
  169 |   await gotoLearnerApp(page);
  170 |   await expect(
  171 |     page.getByRole('heading', { name: 'AI для создания курсов пока не настроен' }),
  172 |   ).toHaveCount(0);
  173 | });
  174 | 
  175 | test('provider transport failure mid-creation: interrupted guidance, resume, library intact', async ({
  176 |   page,
  177 | }) => {
  178 |   await resetGenerationFake();
  179 |   await fakeControl('fail-next/materialize_unit');
  180 |   const subject = 'finance' as const;
  181 |   const planWatcher = watchSessionPlanDigest(page);
  182 |   await driveToConfirmedPlan(page, subject);
  183 |   await expect.poll(() => planWatcher.digest(), { timeout: 90_000 }).not.toBe('');
  184 |   const blueprintDigest = planWatcher.digest();
  185 |   await confirmPlanStep(page);
  186 |   // The first materialize call failed with a transport error: the wizard shows the
  187 |   // interrupted recovery with the no-new-consent note and the continue action.
  188 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  189 |     timeout: 60_000,
  190 |   });
  191 |   await expect(page.getByText('Подтверждать прежнюю программу повторно не нужно')).toBeVisible();
  192 | 
  193 |   // The library stays intact while the wizard sits in the failure phase.
  194 |   await page.getByRole('button', { name: 'В библиотеку' }).click();
  195 |   await expect(page.getByRole('heading', { name: 'Ваше обучение' })).toBeVisible();
  196 |   await gotoLearnerApp(page);
  197 |   // Resume through the draft shelf: the failure state is durable server-side.
  198 |   await openActiveDraft(page, subject, blueprintDigest);
  199 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  200 |     timeout: 60_000,
  201 |   });
  202 | 
  203 |   await page.getByRole('button', { name: 'Продолжить создание' }).click();
> 204 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
      |                                                                       ^ Error: expect(locator).toBeVisible() failed
  205 |     timeout: 180_000,
  206 |   });
  207 | 
  208 |   // The failed unit was re-materialized once; the second unit ran exactly once.
  209 |   expect(await countCalls('materialize_unit', PRACTICE_UNIT_ID)).toBe(2);
  210 |   expect(await countCalls('materialize_unit', TRANSFER_UNIT_ID)).toBe(1);
  211 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  212 |   expect(await countCalls('propose_learning_map')).toBe(0);
  213 | });
  214 | 
  215 | test('interrupted mid-flight: completed checkpoints are not regenerated', async ({ page }) => {
  216 |   await resetGenerationFake();
  217 |   // Practice materializes fine; the transfer provider call is interrupted mid-flight.
  218 |   await fakeControl(`drop-next/materialize_unit/${TRANSFER_UNIT_ID}`);
  219 |   await driveToConfirmedPlan(page, 'management');
  220 |   await confirmPlanStep(page);
  221 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  222 |     timeout: 60_000,
  223 |   });
  224 | 
  225 |   await page.getByRole('button', { name: 'Продолжить создание' }).click();
  226 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  227 |     timeout: 180_000,
  228 |   });
  229 | 
  230 |   // Checkpoint retention: practice ran once; only the interrupted transfer ran again.
  231 |   expect(await countCalls('materialize_unit', PRACTICE_UNIT_ID)).toBe(1);
  232 |   expect(await countCalls('materialize_unit', TRANSFER_UNIT_ID)).toBe(2);
  233 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  234 |   expect(await countCalls('propose_learning_map')).toBe(0);
  235 | });
  236 | 
  237 | test('two tabs: stale plan is never materialized', async ({ page }) => {
  238 |   await resetGenerationFake();
  239 |   const description = subjectDescription('python');
  240 |   await openGenerator(page);
  241 |   await submitGoal(page, description);
  242 |   await confirmProfileStep(page, description.split('\n')[0]);
  243 |   // Tab A stays at the plan preview WITHOUT confirming: the first consent must come
  244 |   // from Tab B, so a stale attempt cannot race an already-running materialize.
  245 |   const tabAPlan = watchSessionPlanDigest(page);
  246 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  247 |     timeout: 90_000,
  248 |   });
  249 |   await expect.poll(() => tabAPlan.digest(), { timeout: 90_000 }).not.toBe('');
  250 |   const staleDigest = tabAPlan.digest();
  251 |   // Tab B attaches to the same durable draft. Its 4s poll may absorb A's revision
  252 |   // before confirm; both outcomes are valid as long as the old plan is not generated.
  253 |   const tabB = await page.context().newPage();
  254 |   const tabBPlan = watchSessionPlanDigest(tabB);
  255 |   try {
  256 |     await gotoLearnerApp(tabB);
  257 |     await openPlanDraftByDigest(tabB, description.split('\n')[0], staleDigest);
  258 |     const confirmB = tabB.getByRole('button', { name: 'Подтвердить и создать курс' });
  259 |     await expect(confirmB).toBeEnabled({ timeout: 90_000 });
  260 |     // The session ref the opened wizard actually reads (a re-minted public ref of the
  261 |     // SAME durable draft tab A opened; matching the compiled digest disambiguates
  262 |     // earlier retained python plan_review sessions without relying on shelf order.
  263 |     const tabBRef = tabBPlan.seenRef();
  264 |     if (tabBRef === '') throw new Error('tab B never fetched its authoring session');
  265 |     const tabBDigest = async () => {
  266 |       const session = await apiGet(tabB, `/api/v1/authoring/sessions/${tabBRef}`);
  267 |       const digest = (
  268 |         session['plan'] as { summary?: { blueprint_digest?: string } } | null
  269 |       )?.summary?.blueprint_digest;
  270 |       return typeof digest === 'string' ? digest : '';
  271 |     };
  272 |     // Two independent proofs of the same stale plan: the in-page digest watcher
  273 |     // (the visible compiled program) AND the direct API read both agree.
  274 |     await expect.poll(() => tabBPlan.digest(), { timeout: 90_000 }).toBe(staleDigest);
  275 |     await expect.poll(() => tabBDigest(), { timeout: 90_000 }).toBe(staleDigest);
  276 | 
  277 |     // One revision only. Weekly +30 may not change «N минут» (fake caps units at 360);
  278 |     // the digest is the consent identity.
  279 |     await page.getByRole('button', { name: 'Изменить', exact: true }).click();
  280 |     // The revise wire now carries whole hours 1..40 (M24-004 from server snapshot);
  281 |     // pick a changed in-range value instead of the old +30-minute delta.
  282 |     const hoursInput = page.locator('input[type="number"]').first();
  283 |     const currentHours = Number(await hoursInput.inputValue());
  284 |     const nextHours = currentHours < 5 ? currentHours + 1 : currentHours - 1;
  285 |     await hoursInput.fill(String(Math.max(1, nextHours)));
  286 |     await page.getByRole('button', { name: 'Применить изменения' }).click();
  287 |     await expect(
  288 |       page.getByRole('button', { name: 'Подтвердить и создать курс' }),
  289 |     ).toBeEnabled({ timeout: 120_000 });
  290 |     await expect.poll(() => tabAPlan.digest(), { timeout: 60_000 }).not.toBe(staleDigest);
  291 |     // A profile revision drafts against the new spec, then deterministic mapping
  292 |     // prepares the plan without an external map-provider call.
  293 |     expect(await countCalls('propose_plan_draft')).toBe(2);
  294 |     expect(await countCalls('propose_learning_map')).toBe(0);
  295 |     expect(await countCalls('materialize_unit')).toBe(0);
  296 | 
  297 |     // Click may wait if B is mid re-read (confirm unmounts while the new map compiles).
  298 |     await confirmB.click({ timeout: 120_000 });
  299 |     const outcome = await waitForTwoTabConfirmOutcome(tabB);
  300 |     if (outcome === 'stale_refused') {
  301 |       expect(await countCalls('materialize_unit')).toBe(0);
  302 |       await expect.poll(() => tabBDigest(), { timeout: 60_000 }).not.toBe(staleDigest);
  303 |       const confirmFresh = tabB.getByRole('button', { name: 'Подтвердить и создать курс' });
  304 |       await expect(confirmFresh).toBeEnabled({ timeout: 60_000 });
```