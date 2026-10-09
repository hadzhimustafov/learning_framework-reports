# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: evaluation-modes-text.spec.ts >> text unit: rejection x2 offers accept with note; the mark clears on a later Mentor pass (E7, E8)
- Location: e2e/evaluation-modes-text.spec.ts:77:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByRole('listitem').filter({ hasText: 'correctly explains the core idea' }).getByText('Review 2 of 2')
Expected: visible
Timeout: 60000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 60000ms
  - waiting for getByRole('listitem').filter({ hasText: 'correctly explains the core idea' }).getByText('Review 2 of 2')

```

```yaml
- banner:
  - button "Меню ученика"
- region "Course workspace bar":
  - button "Back to Course Library": Library
  - 'heading "Learning plan: Team capacity memo accepted with a note" [level=1]'
  - text: sha256:82b4c58191c5554a62cf0ecb9111386dd2444c792669c521f2da4edbfe322219
  - 'status "Server status: Local server active"': Local server active
- complementary "Course Curriculum":
  - text: Overview 0 of 2 units complete 0%
  - 'progressbar "Course progress: 0 of 2 units (0%)"'
  - text: generated-291516ab438b4f70b4d7b66d44b1fe90
  - navigation "Curriculum Navigation":
    - heading "Curriculum" [level=2]
    - text: Foundations of Team capacity memo accepted with a note
    - list:
      - listitem:
        - button "Foundations of Team capacity memo accepted with a note (in_progress, reviewed by AI Mentor)": Foundations of Team capacity memo accepted with a note
      - listitem:
        - button "Foundations of Team capacity memo accepted with a note (locked, reviewed by AI Mentor)" [disabled]: Foundations of Team capacity memo accepted with a note
- main "Lesson Workspace":
  - tablist "Unit workspace panels":
    - tab "Theory"
    - tab "Answer" [selected]
  - tabpanel "Answer":
    - navigation "Learner files":
      - heading "Learner files" [level=3]
      - list:
        - listitem:
          - button "answer.md": units / lf158-group-01-practice / workspace / learner / answer.md
    - button "Preview"
    - 'status "Save status: No unsaved changes"': No unsaved changes
    - textbox "Editor for units/lf158-group-01-practice/workspace/learner/answer.md"
- complementary "Support":
  - region "Support":
    - tablist "Support tools":
      - tab "Rubric" [selected]
      - tab "AI Mentor"
    - tabpanel "Rubric":
      - region "Rubric":
        - heading "Rubric" [level=2]
        - paragraph: "Reviewed by the AI Mentor against 3 criteria. Free form: no required headings or length."
        - alert: Save your changes before checking your work.
        - list "Unit Criteria":
          - listitem: "The answer correctly explains the core idea of: Explain and apply the core ideas of Team capacity memo accepted with a note. Required Review 1 of 2"
          - listitem: "The answer justifies its claims with concrete reasoning about: Explain and apply the core ideas of Team capacity memo accepted with a note. Required"
          - listitem: "The answer applies the idea to a specific case of: Explain and apply the core ideas of Team capacity memo accepted with a note. Optional"
        - button "Submit for review"
```

# Test source

```ts
  1   | /**
  2   |  * EPIC-27 (M27-007) text unit journeys: a generated `text` practice unit is judged by the AI Mentor
  3   |  * against its published rubric (evaluation mode `mentor`). Courses come from the real generation
  4   |  * flow; the Mentor provider is the scripted loopback fake (`/__mentor/*`), never a real provider.
  5   |  *
  6   |  * Acceptance mapping (spec-design-api.md section 1): E4 precondition without a provider call, E7
  7   |  * rejection x2 -> accept with note -> mark -> a later Mentor pass clears it, E8 validated verdicts
  8   |  * only, E6 Mentor outage without a consumed attempt.
  9   |  */
  10  | import { expect, test, type Page } from '@playwright/test';
  11  | import { drainAuthoringQueue, resetGenerationFake } from './course-generation-support';
  12  | import {
  13  |   MODES_SERVER,
  14  |   criterionRow,
  15  |   firstUnit,
  16  |   mentorReviewCalls,
  17  |   modeCourseTitle,
  18  |   openAnswerTab,
  19  |   resetMentorFake,
  20  |   runRubricAction,
  21  |   scriptMentor,
  22  |   startModeCourse,
  23  |   unitCriteria,
  24  |   writeAnswer,
  25  | } from './evaluation-mode-support';
  26  | import { gotoLearnerApp } from './support';
  27  | 
  28  | const SUBSTANCE = 'correctly explains the core idea';
  29  | const REASONING = 'justifies its claims';
  30  | const ANSWER = '# Capacity memo\n\nWe choose plan A and B because they cover the mandatory work.\n';
  31  | 
  32  | test.use(MODES_SERVER);
  33  | test.describe.configure({ mode: 'serial' });
  34  | 
  35  | test.beforeEach(async ({ page }) => {
  36  |   await drainAuthoringQueue(page);
  37  |   await resetMentorFake();
  38  | });
  39  | test.afterEach(async () => {
  40  |   await resetMentorFake();
  41  |   // The armed per-module format is sticky: never leak it into another spec file.
  42  |   await resetGenerationFake();
  43  | });
  44  | 
  45  | async function expectNoRejections(page: Page, courseRef: string, unitRef: string): Promise<void> {
  46  |   const criteria = await unitCriteria(page, courseRef, unitRef);
  47  |   expect(criteria.length).toBe(3);
  48  |   expect(criteria.every((item) => (item.rejection_count ?? 0) === 0)).toBe(true);
  49  | }
  50  | 
  51  | test('text unit: empty answer and unchanged starter send nothing and consume no attempt (E4, E8)', async ({
  52  |   page,
  53  | }) => {
  54  |   const { courseRef } = await startModeCourse(page, 'e27-text-precondition', 'text');
  55  |   const unit = await firstUnit(page, courseRef);
  56  |   expect(unit.evaluation_mode).toBe('mentor');
  57  |   await openAnswerTab(page);
  58  |   await expect(
  59  |     page.getByText(/Reviewed by the AI Mentor against 3 criteria\. Free form/),
  60  |   ).toBeVisible();
  61  | 
  62  |   // Unchanged neutral starter: inline guidance, no provider call.
  63  |   await runRubricAction(page, 'Submit for review');
  64  |   await expect(page.getByText(/Your answer still matches the starter\. Change it, then submit\./)).toBeVisible();
  65  |   expect(await mentorReviewCalls()).toBe(0);
  66  | 
  67  |   // Empty answer: inline guidance naming the file, still no provider call.
  68  |   await writeAnswer(page, '');
  69  |   await runRubricAction(page, 'Submit for review');
  70  |   await expect(page.getByText(/Your answer is empty\. Write it in .*answer\.md, then submit\./)).toBeVisible();
  71  |   expect(await mentorReviewCalls()).toBe(0);
  72  |   await expectNoRejections(page, courseRef, unit.unit_ref);
  73  |   await expect(page.getByText('UNMET CRITERIA', { exact: true })).toHaveCount(0);
  74  |   await expect(page.getByText('NOT MET YET', { exact: true })).toHaveCount(0);
  75  | });
  76  | 
  77  | test('text unit: rejection x2 offers accept with note; the mark clears on a later Mentor pass (E7, E8)', async ({
  78  |   page,
  79  | }) => {
  80  |   const { title, courseRef } = await startModeCourse(page, 'e27-text-accept', 'text');
  81  |   expect(title).toBe(modeCourseTitle('e27-text-accept'));
  82  |   const unit = await firstUnit(page, courseRef);
  83  |   await openAnswerTab(page);
  84  |   await writeAnswer(page, ANSWER);
  85  | 
  86  |   // Two Mentor rejections of the SAME criterion; the other criteria pass each time.
  87  |   await scriptMentor({ times: 2, verdicts: { substance: 'Failed' } });
  88  | 
  89  |   await runRubricAction(page, 'Submit for review');
  90  |   await expect(page.getByText('NOT MET YET', { exact: true })).toBeVisible({ timeout: 60_000 });
  91  |   const substance = criterionRow(page, SUBSTANCE);
  92  |   await expect(substance.getByText('Not met', { exact: true })).toBeVisible();
  93  |   await expect(substance.getByText('Review 1 of 2')).toBeVisible();
  94  |   await expect(substance).toContainText('Published criterion not met yet.');
  95  |   await expect(criterionRow(page, REASONING).getByText('Passed', { exact: true })).toBeVisible();
  96  |   await expect(page.getByRole('button', { name: 'Accept with note...' })).toHaveCount(0);
  97  | 
  98  |   await runRubricAction(page, 'Submit for review');
> 99  |   await expect(substance.getByText('Review 2 of 2')).toBeVisible({ timeout: 60_000 });
      |                                                      ^ Error: expect(locator).toBeVisible() failed
  100 |   const offer = page.getByRole('button', { name: 'Accept with note...' });
  101 |   await expect(offer).toBeVisible();
  102 |   expect(await mentorReviewCalls()).toBe(2);
  103 |   expect(
  104 |     (await unitCriteria(page, courseRef, unit.unit_ref)).find((c) => c.criterion_id.includes('substance')),
  105 |   ).toMatchObject({ rejection_count: 2, accept_with_note_offered: true });
  106 | 
  107 |   // The dialog opens on the safe choice; Keep working changes nothing.
  108 |   await offer.click();
  109 |   const dialog = page.getByRole('dialog', { name: 'Accept without Mentor confirmation?' });
  110 |   await expect(dialog).toBeVisible();
  111 |   await expect(dialog.getByRole('button', { name: 'Keep working', exact: true })).toBeFocused();
  112 |   await dialog.getByRole('button', { name: 'Keep working', exact: true }).click();
  113 |   await expect(dialog).toBeHidden();
  114 |   await expect(offer).toBeVisible();
  115 | 
  116 |   // Confirm: the unit completes WITH NOTE (every required criterion is met or accepted).
  117 |   await offer.click();
  118 |   await dialog.getByRole('button', { name: 'Accept with note', exact: true }).click();
  119 |   await expect(dialog).toBeHidden();
  120 |   await expect(page.getByText('PASSED WITH NOTE', { exact: true })).toBeVisible({ timeout: 30_000 });
  121 |   await expect(page.getByText('Accepted with note', { exact: true })).toBeVisible();
  122 |   await expect(
  123 |     page.getByRole('button', { name: /Foundations of .* \(completed, not confirmed by Mentor\)/ }),
  124 |   ).toBeVisible();
  125 |   expect((await unitCriteria(page, courseRef, unit.unit_ref)).find((c) => c.criterion_id.includes('substance'))).toMatchObject({
  126 |     status: 'accepted_with_note',
  127 |   });
  128 | 
  129 |   // The library card discloses the mark.
  130 |   await gotoLearnerApp(page);
  131 |   const card = page.locator('article').filter({ has: page.getByRole('heading', { name: title, exact: true }) });
  132 |   await expect(card.getByText('1 unit accepted with note (not confirmed by Mentor).')).toBeVisible({ timeout: 30_000 });
  133 |   await card.getByRole('button', { name: /Continue/ }).first().click();
  134 |   await page.getByRole('button', { name: /\(completed, not confirmed by Mentor\)/ }).click();
  135 |   await openAnswerTab(page);
  136 | 
  137 |   // A later rejection never revokes completion: the mark and the curriculum state stay.
  138 |   const completedButton = page.getByRole('button', {
  139 |     name: /Foundations of .* \(completed, not confirmed by Mentor\)/,
  140 |   });
  141 |   await scriptMentor({ verdicts: { substance: 'Failed' } });
  142 |   await page.getByRole('button', { name: 'Submit again for review', exact: true }).click();
  143 |   await expect.poll(() => mentorReviewCalls(), { timeout: 60_000 }).toBe(3);
  144 |   await expect(page.getByRole('button', { name: 'Submit again for review', exact: true })).toBeEnabled({
  145 |     timeout: 60_000,
  146 |   });
  147 |   await expect(page.getByText('PASSED WITH NOTE', { exact: true })).toBeVisible();
  148 |   await expect(completedButton).toBeVisible();
  149 |   expect((await unitCriteria(page, courseRef, unit.unit_ref)).find((c) => c.criterion_id.includes('substance'))).toMatchObject({
  150 |     status: 'accepted_with_note',
  151 |   });
  152 | 
  153 |   // A later Mentor pass clears the mark (the script is empty: every criterion passes).
  154 |   await page.getByRole('button', { name: 'Submit again for review', exact: true }).click();
  155 |   await expect(page.getByText('PASSED WITH NOTE', { exact: true })).toHaveCount(0, { timeout: 60_000 });
  156 |   await expect(page.getByText('PASSED', { exact: true })).toBeVisible();
  157 |   expect(await mentorReviewCalls()).toBe(4);
  158 |   expect((await unitCriteria(page, courseRef, unit.unit_ref)).every((c) => c.status !== 'accepted_with_note')).toBe(true);
  159 |   await gotoLearnerApp(page);
  160 |   await expect(
  161 |     page.locator('article').filter({ has: page.getByRole('heading', { name: title, exact: true }) }).getByText(/accepted with note/),
  162 |   ).toHaveCount(0);
  163 | });
  164 | 
  165 | test('text unit: Mentor unavailable or invalid shows amber MENTOR UNAVAILABLE and Retry, no attempt consumed (E6, E8)', async ({
  166 |   page,
  167 | }) => {
  168 |   const { courseRef } = await startModeCourse(page, 'e27-text-outage', 'text');
  169 |   const unit = await firstUnit(page, courseRef);
  170 |   await openAnswerTab(page);
  171 |   await writeAnswer(page, ANSWER);
  172 | 
  173 |   // E6: an unreachable provider is technical evidence: amber MENTOR UNAVAILABLE with Retry.
  174 |   await scriptMentor({ fault: 'unavailable' });
  175 |   await runRubricAction(page, 'Submit for review');
  176 |   await expect(page.getByText('MENTOR UNAVAILABLE', { exact: true })).toBeVisible({ timeout: 60_000 });
  177 |   await expect(page.getByText(/The AI Mentor did not respond \(provider unavailable\)/)).toBeVisible();
  178 |   await expect(page.getByText('NOT MET YET', { exact: true })).toHaveCount(0);
  179 |   await expect(page.getByText('UNMET CRITERIA', { exact: true })).toHaveCount(0);
  180 |   await expectNoRejections(page, courseRef, unit.unit_ref);
  181 |   expect(await mentorReviewCalls()).toBe(1);
  182 | 
  183 |   // E8: an envelope the application rejects is an invalid verdict, not an outage: the review
  184 |   // failed (approved copy), the attempt was not counted and no rejection is recorded.
  185 |   await scriptMentor({ fault: 'invalid' });
  186 |   await runRubricAction(page, 'Retry review');
  187 |   await expect(
  188 |     page.getByText('The Mentor review could not be completed. Your attempt was not counted. Try again.'),
  189 |   ).toBeVisible({ timeout: 60_000 });
  190 |   await expect(page.getByText('NOT MET YET', { exact: true })).toHaveCount(0);
  191 |   await expect(page.getByText('UNMET CRITERIA', { exact: true })).toHaveCount(0);
  192 |   await expectNoRejections(page, courseRef, unit.unit_ref);
  193 |   expect(await mentorReviewCalls()).toBe(2);
  194 | 
  195 |   // The provider is back; the same action, so the review passes and completes the unit.
  196 |   await runRubricAction(page, 'Submit for review');
  197 |   await expect(page.getByText('PASSED', { exact: true })).toBeVisible({ timeout: 60_000 });
  198 |   await expect(page.getByText('MENTOR UNAVAILABLE', { exact: true })).toHaveCount(0);
  199 |   expect(await mentorReviewCalls()).toBe(3);
```