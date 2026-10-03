# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: evaluation-modes-table-code.spec.ts >> table unit: a failing check skips the Mentor; a passing check starts the review automatically (E5, E7, E8)
- Location: e2e/evaluation-modes-table-code.spec.ts:82:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByText('PASSED', { exact: true })
Expected: visible
Timeout: 60000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 60000ms
  - waiting for getByText('PASSED', { exact: true })

```

```yaml
- banner:
  - button "Меню ученика"
- region "Course workspace bar":
  - button "Back to Course Library": Library
  - 'heading "Learning plan: Household budget table with a Mentor review" [level=1]'
  - text: sha256:061efec666d47d0fd4c1c88738977a8d282a15e0e736f286f6574cf2a4e744f7
  - 'status "Server status: Local server active"': Local server active
- complementary "Course Curriculum":
  - text: Overview 0 of 2 units complete 0%
  - 'progressbar "Course progress: 0 of 2 units (0%)"'
  - text: generated-b846b16b18774caa893be460c5e4cf11
  - navigation "Curriculum Navigation":
    - heading "Curriculum" [level=2]
    - text: Foundations of Household budget table with a Mentor review
    - list:
      - listitem:
        - button "Foundations of Household budget table with a Mentor review (in_progress, check and AI Mentor review)": Foundations of Household budget table with a Mentor review
      - listitem:
        - button "Foundations of Household budget table with a Mentor review (locked, check and AI Mentor review)" [disabled]: Foundations of Household budget table with a Mentor review
- main "Lesson Workspace":
  - tablist "Unit workspace panels":
    - tab "Theory"
    - tab "Code" [selected]
  - tabpanel "Code":
    - navigation "Learner files":
      - heading "Learner files" [level=3]
      - list:
        - listitem:
          - button "table.csv": units / lf158-group-01-practice / workspace / learner / table.csv
    - 'status "Save status: No unsaved changes"': No unsaved changes
    - textbox "Editor for units/lf158-group-01-practice/workspace/learner/table.csv"
- complementary "Support":
  - region "Support":
    - tablist "Support tools":
      - tab "Rubric" [selected]
      - tab "AI Mentor"
    - tabpanel "Rubric":
      - region "Rubric":
        - heading "Rubric" [level=2]
        - text: 1 / 4 Met
        - paragraph: Checked first, then reviewed by the AI Mentor.
        - status: Check passed. AI Mentor is reviewing your answer...
        - group "Unit Criteria":
          - heading "Check" [level=3]
          - list "Check criteria":
            - listitem:
              - text: Explain and apply the core ideas of Household budget table with a Mentor review. Passed
              - paragraph: "Evidence: command check-lf158-group-01-practice passed"
          - heading "Mentor review" [level=3]
          - list "Mentor review criteria":
            - listitem:
              - text: "The answer correctly explains the core idea of: Explain and apply the core ideas of Household budget table with a Mentor review. Reviewing"
              - paragraph: mentor review is pending for this workspace
            - listitem:
              - text: "The answer justifies its claims with concrete reasoning about: Explain and apply the core ideas of Household budget table with a Mentor review. Reviewing"
              - paragraph: mentor review is pending for this workspace
            - listitem:
              - text: "The answer applies the idea to a specific case of: Explain and apply the core ideas of Household budget table with a Mentor review. Reviewing"
              - paragraph: mentor review is pending for this workspace
        - button "Reviewing..." [disabled]
```

# Test source

```ts
  25  |   mentorReviewCalls,
  26  |   openAnswerTab,
  27  |   releaseMentor,
  28  |   resetMentorFake,
  29  |   runRubricAction,
  30  |   scriptMentor,
  31  |   startModeCourse,
  32  |   unitCriteria,
  33  |   writeAnswer,
  34  | } from './evaluation-mode-support';
  35  | 
  36  | const PYTHON_WRONG = 'def double(n):\n    return n + 2\n';
  37  | const PYTHON_RIGHT = 'def double(n):\n    return n * 2\n';
  38  | const TABLE_WRONG = 'category,amount\nrent,100\nfood,80\n';
  39  | const TABLE_RIGHT = 'category,amount\nrent,120\nfood,80\n';
  40  | 
  41  | test.use(MODES_SERVER);
  42  | test.describe.configure({ mode: 'serial' });
  43  | 
  44  | test.beforeEach(async ({ page }) => {
  45  |   await drainAuthoringQueue(page);
  46  |   await resetMentorFake();
  47  | });
  48  | test.afterEach(async () => {
  49  |   await resetMentorFake();
  50  |   // The armed per-module format is sticky: never leak it into another spec file.
  51  |   await resetGenerationFake();
  52  | });
  53  | 
  54  | test('code unit: command mode keeps the check-only journey and never involves the Mentor (E10)', async ({
  55  |   page,
  56  | }) => {
  57  |   const { courseRef } = await startModeCourse(page, 'e27-code', 'code');
  58  |   const unit = await firstUnit(page, courseRef);
  59  |   expect(unit.evaluation_mode).toBe('command');
  60  |   await openCodeTab(page);
  61  | 
  62  |   // Command units keep the legacy wording: Check work, UNMET CRITERIA, no Mentor groups or line.
  63  |   await expect(page.getByRole('button', { name: 'Check work', exact: true })).toBeVisible();
  64  |   await expect(page.getByRole('button', { name: 'Submit for review' })).toHaveCount(0);
  65  |   await expect(page.getByText(/Reviewed by the AI Mentor/)).toHaveCount(0);
  66  |   await expect(page.getByText('Checked automatically.')).toHaveCount(0);
  67  |   await fillEditor(page, PYTHON_WRONG);
  68  |   await waitCodeSaved(page);
  69  |   expect(await checkWork(page)).toBe('UNMET CRITERIA');
  70  |   await expect(page.getByText('NOT MET YET', { exact: true })).toHaveCount(0);
  71  | 
  72  |   await fillEditor(page, PYTHON_RIGHT);
  73  |   await waitCodeSaved(page);
  74  |   expect(await checkWork(page)).toBe('PASSED');
  75  |   await waitForCourseProgress(page, courseRef, { total_units: 2, completed_units: 1, percent: 50 });
  76  |   expect(await mentorReviewCalls()).toBe(0);
  77  |   expect(
  78  |     (await unitCriteria(page, courseRef, unit.unit_ref)).every((item) => item.source === 'check'),
  79  |   ).toBe(true);
  80  | });
  81  | 
  82  | test('table unit: a failing check skips the Mentor; a passing check starts the review automatically (E5, E7, E8)', async ({
  83  |   page,
  84  | }) => {
  85  |   const { courseRef } = await startModeCourse(page, 'e27-table', 'table');
  86  |   const unit = await firstUnit(page, courseRef);
  87  |   expect(unit.evaluation_mode).toBe('hybrid');
  88  |   await openAnswerTab(page, 'Code');
  89  |   await expect(page.getByText('Checked first, then reviewed by the AI Mentor.')).toBeVisible();
  90  |   await expect(page.getByRole('heading', { name: 'Check', exact: true })).toBeVisible();
  91  |   await expect(page.getByRole('heading', { name: 'Mentor review', exact: true })).toBeVisible();
  92  | 
  93  |   // Check fails (twice): the Mentor criteria are not evaluated, no review runs, and a deterministic
  94  |   // check criterion is never offered for accept-with-note.
  95  |   await writeAnswer(page, TABLE_WRONG);
  96  |   for (let attempt = 0; attempt < 2; attempt += 1) {
  97  |     await runRubricAction(page, 'Check work');
  98  |     await expect(page.getByText('The check found a problem. The Mentor review did not run.')).toBeVisible({
  99  |       timeout: 60_000,
  100 |     });
  101 |     const mentorRows = page.getByRole('list', { name: 'Mentor review criteria' }).getByRole('listitem');
  102 |     await expect(mentorRows).toHaveCount(3);
  103 |     for (const row of await mentorRows.all()) {
  104 |       await expect(row.getByText('Not evaluated', { exact: true })).toBeVisible();
  105 |       await expect(row).toContainText('mentor review did not run because a required check criterion is unmet');
  106 |     }
  107 |     await expect(
  108 |       page.getByRole('list', { name: 'Check criteria' }).getByText('Not met', { exact: true }),
  109 |     ).toBeVisible();
  110 |     await expect(page.getByRole('button', { name: 'Accept with note...' })).toHaveCount(0);
  111 |   }
  112 |   expect(await mentorReviewCalls()).toBe(0);
  113 |   const afterFailures = await unitCriteria(page, courseRef, unit.unit_ref);
  114 |   expect(afterFailures.filter((item) => item.source === 'mentor')).toHaveLength(3);
  115 |   expect(afterFailures.every((item) => (item.rejection_count ?? 0) === 0)).toBe(true);
  116 |   expect(afterFailures.every((item) => item.accept_with_note_offered !== true)).toBe(true);
  117 | 
  118 |   // Check passes: the review starts by itself (a held fake keeps the Reviewing state observable).
  119 |   await scriptMentor({ hold: true });
  120 |   await writeAnswer(page, TABLE_RIGHT);
  121 |   await runRubricAction(page, 'Check work');
  122 |   await expect(page.getByRole('button', { name: 'Reviewing...' })).toBeDisabled({ timeout: 30_000 });
  123 |   await expect(page.getByText('Reviewing', { exact: true }).first()).toBeVisible();
  124 |   await releaseMentor();
> 125 |   await expect(page.getByText('PASSED', { exact: true })).toBeVisible({ timeout: 60_000 });
      |                                                           ^ Error: expect(locator).toBeVisible() failed
  126 |   expect(await mentorReviewCalls()).toBe(1);
  127 |   await expect(criterionRow(page, 'correctly explains the core idea').getByText('Passed', { exact: true })).toBeVisible();
  128 |   const completed = await unitCriteria(page, courseRef, unit.unit_ref);
  129 |   expect(completed.every((item) => item.status !== 'accepted_with_note')).toBe(true);
  130 |   await waitForCourseProgress(page, courseRef, { total_units: 2, completed_units: 1, percent: 50 });
  131 | });
  132 | 
```