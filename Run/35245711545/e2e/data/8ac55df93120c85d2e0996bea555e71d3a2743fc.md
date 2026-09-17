# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: learner-journey.spec.ts >> select, learn theory, practice, check and advance a course
- Location: e2e/learner-journey.spec.ts:13:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByText('Journey Course')
Expected: visible
Timeout: 15000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 15000ms
  - waiting for getByText('Journey Course')

```

```yaml
- main:
  - alert: A newer build of the learner workspace is available. Please reload the page.
```

# Test source

```ts
  1   | import { expect, test } from '@playwright/test';
  2   | import { enterCourseWorkspace, enterJourneyCourseWorkspace, fillEditor, gotoLearnerApp } from './support';
  3   | 
  4   | /**
  5   |  * Full learner journey over the live LocalLearnerServer, recorded to `test-results/videos/`:
  6   |  * library -> trust confirmation -> theory quiz (wrong answer -> hint -> correct answer) ->
  7   |  * practice workspace -> editor autosave -> Check work -> PASSED -> Next unit unlocked.
  8   |  *
  9   |  * The journey runs against the dedicated "Journey Course" instance so its theory gate is always
  10  |  * fresh: the accessibility and responsive suites complete the shared tech course's gate through
  11  |  * the evaluation API, and spec files run in alphabetical order.
  12  |  */
  13  | test('select, learn theory, practice, check and advance a course', async ({ page }) => {
  14  |   // Step 1: the course library renders catalog cards; start the journey course and reach the
  15  |   // workspace, accepting the host-trust confirmation when required.
  16  |   await gotoLearnerApp(page);
> 17  |   await expect(page.getByText('Journey Course')).toBeVisible({ timeout: 15_000 });
      |                                                  ^ Error: expect(locator).toBeVisible() failed
  18  |   await enterJourneyCourseWorkspace(page);
  19  | 
  20  |   // Step 2: the theory unit renders its reading content and the interactive quiz on its own tab.
  21  |   await expect(page.getByRole('heading', { name: 'Theory one' }).first()).toBeVisible();
  22  |   await page.getByRole('tab', { name: 'Quiz' }).click();
  23  |   const quiz = page.getByRole('group', { name: 'What is the domain boundary?' });
  24  |   await expect(quiz).toBeVisible({ timeout: 15_000 });
  25  | 
  26  |   // First attempt: the wrong option yields criteria-based error feedback and a hint toggle.
  27  |   await quiz.getByRole('radio', { name: 'Web details cross into domain' }).check();
  28  |   await page.getByRole('button', { name: 'Submit Quiz' }).click();
  29  |   const feedback = quiz.getByRole('alert');
  30  |   await expect(feedback).toContainText('Not quite. Web details crossing into the domain breaks the boundary.');
  31  |   const showHint = quiz.getByRole('button', { name: 'Show Hint' });
  32  |   await expect(showHint).toBeVisible();
  33  |   await showHint.click();
  34  |   await expect(
  35  |     quiz.getByText('Transport details must not leak into business domain'),
  36  |   ).toBeVisible();
  37  | 
  38  |   // Second attempt: the correct option passes the gate and completes the theory unit.
  39  |   await quiz.getByRole('radio', { name: 'Transport details stay at boundary' }).check();
  40  |   await page.getByRole('button', { name: 'Submit Quiz' }).click();
  41  |   await expect(quiz.getByLabel('Passed')).toBeVisible({ timeout: 15_000 });
  42  | 
  43  |   // Advancing past theory loads the practice unit's lesson.
  44  |   const next = page.getByRole('button', { name: 'Next unit' });
  45  |   await expect(next).toBeEnabled({ timeout: 15_000 });
  46  |   await next.click();
  47  |   await expect(page.getByRole('heading', { name: 'Practice one' }).first()).toBeVisible({
  48  |     timeout: 15_000,
  49  |   });
  50  | 
  51  |   // Step 3: the 3-pane practice workspace renders the scaffolded file in the editor.
  52  |   await page.getByRole('tab', { name: 'Code' }).click();
  53  |   const editor = page.locator('.cm-content');
  54  |   await expect(editor).toBeVisible();
  55  | 
  56  |   // Editing marks the buffer dirty and then saves: dirty -> saving -> saved.
  57  |   await fillEditor(page, "def value():\n    return 'hello'");
  58  |   await expect(page.getByText('Unsaved changes')).toBeVisible();
  59  |   await expect(page.getByText('Saved')).toBeVisible({ timeout: 10_000 });
  60  | 
  61  |   // Check work: the in-flight state disables the editor and the check button.
  62  |   await page.getByRole('button', { name: 'Check work' }).click();
  63  |   await expect(page.getByRole('button', { name: 'Checking...' })).toBeVisible();
  64  |   await expect(page.locator('[role="textbox"][aria-readonly="true"]').first()).toBeVisible();
  65  | 
  66  |   // The check reaches the terminal PASSED result with the criterion met.
  67  |   await expect(page.getByText('PASSED', { exact: true })).toBeVisible({ timeout: 20_000 });
  68  |   // The rubric header counts every met criterion against the published unit criteria.
  69  |   await expect(page.getByLabel(/\d of \d criteria met/)).toBeVisible();
  70  |   await expect(page.locator('[role="textbox"][aria-readonly="false"]').first()).toBeVisible();
  71  | 
  72  |   // Next unit unlocks once the practice criterion is met.
  73  |   await expect(page.getByRole('button', { name: 'Next unit' })).toBeEnabled({ timeout: 10_000 });
  74  | });
  75  | 
  76  | /**
  77  |  * Mixed-capability unit journey (TEST-005): one lesson advertises Theory, Quiz and Code tabs
  78  |  * simultaneously. The learner reads, submits the self-check quiz, edits the code, runs
  79  |  * Check work and advances to the stage transfer — all without leaving the unit.
  80  |  */
  81  | test('mixed unit serves theory, quiz and code tabs in one workspace', async ({ page }) => {
  82  |   await gotoLearnerApp(page);
  83  |   await enterCourseWorkspace(page, 'Mixed Course');
  84  | 
  85  |   // The mixed unit renders all three adaptive tabs with Theory active by default.
  86  |   await expect(page.getByRole('heading', { name: 'Mixed theory and practice' }).first()).toBeVisible();
  87  |   const tablist = page.getByRole('tablist', { name: 'Unit workspace panels' });
  88  |   await expect(tablist.getByRole('tab', { name: 'Theory' })).toBeVisible();
  89  |   await expect(tablist.getByRole('tab', { name: 'Quiz' })).toBeVisible();
  90  |   await expect(tablist.getByRole('tab', { name: 'Code' })).toBeVisible();
  91  | 
  92  |   // Quiz tab: the self-check passes on the correct option.
  93  |   await tablist.getByRole('tab', { name: 'Quiz' }).click();
  94  |   const quiz = page.getByRole('group', { name: 'What does a mixed unit combine?' });
  95  |   await expect(quiz).toBeVisible({ timeout: 15_000 });
  96  |   await quiz.getByRole('radio', { name: 'Reading, a quiz and code practice' }).check();
  97  |   await page.getByRole('button', { name: 'Submit Quiz' }).click();
  98  |   await expect(quiz.getByLabel('Passed')).toBeVisible({ timeout: 15_000 });
  99  | 
  100 |   // Code tab: the editor accepts the solution and Check work passes in the same unit.
  101 |   await tablist.getByRole('tab', { name: 'Code' }).click();
  102 |   const editor = page.locator('.cm-content');
  103 |   await expect(editor).toBeVisible();
  104 |   await fillEditor(page, "def value():\n    return 'mixed'");
  105 |   await expect(page.getByText('Saved')).toBeVisible({ timeout: 10_000 });
  106 |   await page.getByRole('button', { name: 'Check work' }).click();
  107 |   await expect(page.getByText('PASSED', { exact: true })).toBeVisible({ timeout: 20_000 });
  108 |   await expect(page.getByLabel(/\d of \d criteria met/)).toBeVisible();
  109 | 
  110 |   // Advancing leaves the mixed unit and opens the stage transfer.
  111 |   const next = page.getByRole('button', { name: 'Next unit' });
  112 |   await expect(next).toBeEnabled({ timeout: 10_000 });
  113 |   await next.click();
  114 |   await expect(page.getByRole('heading', { name: 'Transfer one' }).first()).toBeVisible({
  115 |     timeout: 15_000,
  116 |   });
  117 | });
```