# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-journey.spec.ts >> finance: response-form journey with wrong values, conflict and completion
- Location: e2e/course-generation-journey.spec.ts:173:1

# Error details

```
Error: expect(received).toEqual(expected) // deep equality

- Expected  - 2
+ Received  + 2

  Object {
-   "completed_units": 2,
-   "percent": 100,
+   "completed_units": 0,
+   "percent": 0,
    "total_units": 2,
  }
```

# Page snapshot

```yaml
- generic [ref=e3]:
  - banner [ref=e4]:
    - button "Меню ученика" [ref=e7]
  - generic [ref=e13]:
    - region "Course workspace bar" [ref=e14]:
      - generic [ref=e15]:
        - button "Back to Course Library" [ref=e16] [cursor=pointer]:
          - generic [ref=e20]: Library
        - generic [ref=e22]:
          - 'heading "Learning plan: Personal budget and reserve" [level=1] [ref=e25]'
          - generic [ref=e26]: sha256:245727243de291db73a5cf458eca32f449155a4ba249f0b42f0b863a08ba900b
      - 'status "Server status: Local server active" [ref=e28]':
        - generic [ref=e30]: Local server active
    - generic [ref=e31]:
      - complementary "Course Curriculum" [ref=e32]:
        - generic [ref=e33]:
          - generic [ref=e34]: Overview
          - generic [ref=e40]:
            - generic [ref=e41]:
              - generic [ref=e42]: 2 of 2 units complete
              - generic [ref=e43]: 100%
            - 'progressbar "Course progress: 2 of 2 units (100%)" [ref=e44]'
          - generic [ref=e46]: generated-4512daa649b0452eb7197a065aa45d16
        - navigation "Curriculum Navigation" [ref=e48]:
          - heading "Curriculum" [level=2] [ref=e49]
          - generic [ref=e51]:
            - generic [ref=e52]: "Stage: Personal budget and reserve"
            - list [ref=e53]:
              - listitem [ref=e54]:
                - 'button "Practice: budget balance (completed)" [ref=e55] [cursor=pointer]':
                  - generic [ref=e59]: "Practice: budget balance"
              - listitem [ref=e60]:
                - 'button "Transfer: new budget (completed)" [ref=e61] [cursor=pointer]':
                  - generic [ref=e65]: "Transfer: new budget"
      - main "Lesson Workspace" [ref=e66]:
        - generic [ref=e67]:
          - tablist "Unit workspace panels" [ref=e68]:
            - tab "Theory" [ref=e69] [cursor=pointer]
            - tab "Code" [selected] [ref=e72] [cursor=pointer]
          - tabpanel "Code" [ref=e77]:
            - region "Ваше решение" [ref=e78]:
              - generic [ref=e79]:
                - generic [ref=e80]:
                  - heading "Ваше решение" [level=2] [ref=e81]
                  - paragraph [ref=e82]: Your solution
                - generic [ref=e83]: Сохранено
              - generic [ref=e85]:
                - generic [ref=e86]:
                  - generic [ref=e87]: Free monthly balance
                  - generic [ref=e88]:
                    - textbox "Free monthly balance" [ref=e89]: "24000"
                    - generic [ref=e90]: money units / month
                - generic [ref=e91]:
                  - generic [ref=e92]: Months to the reserve target
                  - generic [ref=e93]:
                    - textbox "Months to the reserve target" [ref=e94]: "3"
                    - generic [ref=e95]: months
                - generic [ref=e96]:
                  - heading "Как проверим" [level=3] [ref=e97]
                  - list [ref=e98]:
                    - listitem [ref=e99]: Compute the free monthly balance and reserve months for a new budget (tolerance 0).
              - button "Открыть файл" [ref=e104] [cursor=pointer]
      - complementary "Support" [ref=e106]:
        - region "Support" [ref=e107]:
          - tablist "Support tools" [ref=e108]:
            - tab "Rubric" [selected] [ref=e109]
            - tab "AI Mentor" [disabled] [ref=e113]
          - paragraph [ref=e116]: AI Mentor is unavailable for this cold transfer
          - tabpanel "Rubric" [ref=e117]:
            - region "Rubric" [ref=e118]:
              - generic [ref=e119]:
                - heading "Rubric" [level=2] [ref=e123]
                - generic "1 of 1 criteria met" [ref=e124]: 1 / 1 Met
              - status [ref=e125]:
                - generic [ref=e126]: PASSED
                - paragraph [ref=e130]: unit u2 check completed
              - list "Unit Criteria" [ref=e131]:
                - listitem [ref=e132]:
                  - generic [ref=e137]:
                    - generic [ref=e138]: Compute the free monthly balance and reserve months for a new budget (tolerance 0).
                    - generic [ref=e139]: Passed
                  - paragraph [ref=e141]: "Evidence: command check-u2 passed"
              - button "Next unit" [ref=e142] [cursor=pointer]
```

# Test source

```ts
  141 |   // Repeat Start returned the SAME working copy and the authored identity is unchanged.
  142 |   expect(view['short_identity']).toBe(identity);
  143 |   expect(view['catalog_path']).toBe(workingCopy);
  144 | 
  145 |   // The learner solution is still in the same working copy's editor: reselect the
  146 |   // practice unit explicitly (the settled re-entry does not preselect a unit) before
  147 |   // reading its Code tab.
  148 |   await openUnitByTitle(page, 'Practice: double');
  149 |   await expect(page.getByRole('heading', { name: 'Practice: double(n) cases' })).toBeVisible({ timeout: 15_000 });
  150 |   await openCodeTab(page);
  151 |   await expect(page.locator('.cm-content')).toContainText('n * 2');
  152 | 
  153 |   // A10: turn the generation provider off; the finished material still works.
  154 |   // Same settled-card wait as above: this site used to click straight after the reload,
  155 |   // racing the catalog refetch (and previously, lock-contention error projections).
  156 |   await fakeControl('off');
  157 |   await page.reload();
  158 |   await gotoLearnerApp(page);
  159 |   await openSettledCompletedCourse(page, title);
  160 |   await grantTrustIfPresent(page);
  161 |   await openUnitByTitle(page, 'Practice: double');
  162 |   await expect(page.getByRole('heading', { name: 'Practice: double(n) cases' })).toBeVisible({ timeout: 15_000 });
  163 |   await openCodeTab(page);
  164 |   await expect(page.locator('.cm-content')).toContainText('n * 2');
  165 |   await fakeControl('on');
  166 | 
  167 |   // Provider call discipline: exactly one map proposal and one materialize per unit.
  168 |   expect(await countCalls('propose_learning_map')).toBe(1);
  169 |   expect(await countCalls('materialize_unit', 'u1')).toBe(1);
  170 |   expect(await countCalls('materialize_unit', 'u2')).toBe(1);
  171 | });
  172 | 
  173 | test('finance: response-form journey with wrong values, conflict and completion', async ({
  174 |   page,
  175 | }) => {
  176 |   await resetGenerationFake();
  177 |   const title = subjectCourseTitle('finance');
  178 |   await driveCreation(page, 'finance');
  179 |   await startSavedCourse(page);
  180 | 
  181 |   // Wizard screens must fit 1440/390/320 without page-wide horizontal scroll (A17).
  182 |   for (const width of [1440, 390, 320]) {
  183 |     await page.setViewportSize({ width, height: 900 });
  184 |     await assertNoHorizontalOverflow(page);
  185 |   }
  186 |   await page.setViewportSize({ width: 1440, height: 900 });
  187 | 
  188 |   await expect(page.getByRole('heading', { name: 'Practice: budget balance' })).toBeVisible();
  189 |   await openCodeTab(page);
  190 |   await expect(page.getByRole('heading', { name: 'Ваше решение' })).toBeVisible({ timeout: 30_000 });
  191 | 
  192 |   // A14: bad values are saved, then the check reports unmet criteria.
  193 |   await fillNumberField(page, 'balance', '15000');
  194 |   await fillNumberField(page, 'months', '2');
  195 |   await waitForFormSaved(page);
  196 |   expect(await checkWork(page)).toBe('UNMET CRITERIA');
  197 | 
  198 |   // Good values pass.
  199 |   await fillNumberField(page, 'balance', '14000');
  200 |   await fillNumberField(page, 'months', '3');
  201 |   await waitForFormSaved(page);
  202 |   expect(await checkWork(page)).toBe('PASSED');
  203 | 
  204 |   // A16 on the transfer unit: an external edit conflicts with the form autosave, the
  205 |   // input is kept, Check waits for the save, and the resolved save passes.
  206 |   await goNextUnit(page);
  207 |   await expect(page.getByRole('heading', { name: 'Transfer: new budget' })).toBeVisible();
  208 |   await openCodeTab(page);
  209 |   await expect(page.getByRole('heading', { name: 'Ваше решение' })).toBeVisible({ timeout: 30_000 });
  210 | 
  211 |   const courseRef = await findCourseRef(page, title);
  212 |   const view = await courseView(page, courseRef);
  213 |   const units = ((view['stages'] as Array<{ units: Array<Record<string, unknown>> }>) ?? []).flatMap(
  214 |     (stage) => stage.units,
  215 |   );
  216 |   const transfer = units.find((unit) => unit['unit_id'] === 'u2');
  217 |   if (transfer === undefined || typeof transfer['unit_ref'] !== 'string') {
  218 |     throw new Error('transfer unit not found in the workspace view');
  219 |   }
  220 |   await externalEditAnswers(page, courseRef, transfer['unit_ref'] as string);
  221 | 
  222 |   // The closed D8 autosave only persists a fully-valid form, and the transfer starts
  223 |   // from the neutral unanswers (months is required and unfilled), so BOTH fields must
  224 |   // be locally complete for the edit to debounce into a save attempt that then hits
  225 |   // the external editor's bumped revision as a 409 conflict (A16).
  226 |   await fillNumberField(page, 'balance', '999');
  227 |   await fillNumberField(page, 'months', '3');
  228 |   await expect(page.getByText('Вернуться к моему вводу')).toBeVisible({ timeout: 30_000 });
  229 |   // The typed input survives the conflict on screen.
  230 |   await expect(page.locator('#balance')).toHaveValue('999');
  231 |   // Check is blocked while the conflict is unresolved.
  232 |   await expect(page.getByRole('button', { name: 'Check work' })).toBeDisabled();
  233 | 
  234 |   await page.getByRole('button', { name: 'Вернуться к моему вводу' }).click();
  235 |   await fillNumberField(page, 'balance', '24000');
  236 |   await fillNumberField(page, 'months', '3');
  237 |   await waitForFormSaved(page);
  238 |   expect(await checkWork(page)).toBe('PASSED');
  239 | 
  240 |   const finalView = await courseView(page, courseRef);
> 241 |   expect(finalView['progress_summary']).toEqual({
      |                                         ^ Error: expect(received).toEqual(expected) // deep equality
  242 |     total_units: 2,
  243 |     completed_units: 2,
  244 |     percent: 100,
  245 |   });
  246 | 
  247 |   // The response-form workspace fits the narrow viewport too.
  248 |   await page.setViewportSize({ width: 390, height: 900 });
  249 |   await assertNoHorizontalOverflow(page);
  250 |   await page.setViewportSize({ width: 1440, height: 900 });
  251 | 
  252 |   // One axe pass over the form workspace (A17).
  253 |   const axe = await new AxeBuilder({ page }).withTags(['wcag2a', 'wcag2aa']).analyze();
  254 |   expect(axe.violations.filter((violation) => violation.impact === 'serious' || violation.impact === 'critical')).toEqual([]);
  255 | });
  256 | 
  257 | test('management: keyboard-only creation and completion (A15/A17)', async ({ page }) => {
  258 |   // No per-journey reset here: the closing dedupe assertion reads the fake's accumulated
  259 |   // call log across all three journeys (python/finance/managements = one propose each).
  260 |   const description = subjectDescription('management');
  261 |   const title = subjectCourseTitle('management');
  262 | 
  263 |   // Per-journey dedupe (CG21-121): every earlier journey resets the generation fake (finance
  264 |   // resets at its own start, wiping python's log), so the absolute call log is never the sum of
  265 |   // the three journeys. Snapshot it and assert this journey's delta is exactly one of each
  266 |   // operation — the zero-duplicate rule without depending on other tests' reset behavior.
  267 |   const callsBeforeJourney = (await generationCalls()).calls.length;
  268 | 
  269 |   await gotoLearnerApp(page);
  270 |   await expect(page.getByRole('button', { name: 'Создать курс' })).toBeVisible({
  271 |     timeout: 30_000,
  272 |   });
  273 |   // Keyboard-only wizard: every step is reached and confirmed with Tab/Enter only.
  274 |   await page.keyboard.press('Tab');
  275 |   await page.getByRole('button', { name: 'Создать курс' }).focus();
  276 |   await page.keyboard.press('Enter');
  277 |   await expect(page.getByRole('heading', { name: 'Чему хотите научиться?' })).toBeVisible();
  278 |   await page.fill('#wizard-goal', description);
  279 |   await page.getByRole('button', { name: 'Продолжить' }).focus();
  280 |   await page.keyboard.press('Enter');
  281 | 
  282 |   const confirmProfile = page.getByRole('button', { name: 'Показать программу' });
  283 |   // Enabled means the controller can actually issue the action (confirm_profile offered, or real
  284 |   // corrections exist): the keyboard Enter below can never be a silent no-op.
  285 |   await expect(confirmProfile).toBeEnabled({ timeout: 60_000 });
  286 |   // The step heading receives focus after each step change (focus management).
  287 |   await expect(page.locator('#wizard-h1')).toBeFocused();
  288 |   await confirmProfile.focus();
  289 |   await page.keyboard.press('Enter');
  290 | 
  291 |   const confirmPlan = page.getByRole('button', { name: 'Подтвердить и создать курс' });
  292 |   await expect(confirmPlan).toBeEnabled({ timeout: 90_000 });
  293 |   await confirmPlan.focus();
  294 |   await page.keyboard.press('Enter');
  295 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  296 |     timeout: 180_000,
  297 |   });
  298 | 
  299 |   await page.getByRole('button', { name: 'Начать обучение' }).focus();
  300 |   await page.keyboard.press('Enter');
  301 |   await grantTrustIfPresent(page);
  302 |   await waitForWorkspace(page);
  303 | 
  304 |   await openCodeTab(page);
  305 |   await expect(page.getByRole('heading', { name: 'Ваше решение' })).toBeVisible({ timeout: 30_000 });
  306 | 
  307 |   // The reflection text alone is never a criterion: the required choice is missing.
  308 |   await page.locator('#reflection').fill('I compared the mandatory work with the capacity.');
  309 |   await expect(page.getByText(/Для вашего разбора/)).toBeVisible();
  310 |   const check = page.getByRole('button', { name: 'Check work' });
  311 |   await expect(check).toBeDisabled();
  312 | 
  313 |   // Wrong plan first: mandatory coverage met but over capacity.
  314 |   await chooseOption(page, 'plan', 'abc');
  315 |   await waitForFormSaved(page);
  316 |   expect(await checkWork(page)).toBe('UNMET CRITERIA');
  317 | 
  318 |   // The feasible plan passes.
  319 |   await chooseOption(page, 'plan', 'ab');
  320 |   await waitForFormSaved(page);
  321 |   expect(await checkWork(page)).toBe('PASSED');
  322 | 
  323 |   await goNextUnit(page);
  324 |   await expect(page.getByRole('heading', { name: 'Transfer: new capacity' })).toBeVisible();
  325 |   await openCodeTab(page);
  326 |   await chooseOption(page, 'plan', 'de');
  327 |   await waitForFormSaved(page);
  328 |   expect(await checkWork(page)).toBe('PASSED');
  329 | 
  330 |   const courseRef = await findCourseRef(page, title);
  331 |   const view = await courseView(page, courseRef);
  332 |   expect(view['progress_summary']).toEqual({ total_units: 2, completed_units: 2, percent: 100 });
  333 | 
  334 |   // This journey added exactly one of each generation operation (no duplicates).
  335 |   const log = await generationCalls();
  336 |   console.log('FAKE-CALLS', JSON.stringify(log.calls));
  337 |   const journeyCalls = log.calls.slice(callsBeforeJourney) as Array<{
  338 |     kind: string;
  339 |     unit_id: string;
  340 |   }>;
  341 |   const journeyCount = (kind: string, unitId?: string) =>
```