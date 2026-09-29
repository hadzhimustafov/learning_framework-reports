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
          - generic [ref=e68]:
            - paragraph [ref=e69]: Готово 0 из 2 занятий
            - progressbar "Готовые занятия" [ref=e70]
          - paragraph [ref=e71]: Foundations of Python functions checked by integer cases
          - paragraph [ref=e72]: Прошло 0:00
        - paragraph [ref=e73]: Можно вернуться в библиотеку. Создание продолжится, пока приложение работает
```

# Test source

```ts
  5   |  * real worker. The only faked surface is the external generation port, reached through
  6   |  * the loopback Python fake (`generation_fake.py`, proxied by `fakeOllama.mjs`) whose
  7   |  * control endpoints inject the bounded recovery faults and expose the provider call log.
  8   |  * No browser API is stubbed and no completion state is written directly.
  9   |  */
  10  | import { readFileSync } from 'node:fs';
  11  | import { dirname, join } from 'node:path';
  12  | import { fileURLToPath } from 'node:url';
  13  | import { expect, type Page } from '@playwright/test';
  14  | import { grantTrustIfPresent, gotoLearnerApp, waitForWorkspace } from './support';
  15  | 
  16  | const HERE = dirname(fileURLToPath(import.meta.url));
  17  | 
  18  | export const GENERATION_FAKE_URL = 'http://127.0.0.1:18001';
  19  | export const PRACTICE_UNIT_ID = 'lf158-group-01-practice';
  20  | export const TRANSFER_UNIT_ID = 'lf158-group-01-transfer';
  21  | 
  22  | /** The three Q3 subject descriptions, read from the Python fixture files (single source). */
  23  | export function subjectDescription(subject: 'python' | 'finance' | 'management'): string {
  24  |   return readFileSync(
  25  |     join(HERE, '../../../tests/fixtures/course_generation', subject, 'description.txt'),
  26  |     'utf8',
  27  |   ).trim();
  28  | }
  29  | 
  30  | /** Course title the compiler derives from one subject's confirmed profile. */
  31  | export function subjectCourseTitle(subject: 'python' | 'finance' | 'management'): string {
  32  |   return `Learning plan: ${subjectDescription(subject).split('\n')[0]}`;
  33  | }
  34  | 
  35  | /** POST one loopback-only fake control command; fails the test on a non-2xx answer. */
  36  | export async function fakeControl(action: string): Promise<void> {
  37  |   const response = await fetch(`${GENERATION_FAKE_URL}/__control/${action}`, { method: 'POST' });
  38  |   if (!response.ok) {
  39  |     throw new Error(`generation fake control /__control/${action} failed with ${response.status}`);
  40  |   }
  41  | }
  42  | 
  43  | /** Clear the fake's call log, pending faults and shelf permissions (test isolation). */
  44  | export async function resetGenerationFake(): Promise<void> {
  45  |   await fakeControl('reset');
  46  | }
  47  | 
  48  | /** Arm one native-check rejection on the next materialize for a unit; repair then passes. */
  49  | export async function armCheckFailOnce(unitId: string): Promise<void> {
  50  |   await fakeControl(`check_fail_once/${unitId}`);
  51  | }
  52  | 
  53  | /** Arm native-check rejection on every materialize and repair for a unit. */
  54  | export async function armCheckFailAlways(unitId: string): Promise<void> {
  55  |   await fakeControl(`check_fail_always/${unitId}`);
  56  | }
  57  | 
  58  | export interface FakeCall {
  59  |   kind: string;
  60  |   operation_id: string;
  61  |   unit_id: string;
  62  |   at: number;
  63  | }
  64  | 
  65  | export interface FakeCallLog {
  66  |   calls: FakeCall[];
  67  |   counts: Record<string, number>;
  68  | }
  69  | 
  70  | /** Read the provider call log the fake records for every generation operation. */
  71  | export async function generationCalls(): Promise<FakeCallLog> {
  72  |   const response = await fetch(`${GENERATION_FAKE_URL}/__calls`);
  73  |   if (!response.ok) {
  74  |     throw new Error(`generation fake /__calls failed with ${response.status}`);
  75  |   }
  76  |   return (await response.json()) as FakeCallLog;
  77  | }
  78  | 
  79  | /** Count provider calls of one kind, optionally restricted to one unit id. */
  80  | export async function countCalls(kind: string, unitId?: string): Promise<number> {
  81  |   const log = await generationCalls();
  82  |   return log.calls.filter(
  83  |     (call) => call.kind === kind && (unitId === undefined || call.unit_id === unitId),
  84  |   ).length;
  85  | }
  86  | 
  87  | /**
  88  |  * Boundedly wait until the fake-provider call log reaches a minimum count of one kind.
  89  |  * Provider calls run behind the async queue worker, so a counter read immediately after
  90  |  * an accepted mutation legitimately lags the queue.
  91  |  */
  92  | export async function waitForCalls(
  93  |   page: Page,
  94  |   kind: string,
  95  |   minLength: number,
  96  |   timeoutMs = 60_000,
  97  | ): Promise<void> {
  98  |   void page;
  99  |   try {
  100 |     await expect
  101 |       .poll(() => countCalls(kind), { timeout: timeoutMs, intervals: [200, 500, 1_000] })
  102 |       .toBeGreaterThanOrEqual(minLength);
  103 |   } catch (error) {
  104 |     const log = await generationCalls();
> 105 |     throw new Error(
      |           ^ Error: waitForCalls(repair_unit >= 1) timed out; counts={"extract_interview":1,"propose_plan_draft":1,"materialize_unit":1}
  106 |       `waitForCalls(${kind} >= ${minLength}) timed out; counts=${JSON.stringify(log.counts)}`,
  107 |       { cause: error },
  108 |     );
  109 |   }
  110 | }
  111 | 
  112 | /**
  113 |  * Wait until every authoring session in the shared catalog scope holds NO non-terminal
  114 |  * operation (test isolation, CG21-121 flake fix).
  115 |  *
  116 |  * The e2e server is one global process with one authoring database for the whole run:
  117 |  * the operation queue is scope-wide FIFO (`peek_queued_operation` picks the OLDEST
  118 |  * queued row, the singleton scope-owner lease guards a claimed one), so a
  119 |  * claimed-or-queued operation leaked by a previous test would run against the NEXT
  120 |  * test's worker cycles and could also consume the shared fake's armed
  121 |  * `fail-next`/`drop-next` fault. Draining the scope at test boundaries removes that
  122 |  * cross-test interference without restarting the server: the poll reads the real
  123 |  * projection (`active_operation_ref` is null exactly when the operation is terminal)
  124 |  * and each state change is driven by the real worker.
  125 |  */
  126 | export async function drainAuthoringQueue(page: Page, timeoutMs = 60_000): Promise<void> {
  127 |   await expect
  128 |     .poll(
  129 |       async () => {
  130 |         const shelf = await apiGet(page, '/api/v1/authoring/sessions?page_size=50');
  131 |         const entries = (shelf['sessions'] ?? []) as Array<{ session_ref?: string }>;
  132 |         const refs = entries
  133 |           .map((entry) => entry['session_ref'])
  134 |           .filter((ref): ref is string => typeof ref === 'string');
  135 |         for (const ref of refs) {
  136 |           const view = await apiGet(page, `/api/v1/authoring/sessions/${ref}`);
  137 |           if (view['active_operation_ref'] != null) {
  138 |             return 'active';
  139 |           }
  140 |         }
  141 |         return 'drained';
  142 |       },
  143 |       { timeout: timeoutMs, intervals: [200, 500, 1_000] },
  144 |     )
  145 |     .toBe('drained');
  146 | }
  147 | 
  148 | // -- wizard driving -----------------------------------------------------------
  149 | 
  150 | /** Open the generator from the library and wait for the goal step. */
  151 | export async function openGenerator(page: Page): Promise<void> {
  152 |   await gotoLearnerApp(page);
  153 |   const create = page.getByRole('button', { name: 'Создать курс' });
  154 |   await expect(create).toBeVisible({ timeout: 30_000 });
  155 |   await create.click();
  156 |   await expect(page.getByRole('heading', { name: 'Чему хотите научиться?' })).toBeVisible({
  157 |     timeout: 30_000,
  158 |   });
  159 | }
  160 | 
  161 | /** Type the subject description and continue to the profile step. */
  162 | export async function submitGoal(page: Page, description: string): Promise<void> {
  163 |   await page.fill('#wizard-goal', description);
  164 |   await page.getByRole('button', { name: 'Продолжить' }).click();
  165 | }
  166 | 
  167 | /**
  168 |  * Wait until the extracted profile is complete and confirm it unchanged (the digest path).
  169 |  * A01: the extracted skill and outcome are shown for confirmation; nothing else is asked.
  170 |  *
  171 |  * LF-141 flip: the confirmation itself enqueues the `propose_plan_draft`, the worker runs
  172 |  * it and the post-draft gate builds the map deterministically, so after «Показать
  173 |  * программу» the wizard waits in the EPIC-25 draft program window
  174 |  * (`draft-version-badge`) until the program is compiled — the consent CTA is the NEW
  175 |  * step, not the confirm output.
  176 |  */
  177 | export async function confirmProfileStep(page: Page, skill: string): Promise<void> {
  178 |   const cta = page.getByRole('button', { name: 'Показать программу' });
  179 |   await expect(page.locator('#wizard-skill')).toHaveValue(skill, { timeout: 60_000 });
  180 |   const language = page.locator('#wizard-language');
  181 |   if ((await language.inputValue()) === '' && !(await cta.isEnabled())) {
  182 |     await language.selectOption({ label: 'Русский' });
  183 |   }
  184 |   await expect(cta).toBeEnabled({ timeout: 60_000 });
  185 |   await cta.click();
  186 |   // The flip-enqueued draft ran: its clarification cards render while the server builds
  187 |   // the current-draft map without a generation-provider request.
  188 |   await expect(page.getByTestId('draft-version-badge')).toBeVisible({ timeout: 120_000 });
  189 |   await waitForCalls(page, 'propose_plan_draft', 1);
  190 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  191 |     timeout: 120_000,
  192 |   });
  193 | }
  194 | 
  195 | /**
  196 |  * Answer ONE draft clarification card through the real card contract: select the first
  197 |  * option of the first open card and Apply it exactly once. The 202 flow re-runs the plan
  198 |  * draft (`propose_plan_draft` >= 2) and the gate rebuilds the map for the moved spec
  199 |  * without a provider call.
  200 |  */
  201 | export async function answerFirstDraftQuestion(page: Page): Promise<void> {
  202 |   // Wait for the settled plan window first: between the draft commit and the
  203 |   // compiled program the controller auto-compiles the gated map, and that
  204 |   // draft→compiled branch change REMOUNTS the question cards and clears any
  205 |   // local selection (LF-146 slice B). Interacting before the settle races the
```