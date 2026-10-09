# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-recovery.spec.ts >> two tabs: stale plan confirmation gets 409, fresh plan needs new consent
- Location: e2e/course-generation-recovery.spec.ts:131:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByRole('button', { name: 'Открыть программу' })
Expected: visible
Timeout: 30000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 30000ms
  - waiting for getByRole('button', { name: 'Открыть программу' })

```

```yaml
- banner:
  - button "Меню ученика"
- main:
  - button "В библиотеку"
  - list "Этапы создания":
    - listitem: ✓ Цель
    - listitem: ✓ Уточнения
    - listitem: 3 Программа
    - listitem: 4 Создание
  - paragraph: Python functions checked by integer cases
  - text: Программа Готовим занятия
  - 'region "Learning plan: Python functions checked by integer cases"':
    - paragraph: Программа
    - 'heading "Learning plan: Python functions checked by integer cases" [level=1]'
    - paragraph: Write and verify small pure functions against published integer cases.
    - paragraph: Как проверим
    - list:
      - listitem: "u1: implementation evidence via published criteria"
      - listitem: "u2: transfer evidence via published criteria"
    - paragraph: Что не проверяем
    - list:
      - listitem: Free-form explanations are not graded as proof of solution quality.
      - listitem: No income, investment or financial outcomes are promised or checked.
      - listitem: Transfer is checked in one neighboring context only.
    - paragraph: Новая задача без подсказок
    - button "Изменить"
    - complementary:
      - paragraph: Ваш маршрут
      - term: Занятия
      - definition: "2"
      - term: Этапы
      - definition: "1"
      - term: Учебное время
      - definition: 720 минут
      - term: Уровень
      - definition: Начинаю с нуля
      - term: Язык
      - definition: ru
      - separator
      - paragraph: Подготовим все занятия и сохраним курс в вашей библиотеке
      - button "Подтвердить и создать курс"
    - button "Назад"
```

# Test source

```ts
  113 |   await driveToConfirmedPlan(page, 'management');
  114 |   await confirmPlanStep(page);
  115 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  116 |     timeout: 60_000,
  117 |   });
  118 | 
  119 |   await page.getByRole('button', { name: 'Продолжить создание' }).click();
  120 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  121 |     timeout: 180_000,
  122 |   });
  123 | 
  124 |   // Checkpoint retention: u1 was materialized exactly once (its checkpoint was kept),
  125 |   // only the interrupted u2 ran a second time. No regeneration of finished units.
  126 |   expect(await countCalls('materialize_unit', 'u1')).toBe(1);
  127 |   expect(await countCalls('materialize_unit', 'u2')).toBe(2);
  128 |   expect(await countCalls('propose_learning_map')).toBe(1);
  129 | });
  130 | 
  131 | test('two tabs: stale plan confirmation gets 409, fresh plan needs new consent', async ({
  132 |   page,
  133 | }) => {
  134 |   await resetGenerationFake();
  135 |   const description = subjectDescription('python');
  136 |   await openGenerator(page);
  137 |   await submitGoal(page, description);
  138 |   await confirmProfileStep(page, description.split('\n')[0]);
  139 |   // Tab A stays at the plan preview WITHOUT confirming: the first consent must come
  140 |   // from the fresh-plan path, so the stale attempt can not race an active operation.
  141 | 
  142 |   // Tab B attaches to the same durable draft from the library shelf; the wizard re-opens
  143 |   // directly at this draft's plan preview, so the fresh-plan consent lives here.
  144 |   const tabB = await page.context().newPage();
  145 |   const tabBPlan = watchSessionPlanDigest(tabB);
  146 |   await gotoLearnerApp(tabB);
  147 |   await openDraftByPhase(tabB, 'python', 'plan_review');
  148 |   const confirmB = tabB.getByRole('button', { name: 'Подтвердить и создать курс' });
  149 |   await expect(confirmB).toBeEnabled({ timeout: 90_000 });
  150 |   // Pin the preview Tab B will replace after 409: the confirm CTA stays enabled on the
  151 |   // stale digest until readSession lands, and clicking it then just 409s again.
  152 |   // Estimated minutes can stay 720: the fake caps units at 360, so weekly +30 is not
  153 |   // a visible time change. The client digest is the consent identity.
  154 |   await expect.poll(() => tabBPlan.digest(), { timeout: 90_000 }).not.toBe('');
  155 |   const staleDigest = tabBPlan.digest();
  156 | 
  157 |   // Tab A revises the plan: a new version invalidates the preview both tabs hold.
  158 |   // The revise wire carries the profile-projection digest (PlanReviseForm), so the
  159 |   // server gate accepts it and the durable session version rises once.
  160 |   // latestPythonSessionVersion sees only python-skill sessions, but the pinned version
  161 |   // must belong to THIS draft (a retained sibling draft could dominate the skill-wide
  162 |   // max for the whole run), so the revision gate pins the draft row itself.
  163 |   const pinnedVersions = await pythonSkillVersionSnapshot(page, 'python');
  164 |   let revisionCommitted = false;
  165 |   for (let attempt = 0; attempt < 4 && !revisionCommitted; attempt += 1) {
  166 |     await page.getByRole('button', { name: 'Изменить', exact: true }).click();
  167 |     const minutesInput = page.locator('input[type="number"]').first();
  168 |     const currentMinutes = await minutesInput.inputValue();
  169 |     await minutesInput.fill(String(Number(currentMinutes) + 30));
  170 |     await page.getByRole('button', { name: 'Применить изменения' }).click();
  171 |     await expect(
  172 |       page.getByRole('button', { name: 'Подтвердить и создать курс' }),
  173 |     ).toBeEnabled({ timeout: 120_000 });
  174 |     // CG21-121 flake fix: the confirm CTA re-enables as soon as the revised map is
  175 |     // proposed, while the shelf version projection can still lag it. Poll the version
  176 |     // rise for the same order the CTA wait allows before considering a retry legitimate.
  177 |     try {
  178 |       await expect
  179 |         .poll(
  180 |           async () =>
  181 |             versionSnapshotChanged(pinnedVersions, await pythonSkillVersionSnapshot(page, 'python')),
  182 |           { timeout: 60_000, intervals: [200, 500, 1_000] },
  183 |         )
  184 |         .toBe(true);
  185 |       revisionCommitted = true;
  186 |     } catch {
  187 |       // Shelf still lagging this attempt; retry the revision rather than a second map.
  188 |     }
  189 |   }
  190 |   expect(revisionCommitted, 'the plan revision must durably raise the session version').toBe(true);
  191 |   // Tab A still leaves its consent unused: neither tab has created anything yet.
  192 |   expect(await countCalls('materialize_unit')).toBe(0);
  193 | 
  194 |   // Tab B still holds the stale preview: its consent is refused as stale_plan (409).
  195 |   // Scope to the plan region: the page-level toast paragraph repeats the same
  196 |   // message, so a bare getByText stays a strict-mode violation.
  197 |   await confirmB.click();
  198 |   await expect(
  199 |     tabB
  200 |       .getByLabel('Learning plan: Python')
  201 |       .getByText('Программа изменилась. Посмотрите свежую версию'),
  202 |   ).toBeVisible({ timeout: 30_000 });
  203 |   // The revision-queued map proposal runs behind the async queue worker: wait for the
  204 |   // fake call log instead of racing it.
  205 |   await waitForCalls(page, 'propose_learning_map', 2);
  206 |   // No duplicate generation: exactly one extra map proposal after the revision.
  207 |   expect(await countCalls('propose_learning_map')).toBe(2);
  208 |   expect(await countCalls('materialize_unit')).toBe(0);
  209 | 
  210 |   // Tab B must consent to the NEW plan only. The stale confirm CTA is still enabled, so
  211 |   // wait for the recovery action and this tab's re-read digest before clicking again.
  212 |   const openFreshPlan = tabB.getByRole('button', { name: 'Открыть программу' });
> 213 |   await expect(openFreshPlan).toBeVisible({ timeout: 30_000 });
      |                               ^ Error: expect(locator).toBeVisible() failed
  214 |   await openFreshPlan.click();
  215 |   await expect.poll(() => tabBPlan.digest(), { timeout: 60_000 }).not.toBe(staleDigest);
  216 |   const confirmB2 = tabB.getByRole('button', { name: 'Подтвердить и создать курс' });
  217 |   await expect(confirmB2).toBeEnabled({ timeout: 60_000 });
  218 |   await confirmB2.click();
  219 |   await waitForCalls(tabB, 'materialize_unit', 1, 90_000);
  220 |   await waitForSavedCourseOrAlert(tabB, 120_000);
  221 |   expect(await countCalls('propose_learning_map')).toBe(2);
  222 |   expect(await countCalls('materialize_unit', 'u1')).toBe(1);
  223 |   expect(await countCalls('materialize_unit', 'u2')).toBe(1);
  224 |   await tabB.close();
  225 | });
  226 | 
  227 | test('failed publication: retry_publication republishes without regeneration', async ({
  228 |   page,
  229 | }) => {
  230 |   await resetGenerationFake();
  231 |   // The fake owns the disposable shelf directory: make the shelf write fail once.
  232 |   await fakeControl('shelf-readonly');
  233 |   await driveToConfirmedPlan(page, 'python');
  234 |   await confirmPlanStep(page);
  235 |   await expect(
  236 |     page.getByRole('alert').getByText('Занятия готовы, сохранить курс пока не удалось'),
  237 |   ).toBeVisible({ timeout: 120_000 });
  238 |   await expect(
  239 |     page.getByText('Повторим сохранение готового курса. Материалы заново создавать не нужно.'),
  240 |   ).toBeVisible();
  241 | 
  242 |   const materializeBefore = await countCalls('materialize_unit');
  243 |   expect(materializeBefore).toBe(2);
  244 | 
  245 |   try {
  246 |     await fakeControl('shelf-writable');
  247 |     await page.getByRole('button', { name: 'Повторить сохранение' }).click();
  248 |     await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  249 |       timeout: 180_000,
  250 |     });
  251 |     expect(await countCalls('materialize_unit')).toBe(materializeBefore);
  252 |     expect(await countCalls('propose_learning_map')).toBe(1);
  253 | 
  254 |     // The published course starts like any other template (A11 fork path). Deterministic
  255 |     // self-ownership: the SPA reaches its workspace through /api/v1/courses/<ref>, so the
  256 |     // ref this page actually operates on identifies THIS course (no status-title matching).
  257 |     const workspaceRefs = new Set<string>();
  258 |     page.on('request', (request) => {
  259 |       const match = request.url().match(/\/api\/v1\/courses\/([^/?#]+)/);
  260 |       if (match) workspaceRefs.add(decodeURIComponent(match[1]));
  261 |     });
  262 |     // The workspace loader itself is requested during startSavedCourse.
  263 |     await startSavedCourse(page);
  264 |     const uniqueRefs = [...workspaceRefs];
  265 |     expect(uniqueRefs, 'the fresh workspace requests must name exactly one course ref').toHaveLength(
  266 |       1,
  267 |     );
  268 |     const [courseRef] = uniqueRefs;
  269 |     const view = await courseView(page, courseRef);
  270 |     expect(view['progress_summary']).toEqual({
  271 |       total_units: 2,
  272 |       completed_units: 0,
  273 |       percent: 0,
  274 |     });
  275 |   } finally {
  276 |     await fakeControl('shelf-writable');
  277 |   }
  278 | });
  279 | 
```