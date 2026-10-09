# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-repair-loop.spec.ts >> one check failure repairs and completes the course
- Location: e2e/course-generation-repair-loop.spec.ts:32:1

# Error details

```
Error: waitForCalls(repair_unit >= 1) timed out; counts={"extract_interview":1,"propose_plan_draft":1,"materialize_unit":1}
```

# Page snapshot

```yaml
- generic [ref=e3]:
  - banner [ref=e4]:
    - button "Меню ученика" [ref=e7]
  - main [ref=e13]:
    - generic [ref=e14]:
      - button "В библиотеку" [ref=e15]
      - list "Этапы создания" [ref=e18]:
        - listitem [ref=e19]:
          - generic [ref=e20]: ✓
          - text: Цель
        - listitem [ref=e22]:
          - generic [ref=e23]: ✓
          - text: Параметры
        - listitem [ref=e25]:
          - generic [ref=e26]: ✓
          - text: Программа
        - listitem [ref=e28]:
          - generic [ref=e29]: "4"
          - text: Создание
      - paragraph [ref=e30]: Python functions checked by integer cases
    - generic [ref=e31]: Создание
    - generic [ref=e32]: Создание прервано. Готовые занятия сохранены
    - generic [ref=e33]:
      - alert [ref=e34]:
        - paragraph [ref=e35]: Создание прервано. Готовые занятия сохранены
        - paragraph [ref=e36]: Можно продолжить с незавершённого занятия. Подтверждать прежнюю программу повторно не нужно.
        - generic [ref=e37]:
          - button "Продолжить создание" [ref=e38] [cursor=pointer]
          - button "Изменить программу" [ref=e40] [cursor=pointer]
      - region [ref=e42]:
        - heading "Готовим ваш курс" [active] [level=1] [ref=e44]
        - generic [ref=e45]:
          - paragraph [ref=e46]: Создание остановлено
          - list [ref=e47]:
            - listitem [ref=e48]:
              - generic [ref=e49]: ✓
              - generic [ref=e50]: Уточняем цель
              - generic [ref=e51]: Готово
            - listitem [ref=e52]:
              - generic [ref=e53]: ✓
              - generic [ref=e54]: Составляем программу
              - generic [ref=e55]: Готово
            - listitem [ref=e56]:
              - generic [ref=e57]: ●
              - generic [ref=e58]: Готовим занятия
              - generic [ref=e59]: выполняется …
            - listitem [ref=e60]:
              - generic [ref=e61]: ○
              - generic [ref=e62]: Проверяем курс
              - generic [ref=e63]: ждёт
            - listitem [ref=e64]:
              - generic [ref=e65]: ○
              - generic [ref=e66]: Сохраняем курс
              - generic [ref=e67]: ждёт
        - paragraph [ref=e70]: Можно вернуться в библиотеку. Создание продолжится, пока приложение работает
```

# Test source

```ts
  123 |       page
  124 |         .getByRole('button', { name: 'Изменить', exact: true })
  125 |         .waitFor({ state: 'visible', timeout: 30_000 })
  126 |         .then(() => 'target' as const),
  127 |       page
  128 |         .getByRole('button', { name: 'Показать программу' })
  129 |         .waitFor({ state: 'visible', timeout: 30_000 })
  130 |         .then(() => 'target' as const),
  131 |       page
  132 |         .getByRole('heading', { name: 'Ваш курс готов' })
  133 |         .waitFor({ state: 'visible', timeout: 30_000 })
  134 |         .then(() => 'wrong' as const),
  135 |     ]).catch(() => 'none' as const);
  136 |     if (right === 'target') {
  137 |       return; // The wizard re-opened the draft at the target phase.
  138 |     }
  139 |     // Wrong draft (e.g. a published twin from an earlier spec): return to the library
  140 |     // and try the next candidate; opening a draft only reads the server state.
  141 |     await page.getByRole('button', { name: 'В библиотеку' }).click();
  142 |     await page
  143 |       .getByRole('button', { name: new RegExp(`^Open draft: ${skill.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')}`) })
  144 |       .first()
  145 |       .waitFor({ state: 'visible', timeout: 15_000 });
  146 |   }
  147 |   if (Date.now() > deadline) {
  148 |     throw new Error(`no ${phase} draft card for ${skill} opened at the phase within the deadline`);
  149 |   }
  150 |   throw new Error(`no ${phase} draft card for ${skill} opened at the phase`);
  151 | }
  152 | 
  153 | /** POST one loopback-only fake control command; fails the test on a non-2xx answer. */
  154 | export async function fakeControl(action: string): Promise<void> {
  155 |   const response = await fetch(`${GENERATION_FAKE_URL}/__control/${action}`, { method: 'POST' });
  156 |   if (!response.ok) {
  157 |     throw new Error(`generation fake control /__control/${action} failed with ${response.status}`);
  158 |   }
  159 | }
  160 | 
  161 | /** Clear the fake's call log, pending faults and shelf permissions (test isolation). */
  162 | export async function resetGenerationFake(): Promise<void> {
  163 |   await fakeControl('reset');
  164 | }
  165 | 
  166 | /** Arm one native-check rejection on the next materialize for a unit; repair then passes. */
  167 | export async function armCheckFailOnce(unitId: string): Promise<void> {
  168 |   await fakeControl(`check_fail_once/${unitId}`);
  169 | }
  170 | 
  171 | /** Arm native-check rejection on every materialize and repair for a unit. */
  172 | export async function armCheckFailAlways(unitId: string): Promise<void> {
  173 |   await fakeControl(`check_fail_always/${unitId}`);
  174 | }
  175 | 
  176 | export interface FakeCall {
  177 |   kind: string;
  178 |   operation_id: string;
  179 |   unit_id: string;
  180 |   at: number;
  181 | }
  182 | 
  183 | export interface FakeCallLog {
  184 |   calls: FakeCall[];
  185 |   counts: Record<string, number>;
  186 | }
  187 | 
  188 | /** Read the provider call log the fake records for every generation operation. */
  189 | export async function generationCalls(): Promise<FakeCallLog> {
  190 |   const response = await fetch(`${GENERATION_FAKE_URL}/__calls`);
  191 |   if (!response.ok) {
  192 |     throw new Error(`generation fake /__calls failed with ${response.status}`);
  193 |   }
  194 |   return (await response.json()) as FakeCallLog;
  195 | }
  196 | 
  197 | /** Count provider calls of one kind, optionally restricted to one unit id. */
  198 | export async function countCalls(kind: string, unitId?: string): Promise<number> {
  199 |   const log = await generationCalls();
  200 |   return log.calls.filter(
  201 |     (call) => call.kind === kind && (unitId === undefined || call.unit_id === unitId),
  202 |   ).length;
  203 | }
  204 | 
  205 | /**
  206 |  * Boundedly wait until the fake-provider call log reaches a minimum count of one kind.
  207 |  * Provider calls run behind the async queue worker, so a counter read immediately after
  208 |  * an accepted mutation legitimately lags the queue.
  209 |  */
  210 | export async function waitForCalls(
  211 |   page: Page,
  212 |   kind: string,
  213 |   minLength: number,
  214 |   timeoutMs = 60_000,
  215 | ): Promise<void> {
  216 |   void page;
  217 |   try {
  218 |     await expect
  219 |       .poll(() => countCalls(kind), { timeout: timeoutMs, intervals: [200, 500, 1_000] })
  220 |       .toBeGreaterThanOrEqual(minLength);
  221 |   } catch (error) {
  222 |     const log = await generationCalls();
> 223 |     throw new Error(
      |           ^ Error: waitForCalls(repair_unit >= 1) timed out; counts={"extract_interview":1,"propose_plan_draft":1,"materialize_unit":1}
  224 |       `waitForCalls(${kind} >= ${minLength}) timed out; counts=${JSON.stringify(log.counts)}`,
  225 |       { cause: error },
  226 |     );
  227 |   }
  228 | }
  229 | 
  230 | /**
  231 |  * Wait until every authoring session in the shared catalog scope holds NO non-terminal
  232 |  * operation (test isolation, CG21-121 flake fix).
  233 |  *
  234 |  * The e2e server is one global process with one authoring database for the whole run:
  235 |  * the operation queue is scope-wide FIFO (`peek_queued_operation` picks the OLDEST
  236 |  * queued row, the singleton scope-owner lease guards a claimed one), so a
  237 |  * claimed-or-queued operation leaked by a previous test would run against the NEXT
  238 |  * test's worker cycles and could also consume the shared fake's armed
  239 |  * `fail-next`/`drop-next` fault. Draining the scope at test boundaries removes that
  240 |  * cross-test interference without restarting the server: the poll reads the real
  241 |  * projection (`active_operation_ref` is null exactly when the operation is terminal)
  242 |  * and each state change is driven by the real worker.
  243 |  */
  244 | export async function drainAuthoringQueue(page: Page, timeoutMs = 60_000): Promise<void> {
  245 |   await expect
  246 |     .poll(
  247 |       async () => {
  248 |         const shelf = await apiGet(page, '/api/v1/authoring/sessions?page_size=50');
  249 |         const entries = (shelf['sessions'] ?? []) as Array<{ session_ref?: string }>;
  250 |         const refs = entries
  251 |           .map((entry) => entry['session_ref'])
  252 |           .filter((ref): ref is string => typeof ref === 'string');
  253 |         for (const ref of refs) {
  254 |           const view = await apiGet(page, `/api/v1/authoring/sessions/${ref}`);
  255 |           if (view['active_operation_ref'] != null) {
  256 |             return 'active';
  257 |           }
  258 |         }
  259 |         return 'drained';
  260 |       },
  261 |       { timeout: timeoutMs, intervals: [200, 500, 1_000] },
  262 |     )
  263 |     .toBe('drained');
  264 | }
  265 | 
  266 | // -- wizard driving -----------------------------------------------------------
  267 | 
  268 | /** Open the generator from the library and wait for the goal step. */
  269 | export async function openGenerator(page: Page): Promise<void> {
  270 |   await gotoLearnerApp(page);
  271 |   const create = page.getByRole('button', { name: 'Создать курс' });
  272 |   await expect(create).toBeVisible({ timeout: 30_000 });
  273 |   await create.click();
  274 |   await expect(page.getByRole('heading', { name: 'Чему хотите научиться?' })).toBeVisible({
  275 |     timeout: 30_000,
  276 |   });
  277 | }
  278 | 
  279 | /** Type the subject description and continue to the profile step. */
  280 | export async function submitGoal(page: Page, description: string): Promise<void> {
  281 |   await page.fill('#wizard-goal', description);
  282 |   await page.getByRole('button', { name: 'Продолжить' }).click();
  283 | }
  284 | 
  285 | /**
  286 |  * Wait until the extracted profile is complete and confirm it unchanged (the digest path).
  287 |  * A01: the extracted skill and outcome are shown for confirmation; nothing else is asked.
  288 |  *
  289 |  * LF-141 flip: the confirmation itself enqueues the `propose_plan_draft`, the worker runs
  290 |  * it and the post-draft gate builds the map deterministically, so after «Показать
  291 |  * программу» the wizard waits in the EPIC-25 draft program window
  292 |  * (`draft-version-badge`) until the program is compiled — the consent CTA is the NEW
  293 |  * step, not the confirm output.
  294 |  */
  295 | export async function confirmProfileStep(page: Page, skill: string): Promise<void> {
  296 |   const cta = page.getByRole('button', { name: 'Показать программу' });
  297 |   await expect(page.locator('#wizard-skill')).toHaveValue(skill, { timeout: 60_000 });
  298 |   const language = page.locator('#wizard-language');
  299 |   if ((await language.inputValue()) === '' && !(await cta.isEnabled())) {
  300 |     await language.selectOption({ label: 'Русский' });
  301 |   }
  302 |   await expect(cta).toBeEnabled({ timeout: 60_000 });
  303 |   await cta.click();
  304 |   // The flip-enqueued draft ran: its clarification cards render while the server builds
  305 |   // the current-draft map without a generation-provider request.
  306 |   await expect(page.getByTestId('draft-version-badge')).toBeVisible({ timeout: 120_000 });
  307 |   await waitForCalls(page, 'propose_plan_draft', 1);
  308 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  309 |     timeout: 120_000,
  310 |   });
  311 | }
  312 | 
  313 | /**
  314 |  * Answer ONE draft clarification card through the real card contract: select the first
  315 |  * option of the first open card and Apply it exactly once. The 202 flow re-runs the plan
  316 |  * draft (`propose_plan_draft` >= 2) and the gate rebuilds the map for the moved spec
  317 |  * without a provider call.
  318 |  */
  319 | export async function answerFirstDraftQuestion(page: Page): Promise<void> {
  320 |   // Wait for the settled plan window first: between the draft commit and the
  321 |   // compiled program the controller auto-compiles the gated map, and that
  322 |   // draft→compiled branch change REMOUNTS the question cards and clears any
  323 |   // local selection (LF-146 slice B). Interacting before the settle races the
```