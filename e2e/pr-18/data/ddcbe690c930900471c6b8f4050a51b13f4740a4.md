# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-recovery.spec.ts >> two tabs: stale plan confirmation gets 409, fresh plan needs new consent
- Location: e2e/course-generation-recovery.spec.ts:130:1

# Error details

```
Error: expect(locator).not.toHaveText(expected) failed

Locator:  locator('dd').filter({ hasText: /\d+\s+минут/ }).first()
Expected: not "720 минут"
Received: "720 минут"
Timeout:  60000ms

Call log:
  - Expect "not toHaveText" with timeout 60000ms
  - waiting for locator('dd').filter({ hasText: /\d+\s+минут/ }).first()
    121 × locator resolved to <dd class="font-medium text-slate-900">720 минут</dd>
        - unexpected value "720 минут"

```

```yaml
- definition: 720 минут
```

# Test source

```ts
  112 |   await driveToConfirmedPlan(page, 'management');
  113 |   await confirmPlanStep(page);
  114 |   await expect(page.getByRole('alert').getByText('Создание прервано. Готовые занятия сохранены')).toBeVisible({
  115 |     timeout: 60_000,
  116 |   });
  117 | 
  118 |   await page.getByRole('button', { name: 'Продолжить создание' }).click();
  119 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  120 |     timeout: 180_000,
  121 |   });
  122 | 
  123 |   // Checkpoint retention: u1 was materialized exactly once (its checkpoint was kept),
  124 |   // only the interrupted u2 ran a second time. No regeneration of finished units.
  125 |   expect(await countCalls('materialize_unit', 'u1')).toBe(1);
  126 |   expect(await countCalls('materialize_unit', 'u2')).toBe(2);
  127 |   expect(await countCalls('propose_learning_map')).toBe(1);
  128 | });
  129 | 
  130 | test('two tabs: stale plan confirmation gets 409, fresh plan needs new consent', async ({
  131 |   page,
  132 | }) => {
  133 |   await resetGenerationFake();
  134 |   const description = subjectDescription('python');
  135 |   await openGenerator(page);
  136 |   await submitGoal(page, description);
  137 |   await confirmProfileStep(page, description.split('\n')[0]);
  138 |   // Tab A stays at the plan preview WITHOUT confirming: the first consent must come
  139 |   // from the fresh-plan path, so the stale attempt can not race an active operation.
  140 | 
  141 |   // Tab B attaches to the same durable draft from the library shelf; the wizard re-opens
  142 |   // directly at this draft's plan preview, so the fresh-plan consent lives here.
  143 |   const tabB = await page.context().newPage();
  144 |   await gotoLearnerApp(tabB);
  145 |   await openDraftByPhase(tabB, 'python', 'plan_review');
  146 |   const confirmB = tabB.getByRole('button', { name: 'Подтвердить и создать курс' });
  147 |   await expect(confirmB).toBeEnabled({ timeout: 90_000 });
  148 |   // Pin the preview Tab B will replace after 409: the confirm CTA stays enabled on the
  149 |   // stale digest until readSession lands, and clicking it then just 409s again.
  150 |   const tabBMinutes = tabB.locator('dd').filter({ hasText: /\d+\s+минут/ }).first();
  151 |   await expect(tabBMinutes).toBeVisible();
  152 |   const staleMinutes = (await tabBMinutes.textContent()) ?? '';
  153 | 
  154 |   // Tab A revises the plan: a new version invalidates the preview both tabs hold.
  155 |   // The revise wire carries the profile-projection digest (PlanReviseForm), so the
  156 |   // server gate accepts it and the durable session version rises once.
  157 |   // latestPythonSessionVersion sees only python-skill sessions, but the pinned version
  158 |   // must belong to THIS draft (a retained sibling draft could dominate the skill-wide
  159 |   // max for the whole run), so the revision gate pins the draft row itself.
  160 |   const pinnedVersions = await pythonSkillVersionSnapshot(page, 'python');
  161 |   let revisionCommitted = false;
  162 |   for (let attempt = 0; attempt < 4 && !revisionCommitted; attempt += 1) {
  163 |     await page.getByRole('button', { name: 'Изменить', exact: true }).click();
  164 |     const minutesInput = page.locator('input[type="number"]').first();
  165 |     const currentMinutes = await minutesInput.inputValue();
  166 |     await minutesInput.fill(String(Number(currentMinutes) + 30));
  167 |     await page.getByRole('button', { name: 'Применить изменения' }).click();
  168 |     await expect(
  169 |       page.getByRole('button', { name: 'Подтвердить и создать курс' }),
  170 |     ).toBeEnabled({ timeout: 120_000 });
  171 |     // CG21-121 flake fix: the confirm CTA re-enables as soon as the revised map is
  172 |     // proposed, while the shelf version projection can still lag it. Poll the version
  173 |     // rise for the same order the CTA wait allows before considering a retry legitimate.
  174 |     try {
  175 |       await expect
  176 |         .poll(
  177 |           async () =>
  178 |             versionSnapshotChanged(pinnedVersions, await pythonSkillVersionSnapshot(page, 'python')),
  179 |           { timeout: 60_000, intervals: [200, 500, 1_000] },
  180 |         )
  181 |         .toBe(true);
  182 |       revisionCommitted = true;
  183 |     } catch {
  184 |       // Shelf still lagging this attempt; retry the revision rather than a second map.
  185 |     }
  186 |   }
  187 |   expect(revisionCommitted, 'the plan revision must durably raise the session version').toBe(true);
  188 |   // Tab A still leaves its consent unused: neither tab has created anything yet.
  189 |   expect(await countCalls('materialize_unit')).toBe(0);
  190 | 
  191 |   // Tab B still holds the stale preview: its consent is refused as stale_plan (409).
  192 |   // Scope to the plan region: the page-level toast paragraph repeats the same
  193 |   // message, so a bare getByText stays a strict-mode violation.
  194 |   await confirmB.click();
  195 |   await expect(
  196 |     tabB
  197 |       .getByLabel('Learning plan: Python')
  198 |       .getByText('Программа изменилась. Посмотрите свежую версию'),
  199 |   ).toBeVisible({ timeout: 30_000 });
  200 |   // The revision-queued map proposal runs behind the async queue worker: wait for the
  201 |   // fake call log instead of racing it.
  202 |   await waitForCalls(page, 'propose_learning_map', 2);
  203 |   // No duplicate generation: exactly one extra map proposal after the revision.
  204 |   expect(await countCalls('propose_learning_map')).toBe(2);
  205 |   expect(await countCalls('materialize_unit')).toBe(0);
  206 | 
  207 |   // Tab B must consent to the NEW plan only. The stale confirm CTA is still enabled, so
  208 |   // wait for the recovery action and the refreshed time before clicking again.
  209 |   const openFreshPlan = tabB.getByRole('button', { name: 'Открыть программу' });
  210 |   await expect(openFreshPlan).toBeVisible({ timeout: 30_000 });
  211 |   await openFreshPlan.click();
> 212 |   await expect(tabBMinutes).not.toHaveText(staleMinutes, { timeout: 60_000 });
      |                                 ^ Error: expect(locator).not.toHaveText(expected) failed
  213 |   const confirmB2 = tabB.getByRole('button', { name: 'Подтвердить и создать курс' });
  214 |   await expect(confirmB2).toBeEnabled({ timeout: 60_000 });
  215 |   await confirmB2.click();
  216 |   await waitForCalls(tabB, 'materialize_unit', 1, 90_000);
  217 |   await waitForSavedCourseOrAlert(tabB, 120_000);
  218 |   expect(await countCalls('propose_learning_map')).toBe(2);
  219 |   expect(await countCalls('materialize_unit', 'u1')).toBe(1);
  220 |   expect(await countCalls('materialize_unit', 'u2')).toBe(1);
  221 |   await tabB.close();
  222 | });
  223 | 
  224 | test('failed publication: retry_publication republishes without regeneration', async ({
  225 |   page,
  226 | }) => {
  227 |   await resetGenerationFake();
  228 |   // The fake owns the disposable shelf directory: make the shelf write fail once.
  229 |   await fakeControl('shelf-readonly');
  230 |   await driveToConfirmedPlan(page, 'python');
  231 |   await confirmPlanStep(page);
  232 |   await expect(
  233 |     page.getByRole('alert').getByText('Занятия готовы, сохранить курс пока не удалось'),
  234 |   ).toBeVisible({ timeout: 120_000 });
  235 |   await expect(
  236 |     page.getByText('Повторим сохранение готового курса. Материалы заново создавать не нужно.'),
  237 |   ).toBeVisible();
  238 | 
  239 |   const materializeBefore = await countCalls('materialize_unit');
  240 |   expect(materializeBefore).toBe(2);
  241 | 
  242 |   try {
  243 |     await fakeControl('shelf-writable');
  244 |     await page.getByRole('button', { name: 'Повторить сохранение' }).click();
  245 |     await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  246 |       timeout: 180_000,
  247 |     });
  248 |     expect(await countCalls('materialize_unit')).toBe(materializeBefore);
  249 |     expect(await countCalls('propose_learning_map')).toBe(1);
  250 | 
  251 |     // The published course starts like any other template (A11 fork path). Deterministic
  252 |     // self-ownership: the SPA reaches its workspace through /api/v1/courses/<ref>, so the
  253 |     // ref this page actually operates on identifies THIS course (no status-title matching).
  254 |     const workspaceRefs = new Set<string>();
  255 |     page.on('request', (request) => {
  256 |       const match = request.url().match(/\/api\/v1\/courses\/([^/?#]+)/);
  257 |       if (match) workspaceRefs.add(decodeURIComponent(match[1]));
  258 |     });
  259 |     // The workspace loader itself is requested during startSavedCourse.
  260 |     await startSavedCourse(page);
  261 |     const uniqueRefs = [...workspaceRefs];
  262 |     expect(uniqueRefs, 'the fresh workspace requests must name exactly one course ref').toHaveLength(
  263 |       1,
  264 |     );
  265 |     const [courseRef] = uniqueRefs;
  266 |     const view = await courseView(page, courseRef);
  267 |     expect(view['progress_summary']).toEqual({
  268 |       total_units: 2,
  269 |       completed_units: 0,
  270 |       percent: 0,
  271 |     });
  272 |   } finally {
  273 |     await fakeControl('shelf-writable');
  274 |   }
  275 | });
  276 | 
```