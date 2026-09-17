# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-recovery.spec.ts >> capabilities read failure: library notice, wizard guidance, library intact
- Location: e2e/course-generation-recovery.spec.ts:102:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByRole('heading', { name: 'AI для создания курсов пока не настроен' })
Expected: visible
Timeout: 30000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 30000ms
  - waiting for getByRole('heading', { name: 'AI для создания курсов пока не настроен' })

```

```yaml
- main:
  - alert: A newer build of the learner workspace is available. Please reload the page.
```

# Test source

```ts
  12  |   confirmProfileStep,
  13  |   countCalls,
  14  |   courseView,
  15  |   drainAuthoringQueue,
  16  |   driveToConfirmedPlan,
  17  |   fakeControl,
  18  |   generationCalls,
  19  |   openActiveDraft,
  20  |   openDraftByPhase,
  21  |   openGenerator,
  22  |   resetGenerationFake,
  23  |   startSavedCourse,
  24  |   subjectDescription,
  25  |   submitGoal,
  26  |   waitForCalls,
  27  |   waitForSavedCourseOrAlert,
  28  |   watchSessionPlanDigest,
  29  | } from './course-generation-support';
  30  | import { gotoLearnerApp } from './support';
  31  | 
  32  | const STALE_PLAN_COPY = 'Программа изменилась. Посмотрите свежую версию';
  33  | 
  34  | /** Materialize started only after the revised map; otherwise still waiting. */
  35  | async function twoTabConfirmOutcome(): Promise<'fresh_started' | 'pending'> {
  36  |   const log = await generationCalls();
  37  |   const maps = log.calls.filter((call) => call.kind === 'propose_learning_map');
  38  |   const mats = log.calls.filter((call) => call.kind === 'materialize_unit');
  39  |   const firstMat = mats[0];
  40  |   const secondMap = maps[1];
  41  |   if (firstMat !== undefined && (secondMap === undefined || firstMat.at <= secondMap.at)) {
  42  |     throw new Error(
  43  |       `stale plan was materialized; maps=${maps.length} firstMaterializeAt=${firstMat.at} secondMapAt=${secondMap?.at}`,
  44  |     );
  45  |   }
  46  |   if (secondMap !== undefined && firstMat !== undefined && firstMat.at > secondMap.at) {
  47  |     return 'fresh_started';
  48  |   }
  49  |   return 'pending';
  50  | }
  51  | 
  52  | async function waitForTwoTabConfirmOutcome(
  53  |   tabB: Page,
  54  | ): Promise<'stale_refused' | 'fresh_started'> {
  55  |   await expect
  56  |     .poll(
  57  |       async () => {
  58  |         const noticed =
  59  |           (await tabB.getByRole('alert').filter({ hasText: STALE_PLAN_COPY }).count()) > 0;
  60  |         const started = await twoTabConfirmOutcome();
  61  |         if (started === 'fresh_started') return 'fresh_started';
  62  |         if (noticed && (await countCalls('materialize_unit')) === 0) return 'stale_refused';
  63  |         return 'pending';
  64  |       },
  65  |       { timeout: 90_000, intervals: [200, 500, 1_000] },
  66  |     )
  67  |     .toMatch(/^(stale_refused|fresh_started)$/);
  68  |   const noticed =
  69  |     (await tabB.getByRole('alert').filter({ hasText: STALE_PLAN_COPY }).count()) > 0;
  70  |   const started = await twoTabConfirmOutcome();
  71  |   if (started === 'fresh_started') return 'fresh_started';
  72  |   if (noticed && (await countCalls('materialize_unit')) === 0) return 'stale_refused';
  73  |   throw new Error('two-tab confirm settled then reverted to pending');
  74  | }
  75  | 
  76  | async function assertMaterializeFollowsRevisedMap(): Promise<void> {
  77  |   const log = await generationCalls();
  78  |   const maps = log.calls.filter((call) => call.kind === 'propose_learning_map');
  79  |   const mats = log.calls.filter((call) => call.kind === 'materialize_unit');
  80  |   const firstMat = mats[0];
  81  |   const secondMap = maps[1];
  82  |   expect(maps).toHaveLength(2);
  83  |   expect(firstMat).toBeDefined();
  84  |   expect(secondMap).toBeDefined();
  85  |   if (firstMat === undefined || secondMap === undefined) return;
  86  |   expect(firstMat.at).toBeGreaterThan(secondMap.at);
  87  | }
  88  | 
  89  | test.describe.configure({ mode: 'serial' });
  90  | 
  91  | // Scope drain at every boundary (CG21-121 flake fix): one global server shares the
  92  | // authoring queue and the fake's fault arming across tests, so a leaked non-terminal
  93  | // operation from a prior test would either steal worker cycles (FIFO scope queue) or
  94  | // consume this test's armed fault. The hooks keep every test on a quiet scope.
  95  | test.beforeEach(async ({ page }) => {
  96  |   await drainAuthoringQueue(page);
  97  | });
  98  | test.afterEach(async ({ page }) => {
  99  |   await drainAuthoringQueue(page);
  100 | });
  101 | 
  102 | test('capabilities read failure: library notice, wizard guidance, library intact', async ({
  103 |   page,
  104 | }) => {
  105 |   await resetGenerationFake();
  106 |   // Bounded read-failure injection on the capabilities projection only (never a faked
  107 |   // success): the app must treat the unknown state conservatively.
  108 |   await page.route('**/api/v1/authoring/capabilities', (route) => route.abort());
  109 |   await gotoLearnerApp(page);
  110 |   await expect(
  111 |     page.getByRole('heading', { name: 'AI для создания курсов пока не настроен' }),
> 112 |   ).toBeVisible({ timeout: 30_000 });
      |     ^ Error: expect(locator).toBeVisible() failed
  113 |   // The library stays intact: existing courses and templates remain listed.
  114 |   await expect(page.getByRole('button', { name: 'Создать курс' })).toBeVisible();
  115 | 
  116 |   await page.getByRole('button', { name: 'Создать курс' }).click();
  117 |   await expect(page.getByText('AI для создания курсов пока не настроен')).toBeVisible();
  118 |   // The goal CTA stays disabled: no creation is promised without a working provider.
  119 |   await page.fill('#wizard-goal', 'Anything at all');
  120 |   await expect(page.getByRole('button', { name: 'Продолжить' })).toBeDisabled();
  121 | 
  122 |   await page.unroute('**/api/v1/authoring/capabilities');
  123 |   await page.reload();
  124 |   await gotoLearnerApp(page);
  125 |   await expect(
  126 |     page.getByRole('heading', { name: 'AI для создания курсов пока не настроен' }),
  127 |   ).toHaveCount(0);
  128 | });
  129 | 
  130 | test('provider transport failure mid-creation: interrupted guidance, resume, library intact', async ({
  131 |   page,
  132 | }) => {
  133 |   await resetGenerationFake();
  134 |   await fakeControl('fail-next/materialize_unit');
  135 |   const subject = 'finance' as const;
  136 |   await driveToConfirmedPlan(page, subject);
  137 |   await confirmPlanStep(page);
  138 |   // The first materialize call failed with a transport error: the wizard shows the
  139 |   // interrupted recovery with the no-new-consent note and the continue action.
  140 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  141 |     timeout: 60_000,
  142 |   });
  143 |   await expect(page.getByText('Подтверждать прежнюю программу повторно не нужно')).toBeVisible();
  144 | 
  145 |   // The library stays intact while the wizard sits in the failure phase.
  146 |   await page.getByRole('button', { name: 'В библиотеку' }).click();
  147 |   await expect(page.getByRole('heading', { name: 'Ваше обучение' })).toBeVisible();
  148 |   await gotoLearnerApp(page);
  149 |   // Resume through the draft shelf: the failure state is durable server-side.
  150 |   await openActiveDraft(page);
  151 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  152 |     timeout: 60_000,
  153 |   });
  154 | 
  155 |   await page.getByRole('button', { name: 'Продолжить создание' }).click();
  156 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  157 |     timeout: 180_000,
  158 |   });
  159 | 
  160 |   // The failed unit was re-materialized once; the second unit ran exactly once.
  161 |   expect(await countCalls('materialize_unit', 'u1')).toBe(2);
  162 |   expect(await countCalls('materialize_unit', 'u2')).toBe(1);
  163 |   expect(await countCalls('propose_learning_map')).toBe(1);
  164 | });
  165 | 
  166 | test('interrupted mid-flight: completed checkpoints are not regenerated', async ({ page }) => {
  167 |   await resetGenerationFake();
  168 |   // u1 materializes fine; the u2 provider call is killed mid-flight (connection drop).
  169 |   await fakeControl('drop-next/materialize_unit/u2');
  170 |   await driveToConfirmedPlan(page, 'management');
  171 |   await confirmPlanStep(page);
  172 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  173 |     timeout: 60_000,
  174 |   });
  175 | 
  176 |   await page.getByRole('button', { name: 'Продолжить создание' }).click();
  177 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  178 |     timeout: 180_000,
  179 |   });
  180 | 
  181 |   // Checkpoint retention: u1 was materialized exactly once (its checkpoint was kept),
  182 |   // only the interrupted u2 ran a second time. No regeneration of finished units.
  183 |   expect(await countCalls('materialize_unit', 'u1')).toBe(1);
  184 |   expect(await countCalls('materialize_unit', 'u2')).toBe(2);
  185 |   expect(await countCalls('propose_learning_map')).toBe(1);
  186 | });
  187 | 
  188 | test('two tabs: stale plan is never materialized', async ({ page }) => {
  189 |   await resetGenerationFake();
  190 |   const description = subjectDescription('python');
  191 |   await openGenerator(page);
  192 |   await submitGoal(page, description);
  193 |   await confirmProfileStep(page, description.split('\n')[0]);
  194 |   // Tab A stays at the plan preview WITHOUT confirming: the first consent must come
  195 |   // from Tab B, so a stale attempt cannot race an already-running materialize.
  196 |   const tabAPlan = watchSessionPlanDigest(page);
  197 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  198 |     timeout: 90_000,
  199 |   });
  200 |   await expect.poll(() => tabAPlan.digest(), { timeout: 90_000 }).not.toBe('');
  201 |   const staleDigest = tabAPlan.digest();
  202 | 
  203 |   // Tab B attaches to the same durable draft. Its 4s poll may absorb A's revision
  204 |   // before confirm; both outcomes are valid as long as the old plan is not generated.
  205 |   const tabB = await page.context().newPage();
  206 |   const tabBPlan = watchSessionPlanDigest(tabB);
  207 |   try {
  208 |     await gotoLearnerApp(tabB);
  209 |     await openDraftByPhase(tabB, 'python', 'plan_review');
  210 |     const confirmB = tabB.getByRole('button', { name: 'Подтвердить и создать курс' });
  211 |     await expect(confirmB).toBeEnabled({ timeout: 90_000 });
  212 |     await expect.poll(() => tabBPlan.digest(), { timeout: 90_000 }).toBe(staleDigest);
```