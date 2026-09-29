# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: response-form.spec.ts >> capacity: keyboard option choice, reflection alone never enables Check
- Location: e2e/response-form.spec.ts:177:1

# Error details

```
Error: option ab is not part of plan: 
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
          - heading "Response Form Capacity" [level=1] [ref=e25]
          - generic [ref=e26]: sha256:6e47774fc09ae715ba6b40df1e738b8eabb65544c93d4641b6dea7a9bb61dd26
      - 'status "Server status: Local server active" [ref=e28]':
        - generic [ref=e30]: Local server active
    - generic [ref=e31]:
      - complementary "Course Curriculum" [ref=e32]:
        - generic [ref=e33]:
          - generic [ref=e34]: Overview
          - generic [ref=e40]:
            - generic [ref=e41]:
              - generic [ref=e42]: 0 of 2 units complete
              - generic [ref=e43]: 0%
            - 'progressbar "Course progress: 0 of 2 units (0%)" [ref=e44]'
          - generic [ref=e45]: response-form-capacity
        - navigation "Curriculum Navigation" [ref=e47]:
          - heading "Curriculum" [level=2] [ref=e48]
          - generic [ref=e50]:
            - generic [ref=e51]: One
            - list [ref=e52]:
              - listitem [ref=e53]:
                - 'button "Practice: capacity plan (in_progress)" [ref=e54] [cursor=pointer]':
                  - generic [ref=e58]: "Practice: capacity plan"
              - listitem [ref=e59]:
                - 'button "Transfer: new capacity (locked)" [disabled] [ref=e60]':
                  - generic [ref=e64]: "Transfer: new capacity"
      - main "Lesson Workspace" [ref=e65]:
        - generic [ref=e66]:
          - tablist "Unit workspace panels" [ref=e67]:
            - tab "Theory" [ref=e68] [cursor=pointer]
            - tab "Code" [selected] [ref=e71] [cursor=pointer]
          - tabpanel "Code" [ref=e76]:
            - region "Ваше решение" [ref=e77]:
              - generic [ref=e78]:
                - generic [ref=e79]:
                  - heading "Ваше решение" [level=2] [ref=e80]
                  - paragraph [ref=e81]: Your solution
                - generic [ref=e82]: Сохранено
              - generic [ref=e84]:
                - group "Work plan" [ref=e85]:
                  - generic [ref=e87]:
                    - radio "A + B" [ref=e88]
                    - text: A + B
                  - generic [ref=e89]:
                    - radio "A + B + C" [checked] [ref=e90]
                    - text: A + B + C
                  - generic [ref=e91]:
                    - radio "A + C" [ref=e92]
                    - text: A + C
                  - generic [ref=e93]:
                    - radio "B + C" [ref=e94]
                    - text: B + C
                - generic [ref=e95]:
                  - generic [ref=e96]: How you reasonedДля вашего разбора
                  - textbox "How you reasonedДля вашего разбора" [ref=e97]: I compared the mandatory work with the capacity.
                  - generic [ref=e98]:
                    - paragraph [ref=e99]: Качество этого пояснения не проверяется автоматически — оно нужно для вашего разбора.
                    - generic [ref=e100]: 48 / 500
                - generic [ref=e101]:
                  - heading "Как проверим" [level=3] [ref=e102]
                  - list [ref=e103]:
                    - listitem [ref=e104]: Choose the plan covering the mandatory work within capacity.
              - button "Открыть файл" [ref=e109] [cursor=pointer]
      - complementary "Support" [ref=e111]:
        - region "Support" [ref=e112]:
          - tablist "Support tools" [ref=e113]:
            - tab "Rubric" [selected] [ref=e114]
            - tab "AI Mentor" [ref=e118]
          - tabpanel "Rubric" [ref=e121]:
            - region "Rubric" [ref=e122]:
              - generic [ref=e123]:
                - heading "Rubric" [level=2] [ref=e127]
                - generic "0 of 1 criteria met" [ref=e128]: 0 / 1 Met
              - status [ref=e129]:
                - generic [ref=e130]: UNMET CRITERIA
                - paragraph [ref=e133]: unit cap-practice check unmet
                - paragraph [ref=e134]: "Smallest next action: Choose the plan covering the mandatory work within capacity."
              - list "Unit Criteria" [ref=e135]:
                - listitem [ref=e136]:
                  - generic [ref=e140]:
                    - generic [ref=e141]: Choose the plan covering the mandatory work within capacity.
                    - generic [ref=e142]: Failed
                  - alert [ref=e144]:
                    - generic [ref=e147]: command check-cap-practice exited 1
                  - group [ref=e148]:
                    - generic "Show Hint" [ref=e149] [cursor=pointer]
              - button "Check work" [ref=e152] [cursor=pointer]
```

# Test source

```ts
  1   | /**
  2   |  * Response-form workspace journeys on FIXED-template courses (finance budget and capacity
  3   |  * plan; produced by `response_form_fixture.py` from the framework's own evidence templates).
  4   |  * No generation provider is involved: the specs drive the real form, autosave, conflict
  5   |  * handling, keyboard choice, check evaluation and accessibility of the shipped form contract.
  6   |  */
  7   | import { AxeBuilder } from '@axe-core/playwright';
  8   | import { expect, test, type Page } from '@playwright/test';
  9   | import {
  10  |   apiGet,
  11  |   apiSend,
  12  |   checkWork,
  13  |   courseView,
  14  |   findCourseRef,
  15  |   goNextUnit,
  16  |   openCodeTab,
  17  |   waitForCourseProgress,
  18  | } from './course-generation-support';
  19  | import { enterCourseWorkspace, expectNoHorizontalOverflow, gotoLearnerApp } from './support';
  20  | 
  21  | const FINANCE_TITLE = 'Response Form Finance';
  22  | const CAPACITY_TITLE = 'Response Form Capacity';
  23  | 
  24  | /** Fill one number field of the response form. */
  25  | async function fillNumberField(page: Page, fieldId: string, value: string): Promise<void> {
  26  |   await page.locator(`#${fieldId}`).fill(value);
  27  | }
  28  | 
  29  | /**
  30  |  * Choose one radio option with the keyboard only: focus the group's first radio, then either
  31  |  * press Space (first option) or ArrowDown once per step (the native radio group moves and
  32  |  * selects). Never `locator.check()`.
  33  |  */
  34  | async function chooseOptionByKeyboard(page: Page, fieldId: string, optionId: string): Promise<void> {
  35  |   const radios = page.locator(`input[type="radio"][name="${fieldId}"]`);
  36  |   const values = await radios.evaluateAll((elements) =>
  37  |     elements.map((element) => (element as HTMLInputElement).value),
  38  |   );
  39  |   const target = values.indexOf(optionId);
> 40  |   if (target === -1) throw new Error(`option ${optionId} is not part of ${fieldId}: ${values}`);
      |                            ^ Error: option ab is not part of plan: 
  41  |   await radios.first().focus();
  42  |   await expect(radios.first()).toBeFocused();
  43  |   if (target === 0) {
  44  |     await page.keyboard.press('Space');
  45  |   } else {
  46  |     for (let step = 0; step < target; step += 1) await page.keyboard.press('ArrowDown');
  47  |   }
  48  |   const chosen = page.locator(`input[type="radio"][name="${fieldId}"][value="${optionId}"]`);
  49  |   await expect(chosen).toBeChecked();
  50  |   await expect(chosen).toBeFocused();
  51  | }
  52  | 
  53  | /**
  54  |  * Wait until the form reports a saved state (autosave settled). The «Сохранено» label is
  55  |  * sticky from the PREVIOUS save, so first require the fresh edit to surface the dirty state
  56  |  * (it replaces the label for the whole debounce window), then wait for the save cycle to
  57  |  * settle back on «Сохранено». A new value can never skip the dirty state.
  58  |  */
  59  | async function waitForFormSaved(page: Page): Promise<void> {
  60  |   await expect(page.getByText('Есть изменения')).toBeVisible({ timeout: 30_000 });
  61  |   await expect(page.getByText('Сохранено')).toBeVisible({ timeout: 30_000 });
  62  | }
  63  | 
  64  | /**
  65  |  * Simulate one external editor write on the unit's answers file: read the current revision
  66  |  * through the API and save a DIFFERENT still-valid payload (same field ids, changed number),
  67  |  * bumping the content-hash revision so the form's next autosave hits the typed conflict.
  68  |  */
  69  | async function externalEditAnswers(page: Page, courseRef: string, unitRef: string): Promise<void> {
  70  |   const inventory = await apiGet(page, `/api/v1/courses/${courseRef}/units/${unitRef}/files`);
  71  |   const files = (inventory['files'] ?? []) as Array<{ file_ref?: string; is_editable?: boolean }>;
  72  |   const editable = files.find((file) => file.is_editable === true);
  73  |   if (editable === undefined || typeof editable.file_ref !== 'string') {
  74  |     throw new Error('no editable answers file found for the external edit');
  75  |   }
  76  |   const content = await apiGet(page, `/api/v1/files/${editable.file_ref}`);
  77  |   const payload = JSON.parse(content['content'] as string) as {
  78  |     schema_version: number;
  79  |     answers: Record<string, unknown>;
  80  |   };
  81  |   payload.answers = { ...payload.answers, months: '2' };
  82  |   const saved = await apiSend(
  83  |     page,
  84  |     'PUT',
  85  |     `/api/v1/files/${editable.file_ref}`,
  86  |     { schema_version: 1, expected_revision: content['revision'], content: JSON.stringify(payload) },
  87  |     `external-edit-${Date.now()}`,
  88  |   );
  89  |   if (saved.status !== 200) {
  90  |     throw new Error(`external edit failed with ${saved.status}: ${JSON.stringify(saved.body)}`);
  91  |   }
  92  | }
  93  | 
  94  | /** Resolve one unit's opaque ref in the workspace view by its authored unit id. */
  95  | async function unitRefById(page: Page, courseRef: string, unitId: string): Promise<string> {
  96  |   const view = await courseView(page, courseRef);
  97  |   const units = ((view['stages'] as Array<{ units: Array<Record<string, unknown>> }>) ?? []).flatMap(
  98  |     (stage) => stage.units,
  99  |   );
  100 |   const matches = units.filter((unit) => unit['unit_id'] === unitId);
  101 |   const unitRef = matches.length === 1 ? matches[0]?.['unit_ref'] : undefined;
  102 |   if (typeof unitRef !== 'string') {
  103 |     throw new Error(`expected exactly one ${unitId} unit in the workspace view`);
  104 |   }
  105 |   return unitRef;
  106 | }
  107 | 
  108 | async function openFormWorkspace(page: Page, lessonTitle: string): Promise<void> {
  109 |   // The workspace unit heading (h2); the lesson body repeats the title as its own h1.
  110 |   await expect(page.getByRole('heading', { level: 2, name: lessonTitle, exact: true })).toBeVisible();
  111 |   await openCodeTab(page);
  112 |   await expect(page.getByRole('heading', { name: 'Ваше решение' })).toBeVisible({ timeout: 30_000 });
  113 | }
  114 | 
  115 | test.describe.configure({ mode: 'serial' });
  116 | 
  117 | test('finance: wrong values are unmet, an external edit conflicts and keeps input', async ({
  118 |   page,
  119 | }) => {
  120 |   await gotoLearnerApp(page);
  121 |   await enterCourseWorkspace(page, FINANCE_TITLE);
  122 |   await openFormWorkspace(page, 'Practice: budget balance');
  123 | 
  124 |   // A14: bad values are saved, then the check reports unmet criteria.
  125 |   await fillNumberField(page, 'balance', '15000');
  126 |   await fillNumberField(page, 'months', '2');
  127 |   await waitForFormSaved(page);
  128 |   expect(await checkWork(page)).toBe('UNMET CRITERIA');
  129 | 
  130 |   // Good values pass.
  131 |   await fillNumberField(page, 'balance', '14000');
  132 |   await fillNumberField(page, 'months', '3');
  133 |   await waitForFormSaved(page);
  134 |   expect(await checkWork(page)).toBe('PASSED');
  135 | 
  136 |   // A16 on the transfer unit: an external edit conflicts with the form autosave, the
  137 |   // input is kept, Check waits for the save, and the resolved save passes.
  138 |   await goNextUnit(page);
  139 |   await openFormWorkspace(page, 'Transfer: new budget');
  140 |   const courseRef = await findCourseRef(page, FINANCE_TITLE);
```