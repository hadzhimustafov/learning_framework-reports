# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-recovery.spec.ts >> two tabs: stale plan confirmation gets 409, fresh plan needs new consent
- Location: e2e/course-generation-recovery.spec.ts:130:1

# Error details

```
Error: expect(received).toBeGreaterThanOrEqual(expected)

Expected: >= 1
Received:    0

Call Log:
- Timeout 60000ms exceeded while waiting on the predicate
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
          - text: Уточнения
        - listitem [ref=e25]:
          - generic [ref=e26]: "3"
          - text: Программа
        - listitem [ref=e28]:
          - generic [ref=e29]: "4"
          - text: Создание
      - paragraph [ref=e30]: Python functions checked by integer cases
    - generic [ref=e31]: Программа
    - generic [ref=e32]: Готовим занятия
    - region [ref=e33]:
      - generic [ref=e34]:
        - paragraph [ref=e35]: Программа
        - 'heading "Learning plan: Python functions checked by integer cases" [level=1] [ref=e36]'
        - paragraph [ref=e37]: Write and verify small pure functions against published integer cases.
      - generic [ref=e38]:
        - generic [ref=e39]:
          - generic [ref=e40]:
            - paragraph [ref=e41]: Как проверим
            - list [ref=e42]:
              - listitem [ref=e43]: "u1: implementation evidence via published criteria"
              - listitem [ref=e44]: "u2: transfer evidence via published criteria"
          - generic [ref=e45]:
            - paragraph [ref=e46]: Что не проверяем
            - list [ref=e47]:
              - listitem [ref=e48]: Free-form explanations are not graded as proof of solution quality.
              - listitem [ref=e49]: No income, investment or financial outcomes are promised or checked.
              - listitem [ref=e50]: Transfer is checked in one neighboring context only.
          - paragraph [ref=e51]: Новая задача без подсказок
          - button "Изменить" [ref=e52] [cursor=pointer]
        - complementary [ref=e54]:
          - paragraph [ref=e55]: Ваш маршрут
          - generic [ref=e56]:
            - term [ref=e57]: Занятия
            - definition [ref=e58]: "2"
            - term [ref=e59]: Этапы
            - definition [ref=e60]: "1"
            - term [ref=e61]: Учебное время
            - definition [ref=e62]: 720 минут
            - term [ref=e63]: Уровень
            - definition [ref=e64]: Уверенно знаю основы
            - term [ref=e65]: Язык
            - definition [ref=e66]: ru
          - separator [ref=e67]
          - paragraph [ref=e68]: Подготовим все занятия и сохраним курс в вашей библиотеке
          - button "Подтвердить и создать курс" [ref=e69] [cursor=pointer]
      - button "Назад" [ref=e71] [cursor=pointer]
```

# Test source

```ts
  122 |       .first()
  123 |       .waitFor({ state: 'visible', timeout: Math.min(timeoutMs, 60_000) })
  124 |       .catch(() => {
  125 |         throw new Error(`no shelf card for skill ${skill}`);
  126 |       });
  127 |   }
  128 |   for (let index = 0; index < (await candidates.count()); index += 1) {
  129 |     await candidates.nth(index).click();
  130 |     // The reopened wizard needs a moment to load its authoritative session; race the
  131 |     // plan-preview markers against the known wrong-draft screen ('Ваш курс готов' of a
  132 |     // published twin). A resumed plan_review draft opens directly at the plan preview
  133 |     // ('Изменить' visible); the profile-step CTA ('Показать программу') is the
  134 |     // fallback when the wizard re-enters at the clarifications step.
  135 |     const right = await Promise.race([
  136 |       page
  137 |         .getByRole('button', { name: 'Изменить', exact: true })
  138 |         .waitFor({ state: 'visible', timeout: 30_000 })
  139 |         .then(() => 'target' as const),
  140 |       page
  141 |         .getByRole('button', { name: 'Показать программу' })
  142 |         .waitFor({ state: 'visible', timeout: 30_000 })
  143 |         .then(() => 'target' as const),
  144 |       page
  145 |         .getByRole('heading', { name: 'Ваш курс готов' })
  146 |         .waitFor({ state: 'visible', timeout: 30_000 })
  147 |         .then(() => 'wrong' as const),
  148 |     ]).catch(() => 'none' as const);
  149 |     if (right === 'target') {
  150 |       return; // The wizard re-opened the draft at the target phase.
  151 |     }
  152 |     // Wrong draft (e.g. a published twin from an earlier spec): return to the library
  153 |     // and try the next candidate; opening a draft only reads the server state.
  154 |     await page.getByRole('button', { name: 'В библиотеку' }).click();
  155 |     await page
  156 |       .getByRole('button', { name: new RegExp(`^Open draft: ${skill.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')}`) })
  157 |       .first()
  158 |       .waitFor({ state: 'visible', timeout: 15_000 });
  159 |   }
  160 |   if (Date.now() > deadline) {
  161 |     throw new Error(`no ${phase} draft card for ${skill} opened at the phase within the deadline`);
  162 |   }
  163 |   throw new Error(`no ${phase} draft card for ${skill} opened at the phase`);
  164 | }
  165 | 
  166 | /** POST one loopback-only fake control command; fails the test on a non-2xx answer. */
  167 | export async function fakeControl(action: string): Promise<void> {
  168 |   const response = await fetch(`${GENERATION_FAKE_URL}/__control/${action}`, { method: 'POST' });
  169 |   if (!response.ok) {
  170 |     throw new Error(`generation fake control /__control/${action} failed with ${response.status}`);
  171 |   }
  172 | }
  173 | 
  174 | /** Clear the fake's call log, pending faults and shelf permissions (test isolation). */
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
  220 |   await expect
  221 |     .poll(() => countCalls(kind), { timeout: timeoutMs, intervals: [200, 500, 1_000] })
> 222 |     .toBeGreaterThanOrEqual(minLength);
      |      ^ Error: expect(received).toBeGreaterThanOrEqual(expected)
  223 | }
  224 | 
  225 | /**
  226 |  * Wait until every authoring session in the shared catalog scope holds NO non-terminal
  227 |  * operation (test isolation, CG21-121 flake fix).
  228 |  *
  229 |  * The e2e server is one global process with one authoring database for the whole run:
  230 |  * the operation queue is scope-wide FIFO (`peek_queued_operation` picks the OLDEST
  231 |  * queued row, the singleton scope-owner lease guards a claimed one), so a
  232 |  * claimed-or-queued operation leaked by a previous test would run against the NEXT
  233 |  * test's worker cycles and could also consume the shared fake's armed
  234 |  * `fail-next`/`drop-next` fault. Draining the scope at test boundaries removes that
  235 |  * cross-test interference without restarting the server: the poll reads the real
  236 |  * projection (`active_operation_ref` is null exactly when the operation is terminal)
  237 |  * and each state change is driven by the real worker.
  238 |  */
  239 | export async function drainAuthoringQueue(page: Page, timeoutMs = 60_000): Promise<void> {
  240 |   await expect
  241 |     .poll(
  242 |       async () => {
  243 |         const shelf = await apiGet(page, '/api/v1/authoring/sessions?page_size=50');
  244 |         const entries = (shelf['sessions'] ?? []) as Array<{ session_ref?: string }>;
  245 |         const refs = entries
  246 |           .map((entry) => entry['session_ref'])
  247 |           .filter((ref): ref is string => typeof ref === 'string');
  248 |         for (const ref of refs) {
  249 |           const view = await apiGet(page, `/api/v1/authoring/sessions/${ref}`);
  250 |           if (view['active_operation_ref'] != null) {
  251 |             return 'active';
  252 |           }
  253 |         }
  254 |         return 'drained';
  255 |       },
  256 |       { timeout: timeoutMs, intervals: [200, 500, 1_000] },
  257 |     )
  258 |     .toBe('drained');
  259 | }
  260 | 
  261 | // -- wizard driving -----------------------------------------------------------
  262 | 
  263 | /** Open the generator from the library and wait for the goal step. */
  264 | export async function openGenerator(page: Page): Promise<void> {
  265 |   await gotoLearnerApp(page);
  266 |   const create = page.getByRole('button', { name: 'Создать курс' });
  267 |   await expect(create).toBeVisible({ timeout: 30_000 });
  268 |   await create.click();
  269 |   await expect(page.getByRole('heading', { name: 'Чему хотите научиться?' })).toBeVisible({
  270 |     timeout: 30_000,
  271 |   });
  272 | }
  273 | 
  274 | /** Type the subject description and continue to the profile step. */
  275 | export async function submitGoal(page: Page, description: string): Promise<void> {
  276 |   await page.fill('#wizard-goal', description);
  277 |   await page.getByRole('button', { name: 'Продолжить' }).click();
  278 | }
  279 | 
  280 | /**
  281 |  * Wait until the extracted profile is complete and confirm it unchanged (the digest path).
  282 |  * A01: the extracted skill and outcome are shown for confirmation; nothing else is asked.
  283 |  */
  284 | export async function confirmProfileStep(page: Page, skill: string): Promise<void> {
  285 |   const cta = page.getByRole('button', { name: 'Показать программу' });
  286 |   await expect(cta).toBeEnabled({ timeout: 60_000 });
  287 |   await expect(page.locator('#wizard-skill')).toHaveValue(skill, { timeout: 60_000 });
  288 |   await cta.click();
  289 | }
  290 | 
  291 | /** Wait for the compiled plan and confirm it once (the single consent point). */
  292 | export async function confirmPlanStep(page: Page): Promise<void> {
  293 |   const cta = page.getByRole('button', { name: 'Подтвердить и создать курс' });
  294 |   await expect(cta).toBeEnabled({ timeout: 90_000 });
  295 |   await cta.click();
  296 | }
  297 | 
  298 | /** Wait for the wizard to reach the saved-course screen. */
  299 | export async function waitForSavedCourse(page: Page): Promise<void> {
  300 |   await expect(page.getByRole('heading', { name: 'Ваш курс готов' })).toBeVisible({
  301 |     timeout: 180_000,
  302 |   });
  303 | }
  304 | 
  305 | /**
  306 |  * Wait for the saved-course heading after consent, failing on a visible recovery alert.
  307 |  *
  308 |  * Used when a 180s heading timeout would hide the actual failure: the worker call log
  309 |  * must first show materialization started, then the heading or an alert decides.
  310 |  */
  311 | export async function waitForSavedCourseOrAlert(page: Page, timeoutMs = 60_000): Promise<void> {
  312 |   const saved = page.getByRole('heading', { name: 'Ваш курс готов' });
  313 |   try {
  314 |     await expect(saved).toBeVisible({ timeout: timeoutMs });
  315 |   } catch (error) {
  316 |     const alerts = await page.getByRole('alert').allTextContents();
  317 |     const detail = alerts.map((text) => text.trim()).filter((text) => text.length > 0);
  318 |     throw new Error(
  319 |       `saved heading missing after ${timeoutMs}ms; alerts: ${detail.join(' | ') || 'none'}`,
  320 |       { cause: error },
  321 |     );
  322 |   }
```