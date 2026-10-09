# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-journey.spec.ts >> python: full Q2 journey from description to completed transfer
- Location: e2e/course-generation-journey.spec.ts:81:1

# Error details

```
Error: expect(received).toBe(expected) // Object.is equality

Expected: "564f61b34d76e411"
Received: "sha256:b459ce1621a2c51166859c583e2da86c5ab28263db3ed20774d1a09c72bf9e6c"
```

# Page snapshot

```yaml
- generic [ref=f3e3]:
  - banner [ref=f3e4]:
    - button "Меню ученика" [ref=f3e7]
  - generic [ref=f3e13]:
    - region "Course workspace bar" [ref=f3e14]:
      - generic [ref=f3e15]:
        - button "Back to Course Library" [ref=f3e16] [cursor=pointer]:
          - generic [ref=f3e20]: Library
        - generic [ref=f3e22]:
          - 'heading "Learning plan: Python functions checked by integer cases" [level=1] [ref=f3e25]'
          - generic [ref=f3e26]: sha256:b459ce1621a2c51166859c583e2da86c5ab28263db3ed20774d1a09c72bf9e6c
      - 'status "Server status: Local server active" [ref=f3e28]':
        - generic [ref=f3e30]: Local server active
    - generic [ref=f3e31]:
      - complementary "Course Curriculum" [ref=f3e32]:
        - generic [ref=f3e33]:
          - generic [ref=f3e34]: Overview
          - generic [ref=f3e40]:
            - generic [ref=f3e41]:
              - generic [ref=f3e42]: 2 of 2 units complete
              - generic [ref=f3e43]: 100%
            - 'progressbar "Course progress: 2 of 2 units (100%)" [ref=f3e44]'
          - generic [ref=f3e46]: generated-828996ac065242d2831a1aada43a8997
        - navigation "Curriculum Navigation" [ref=f3e48]:
          - heading "Curriculum" [level=2] [ref=f3e49]
          - generic [ref=f3e51]:
            - generic [ref=f3e52]: Foundations of Python functions checked by integer cases
            - list [ref=f3e53]:
              - listitem [ref=f3e54]:
                - button "Foundations of Python functions checked by integer cases (completed)" [ref=f3e55] [cursor=pointer]:
                  - generic [ref=f3e59]: Foundations of Python functions checked by integer cases
              - listitem [ref=f3e60]:
                - button "Foundations of Python functions checked by integer cases (completed)" [active] [ref=f3e61] [cursor=pointer]:
                  - generic [ref=f3e65]: Foundations of Python functions checked by integer cases
      - main "Lesson Workspace" [ref=f3e66]:
        - generic [ref=f3e67]:
          - tablist "Unit workspace panels" [ref=f3e68]:
            - tab "Theory" [selected] [ref=f3e69] [cursor=pointer]
            - tab "Code" [ref=f3e72] [cursor=pointer]
          - tabpanel "Theory" [ref=f3e77]:
            - article "Lesson" [ref=f3e78]:
              - generic [ref=f3e79]:
                - generic [ref=f3e80]: Lesson Content
                - heading "Foundations of Python functions checked by integer cases" [level=2] [ref=f3e84]
                - generic [ref=f3e85]: "Goal: Explain and apply the core ideas of Python functions checked by integer cases."
              - generic [ref=f3e91]:
                - 'heading "Transfer: counting pairs in a new context" [level=1] [ref=f3e92]'
                - heading "New context" [level=2] [ref=f3e93]
                - paragraph [ref=f3e94]: A club stores its tournament entries as a Python list of names. For the draw the organizer needs the number of distinct unordered pairs of entries.
                - heading "Your task" [level=2] [ref=f3e95]
                - paragraph [ref=f3e96]:
                  - text: Implement
                  - code [ref=f3e97]: total_pairs(items)
                  - text: in
                  - code [ref=f3e98]: learner/artifact.py
                  - text: ": for a list"
                  - code [ref=f3e99]: items
                  - text: "it returns the number of unordered pairs of distinct positions. An empty list and a single-element list have zero pairs. The published integer cases decide the result. This unit has no hints: apply the same method you used before, in this new context."
                - heading "Worked example" [level=2] [ref=f3e100]
                - paragraph [ref=f3e101]:
                  - text: See
                  - link "the worked example" [ref=f3e102] [cursor=pointer]:
                    - /url: ../../examples/lf158-group-01-practice-shape.md
                  - text: for the shape of a complete attempt.
      - complementary "Support" [ref=f3e103]:
        - region "Support" [ref=f3e104]:
          - tablist "Support tools" [ref=f3e105]:
            - tab "Rubric" [selected] [ref=f3e106]
            - tab "AI Mentor" [disabled] [ref=f3e110]
          - paragraph [ref=f3e113]: AI Mentor is unavailable for this cold transfer
          - tabpanel "Rubric" [ref=f3e114]:
            - region "Rubric" [ref=f3e115]:
              - heading "Rubric" [level=2] [ref=f3e120]
              - list "Unit Criteria" [ref=f3e121]:
                - listitem [ref=f3e122]:
                  - generic [ref=f3e125]:
                    - generic [ref=f3e126]: Explain and apply the core ideas of Python functions checked by integer cases.
                    - generic [ref=f3e127]: Required
              - button "Next unit" [ref=f3e128] [cursor=pointer]
```

# Test source

```ts
  131 |   await page.setViewportSize({ width: 1440, height: 900 });
  132 | 
  133 |   // Answer one module question; the current draft map is rebuilt without provider I/O.
  134 |   const planWatcher = watchSessionPlanDigest(page);
  135 |   // The single-Create proof: count the /plan-confirmation POSTs this page sends.
  136 |   let confirmationPosts = 0;
  137 |   page.on('request', (request) => {
  138 |     if (
  139 |       request.method() === 'POST' &&
  140 |       /\/api\/v1\/authoring\/sessions\/[^/]+\/plan-confirmation$/.test(
  141 |         new URL(request.url()).pathname,
  142 |       )
  143 |     ) {
  144 |       confirmationPosts += 1;
  145 |     }
  146 |   });
  147 |   await answerFirstDraftQuestion(page);
  148 |   await expect(
  149 |     page.getByRole('button', { name: 'Подтвердить и создать курс' }),
  150 |   ).toBeEnabled({ timeout: 120_000 });
  151 |   const answeredLog = await generationCalls();
  152 |   const answeredCount = (kind: string) =>
  153 |     answeredLog.calls
  154 |       .slice(callsBefore)
  155 |       .filter((call) => call.kind === kind)
  156 |       .length;
  157 |   expect(answeredCount('propose_plan_draft')).toBe(2);
  158 |   expect(answeredCount('propose_learning_map')).toBe(0);
  159 |   expect(answeredCount('materialize_unit')).toBe(0);
  160 | 
  161 |   // The visible compiled program matches the consented plan digest.
  162 |   const confirmedDigest = await confirmPlanStep(page);
  163 |   await expect.poll(() => planWatcher.digest(), { timeout: 30_000 }).toBe(confirmedDigest);
  164 |   await waitForSavedCourse(page);
  165 |   // Each mapped unit materializes once after the single consent.
  166 |   const log = await generationCalls();
  167 |   const journeyCalls = log.calls.slice(callsBefore) as Array<{ kind: string; unit_id: string }>;
  168 |   const journeyCount = (kind: string, unitId?: string) =>
  169 |     journeyCalls.filter(
  170 |       (call) => call.kind === kind && (unitId === undefined || call.unit_id === unitId),
  171 |     ).length;
  172 |   expect(confirmationPosts).toBe(1); // the single Create: exactly one consent POST
  173 |   expect(journeyCount('propose_plan_draft')).toBe(2);
  174 |   expect(journeyCount('propose_learning_map')).toBe(0);
  175 |   expect(journeyCount('materialize_unit', PRACTICE_UNIT_ID)).toBe(1);
  176 |   expect(journeyCount('materialize_unit', TRANSFER_UNIT_ID)).toBe(1);
  177 |   // The saved screen names the library receipt and the single consent already happened.
  178 |   await expect(page.getByText('Сохранён в библиотеке')).toBeVisible();
  179 | 
  180 |   await startSavedCourse(page);
  181 |   // Read the preparation, then practice in the editor.
  182 |   await expect(page.getByRole('heading', { name: 'Practice: write and check double(n)' })).toBeVisible();
  183 |   const courseRef = await findCourseRef(page, title);
  184 |   const startView = await courseView(page, courseRef);
  185 |   const identity = startView['short_identity'];
  186 |   const workingCopy = startView['catalog_path'];
  187 |   expect(typeof identity).toBe('string');
  188 |   expect(typeof workingCopy).toBe('string');
  189 |   await openCodeTab(page);
  190 |   await expect(page.locator('.cm-content')).toContainText('LEARNER-TODO');
  191 | 
  192 |   // This generated path checks structure; fixed-template journeys cover wrong answers.
  193 |   await fillEditor(page, PYTHON_RIGHT);
  194 |   await waitCodeSaved(page);
  195 |   expect(await checkWork(page)).toBe('PASSED');
  196 | 
  197 |   // Transfer has a separate scaffold and no previous solution.
  198 |   await goNextUnit(page);
  199 |   await expect(page.getByRole('heading', { name: 'Transfer: counting pairs in a new context' })).toBeVisible({ timeout: 30_000 });
  200 |   await openCodeTab(page);
  201 |   await expect(page.locator('.cm-content')).toContainText('LEARNER-TODO');
  202 |   await expect(page.locator('.cm-content')).not.toContainText('n * 2');
  203 |   await fillEditor(page, TRANSFER_RIGHT);
  204 |   await waitCodeSaved(page);
  205 |   expect(await checkWork(page)).toBe('PASSED');
  206 | 
  207 |   // Verify completion in the progress projection.
  208 |   await waitForCourseProgress(page, courseRef, {
  209 |     total_units: 2,
  210 |     completed_units: 2,
  211 |     percent: 100,
  212 |   });
  213 |   let view = await courseView(page, courseRef);
  214 | 
  215 |   await page.reload();
  216 |   await gotoLearnerApp(page);
  217 |   // Reopen only after the settled completion projection appears.
  218 |   await openSettledCompletedCourse(page, title);
  219 |   await grantTrustIfPresent(page);
  220 |   // The settled re-entry has no preselected unit.
  221 |   const sourceTitle = `Foundations of ${subjectDescription('python').split('\n')[0]}`;
  222 |   await openUnitByTitle(page, sourceTitle, 1);
  223 |   await expect(page.getByRole('heading', { name: 'Transfer: counting pairs in a new context' })).toBeVisible({ timeout: 30_000 });
  224 |   await waitForCourseProgress(page, courseRef, {
  225 |     total_units: 2,
  226 |     completed_units: 2,
  227 |     percent: 100,
  228 |   });
  229 |   view = await courseView(page, courseRef);
  230 |   // Repeat Start returns the same working copy.
> 231 |   expect(view['short_identity']).toBe(identity);
      |                                  ^ Error: expect(received).toBe(expected) // Object.is equality
  232 |   expect(view['catalog_path']).toBe(workingCopy);
  233 | 
  234 |   // Reopen the practice unit to verify the learner artifact survived reload.
  235 |   await openUnitByTitle(page, sourceTitle);
  236 |   await expect(page.getByRole('heading', { name: 'Practice: write and check double(n)' })).toBeVisible({ timeout: 15_000 });
  237 |   await openCodeTab(page);
  238 |   await expect(page.locator('.cm-content')).toContainText('n * 2');
  239 | 
  240 |   // Finished material remains available while the generation provider is off.
  241 |   await fakeControl('off');
  242 |   await page.reload();
  243 |   await gotoLearnerApp(page);
  244 |   await openSettledCompletedCourse(page, title);
  245 |   await grantTrustIfPresent(page);
  246 |   await openUnitByTitle(page, sourceTitle);
  247 |   await expect(page.getByRole('heading', { name: 'Practice: write and check double(n)' })).toBeVisible({ timeout: 15_000 });
  248 |   await openCodeTab(page);
  249 |   await expect(page.locator('.cm-content')).toContainText('n * 2');
  250 |   await fakeControl('on');
  251 | 
  252 |   // Verify provider call discipline.
  253 |   expect(await countCalls('propose_plan_draft')).toBe(2);
  254 |   expect(await countCalls('propose_learning_map')).toBe(0);
  255 |   expect(await countCalls('materialize_unit', PRACTICE_UNIT_ID)).toBe(1);
  256 |   expect(await countCalls('materialize_unit', TRANSFER_UNIT_ID)).toBe(1);
  257 | });
  258 | 
  259 | test('finance: create, submit structural attempts, progress and reload', async ({ page }) => {
  260 |   await resetGenerationFake();
  261 |   const title = subjectCourseTitle('finance');
  262 |   expect(await driveCreation(page, 'finance')).toBe(title);
  263 |   await startSavedCourse(page);
  264 | 
  265 |   const courseRef = await findCourseRef(page, title);
  266 |   const practiceTitle = await courseUnitTitle(page, courseRef, PRACTICE_UNIT_ID);
  267 |   const transferTitle = await courseUnitTitle(page, courseRef, TRANSFER_UNIT_ID);
  268 |   await expect(page.getByRole('heading', { name: practiceTitle, exact: true })).toBeVisible();
  269 |   await openCodeTab(page);
  270 |   await expect(page.locator('.cm-content')).toBeVisible();
  271 |   for (const width of [1440, 390, 320]) {
  272 |     await page.setViewportSize({ width, height: 900 });
  273 |     await assertNoHorizontalOverflow(page);
  274 |   }
  275 |   await page.setViewportSize({ width: 1440, height: 900 });
  276 |   await submitStructuralAttempt(page);
  277 |   await goNextUnit(page);
  278 |   await expect(page.getByRole('heading', { name: transferTitle, exact: true })).toBeVisible();
  279 |   await openCodeTab(page);
  280 |   await submitStructuralAttempt(page);
  281 |   await waitForCourseProgress(page, courseRef, {
  282 |     total_units: 2,
  283 |     completed_units: 2,
  284 |     percent: 100,
  285 |   });
  286 |   const axe = await new AxeBuilder({ page }).withTags(['wcag2a', 'wcag2aa']).analyze();
  287 |   expect(
  288 |     axe.violations.filter(
  289 |       (violation) => violation.impact === 'serious' || violation.impact === 'critical',
  290 |     ),
  291 |   ).toEqual([]);
  292 | 
  293 |   await page.reload();
  294 |   await gotoLearnerApp(page);
  295 |   await openSettledCompletedCourse(page, title);
  296 |   await grantTrustIfPresent(page);
  297 |   await openUnitByTitle(page, practiceTitle);
  298 |   await expect(page.getByRole('heading', { name: practiceTitle, exact: true })).toBeVisible();
  299 |   await openCodeTab(page);
  300 |   await expect(page.locator('.cm-content')).toContainText('attempt = "learner work"');
  301 | });
  302 | 
  303 | test('management: keyboard creation, structural submission, progression and reload', async ({ page }) => {
  304 |   await resetGenerationFake();
  305 |   const description = subjectDescription('management');
  306 |   const title = subjectCourseTitle('management');
  307 |   const callsBeforeJourney = (await generationCalls()).calls.length;
  308 | 
  309 |   await gotoLearnerApp(page);
  310 |   await expect(page.getByRole('button', { name: 'Создать курс' })).toBeVisible({
  311 |     timeout: 30_000,
  312 |   });
  313 |   // Keyboard-only wizard: every step is reached and confirmed with Tab/Enter only.
  314 |   await page.keyboard.press('Tab');
  315 |   await page.getByRole('button', { name: 'Создать курс' }).focus();
  316 |   await page.keyboard.press('Enter');
  317 |   await expect(page.getByRole('heading', { name: 'Чему хотите научиться?' })).toBeVisible();
  318 |   await page.fill('#wizard-goal', description);
  319 |   await page.getByRole('button', { name: 'Продолжить' }).focus();
  320 |   await page.keyboard.press('Enter');
  321 | 
  322 |   const confirmProfile = page.getByRole('button', { name: 'Показать программу' });
  323 |   // Enabled means the controller can actually issue the action (confirm_profile offered, or real
  324 |   // corrections exist): the keyboard Enter below can never be a silent no-op.
  325 |   await expect(confirmProfile).toBeEnabled({ timeout: 60_000 });
  326 |   // The step heading receives focus after each step change (focus management).
  327 |   await expect(page.locator('#wizard-h1')).toBeFocused();
  328 |   await confirmProfile.focus();
  329 |   await page.keyboard.press('Enter');
  330 | 
  331 |   const confirmPlan = page.getByRole('button', { name: 'Подтвердить и создать курс' });
```