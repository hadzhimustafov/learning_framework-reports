# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: evaluation-modes-table-code.spec.ts >> table unit: a failing check skips the Mentor; a passing check starts the review automatically (E5, E7, E8)
- Location: e2e/evaluation-modes-table-code.spec.ts:86:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: locator('.verdict__t .state-label').getByText('Passed', { exact: true })
Expected: visible
Timeout: 60000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 60000ms
  - waiting for locator('.verdict__t .state-label').getByText('Passed', { exact: true })

```

```yaml
- banner:
  - button "Learning Framework, в библиотеку": LF
  - navigation "Путь":
    - button "Обучение"
    - paragraph: Household budget table with a Mentor review
  - tablist "Панели главы":
    - tab "Теория"
    - tab "Код" [selected]
  - button "Оглавление" [pressed]
  - button "Поддержка" [pressed]
  - button "Читать R"
  - button "Меню ученика"
  - text: "Локальный сервер: доступен"
  - status
- complementary "Оглавление курса":
  - paragraph: Оглавление
  - button "Скрыть оглавление"
  - navigation "Оглавление курса":
    - heading "Household budget table with a Mentor review" [level=2]
    - text: 0 из 2
    - heading "Этап I · Foundations of Household budget table with a Mentor review" [level=3]
    - 'button "Переработать модуль: Этап I · Foundations of Household budget table with a Mentor review"'
    - list:
      - listitem:
        - button "Foundations of Household budget table with a Mentor review (в работе, проверка и отзыв ИИ-ментора)": 01 Foundations of Household budget table with a Mentor review Вы здесь
      - listitem:
        - button "Foundations of Household budget table with a Mentor review (закрыта, проверка и отзыв ИИ-ментора)" [disabled]: 02 Foundations of Household budget table with a Mentor review Откроется после предыдущих глав.
    - paragraph: Откроется после предыдущих глав.
  - paragraph: Версия v1
  - paragraph: generated-51869f8db9344806934096497355339c
- main "Рабочая область главы":
  - tabpanel "Код":
    - paragraph: Этап I · Глава 1
    - heading "Foundations of Household budget table with a Mentor review" [level=1]
    - region "Требования":
      - heading "Требования" [level=2]
      - text: 2 из 2 выполнено · проверка 18:51
      - button "Переработать"
      - button "Задание неверно"
      - list:
        - listitem: "R1 The answer is complete for: Explain and apply the core ideas of Household budget table with a Mentor review. Выполнено"
        - listitem: "R2 The answer is correct for: Explain and apply the core ideas of Household budget table with a Mentor review. Выполнено"
    - navigation "Файлы ответа":
      - heading "Файлы ответа" [level=2]
      - list:
        - listitem:
          - button "table.csv"
    - 'status "Состояние сохранения: Сохранено"': Сохранено
    - textbox "Редактор файла units/lf158-group-01-practice/workspace/learner/table.csv"
- complementary "Поддержка":
  - region "Поддержка":
    - tablist "Инструменты поддержки":
      - tab "Критерии" [selected]
      - tab "AI Mentor"
    - tabpanel "Критерии":
      - region "Критерии":
        - heading "Критерии" [level=2]
        - paragraph: Сначала проверка, затем отзыв AI Mentor.
        - status:
          - paragraph: Проверка ментором не завершилась. Попытка не засчитана. Попробуйте ещё раз.
        - paragraph: 2 of 5 criteria passed
        - status:
          - paragraph: Needs changes
          - paragraph: Проверка нашла проблему. Проверка ментором не запускалась.
        - paragraph:
          - button "Нужна подсказка? Открыть помощь"
        - group "Критерии главы":
          - heading "Проверка" [level=3]
          - 'list "Критерии: Проверка"':
            - listitem:
              - paragraph: "R1The answer is complete for: Explain and apply the core ideas of Household budget table with a Mentor review."
              - text: Passed
            - listitem:
              - paragraph: "R2The answer is correct for: Explain and apply the core ideas of Household budget table with a Mentor review."
              - text: Passed
          - heading "Проверка ментором" [level=3]
          - 'list "Критерии: Проверка ментором"':
            - listitem:
              - paragraph: "The answer correctly explains the core idea of: Explain and apply the core ideas of Household budget table with a Mentor review."
              - text: Не оценено
              - paragraph: Запустится после успешной проверки.
            - listitem:
              - paragraph: "The answer justifies its claims with concrete reasoning about: Explain and apply the core ideas of Household budget table with a Mentor review."
              - text: Не оценено
              - paragraph: Запустится после успешной проверки.
            - listitem:
              - paragraph: "The answer applies the idea to a specific case of: Explain and apply the core ideas of Household budget table with a Mentor review."
              - text: Не оценено
              - paragraph: Запустится после успешной проверки.
        - button "Проверить работу"
  - button "Скрыть панель"
```

# Test source

```ts
  32  |   scriptMentor,
  33  |   startModeCourse,
  34  |   unitCriteria,
  35  |   writeAnswer,
  36  | } from './evaluation-mode-support';
  37  | 
  38  | const PYTHON_WRONG = 'def double(n):\n    return n + 2\n';
  39  | const PYTHON_RIGHT = 'def double(n):\n    return n * 2\n';
  40  | const TABLE_WRONG = 'category,amount\nrent,100\nfood,80\n';
  41  | const TABLE_RIGHT = 'category,amount\nrent,120\nfood,80\n';
  42  | 
  43  | test.use(MODES_SERVER);
  44  | test.describe.configure({ mode: 'serial' });
  45  | 
  46  | test.beforeEach(async ({ page }) => {
  47  |   await drainAuthoringQueue(page);
  48  |   await resetMentorFake();
  49  | });
  50  | test.afterEach(async () => {
  51  |   await resetMentorFake();
  52  |   // The armed per-module format is sticky: never leak it into another spec file.
  53  |   await resetGenerationFake();
  54  | });
  55  | 
  56  | test('code unit: command mode keeps the check-only journey and never involves the Mentor (E10)', async ({
  57  |   page,
  58  | }) => {
  59  |   const { courseRef } = await startModeCourse(page, 'e27-code', 'code');
  60  |   const unit = await firstUnit(page, courseRef);
  61  |   expect(unit.evaluation_mode).toBe('command');
  62  |   await openCodeTab(page);
  63  | 
  64  |   // Command units keep the legacy wording: Check work, Needs changes, no Mentor groups or line.
  65  |   await expect(page.getByRole('button', { name: m('action.check'), exact: true })).toBeVisible();
  66  |   await expect(page.getByRole('button', { name: m('action.submit') })).toHaveCount(0);
  67  |   await expect(page.getByText(/Проверяет AI Mentor/)).toHaveCount(0);
  68  |   await expect(page.getByText('Проверено автоматически.')).toHaveCount(0);
  69  |   await fillEditor(page, PYTHON_WRONG);
  70  |   await waitCodeSaved(page);
  71  |   expect(await checkWork(page)).toBe('Needs changes');
  72  |   // S7 unified the verdict pill for every mode; a command verdict still has no Mentor group or line.
  73  |   await expect(page.getByRole('list', { name: 'Критерии: Проверка ментором' })).toHaveCount(0);
  74  |   await expect(page.getByText('Сначала проверка, затем отзыв AI Mentor.')).toHaveCount(0);
  75  | 
  76  |   await fillEditor(page, PYTHON_RIGHT);
  77  |   await waitCodeSaved(page);
  78  |   expect(await checkWork(page)).toBe('Passed');
  79  |   await waitForCourseProgress(page, courseRef, { total_units: 2, completed_units: 1, percent: 50 });
  80  |   expect(await mentorReviewCalls()).toBe(0);
  81  |   expect(
  82  |     (await unitCriteria(page, courseRef, unit.unit_ref)).every((item) => item.source === 'check'),
  83  |   ).toBe(true);
  84  | });
  85  | 
  86  | test('table unit: a failing check skips the Mentor; a passing check starts the review automatically (E5, E7, E8)', async ({
  87  |   page,
  88  | }) => {
  89  |   const { courseRef } = await startModeCourse(page, 'e27-table', 'table');
  90  |   const unit = await firstUnit(page, courseRef);
  91  |   expect(unit.evaluation_mode).toBe('hybrid');
  92  |   await openAnswerTab(page, m('chapter.tab.code'));
  93  |   await expect(page.getByText('Сначала проверка, затем отзыв AI Mentor.')).toBeVisible();
  94  |   await expect(page.getByRole('heading', { name: 'Проверка', exact: true })).toBeVisible();
  95  |   await expect(page.getByRole('heading', { name: 'Проверка ментором', exact: true })).toBeVisible();
  96  | 
  97  |   // Check fails (twice): the Mentor criteria are not evaluated, no review runs, and a deterministic
  98  |   // check criterion is never offered for accept-with-note.
  99  |   await writeAnswer(page, TABLE_WRONG);
  100 |   for (let attempt = 0; attempt < 2; attempt += 1) {
  101 |     await runRubricAction(page, m('action.check'));
  102 |     // S7: on a v3 unit the requirement pointer replaces the verdict summary; the Mentor rows below
  103 |     // still say the review did not run.
  104 |     await expect(
  105 |       page.locator('.verdict__t').getByText('0 из 2 выполнено · исправьте R1 и R2, затем проверьте снова'),
  106 |     ).toBeVisible({ timeout: 60_000 });
  107 |     const mentorRows = page.getByRole('list', { name: 'Критерии: Проверка ментором' }).getByRole('listitem');
  108 |     await expect(mentorRows).toHaveCount(3);
  109 |     for (const row of await mentorRows.all()) {
  110 |       await expect(row.getByText(m('rubric.state.not_evaluated'), { exact: true })).toBeVisible();
  111 |       await expect(row).toContainText(m('mode.runs_after_check'));
  112 |     }
  113 |     // Both requirement criteria (R1, R2) of the v3 check read «Needs changes».
  114 |     await expect(
  115 |       page.getByRole('list', { name: 'Критерии: Проверка' }).getByText('Needs changes', { exact: true }),
  116 |     ).toHaveCount(2);
  117 |     await expect(page.getByRole('button', { name: 'Принять с пометкой…' })).toHaveCount(0);
  118 |   }
  119 |   expect(await mentorReviewCalls()).toBe(0);
  120 |   const afterFailures = await unitCriteria(page, courseRef, unit.unit_ref);
  121 |   expect(afterFailures.filter((item) => item.source === 'mentor')).toHaveLength(3);
  122 |   expect(afterFailures.every((item) => (item.rejection_count ?? 0) === 0)).toBe(true);
  123 |   expect(afterFailures.every((item) => item.accept_with_note_offered !== true)).toBe(true);
  124 | 
  125 |   // Check passes: the review starts by itself (a held fake keeps the Reviewing state observable).
  126 |   await scriptMentor({ hold: true });
  127 |   await writeAnswer(page, TABLE_RIGHT);
  128 |   await runRubricAction(page, m('action.check'));
  129 |   await expect(page.getByRole('button', { name: m('action.reviewing') })).toBeDisabled({ timeout: 30_000 });
  130 |   await expect(page.getByText(m('rubric.state.reviewing'), { exact: true }).first()).toBeVisible();
  131 |   await releaseMentor();
> 132 |   await expect(page.locator('.verdict__t .state-label').getByText('Passed', { exact: true })).toBeVisible({ timeout: 60_000 });
      |                                                                                               ^ Error: expect(locator).toBeVisible() failed
  133 |   expect(await mentorReviewCalls()).toBe(1);
  134 |   await expect(criterionRow(page, 'correctly explains the core idea').getByText('Passed', { exact: true })).toBeVisible();
  135 |   const completed = await unitCriteria(page, courseRef, unit.unit_ref);
  136 |   expect(completed.every((item) => item.status !== 'accepted_with_note')).toBe(true);
  137 |   await waitForCourseProgress(page, courseRef, { total_units: 2, completed_units: 1, percent: 50 });
  138 | });
  139 | 
  140 | test('table unit completed with a note: Submit again for review re-checks it; a Mentor pass clears the mark, progress never changes (LF-207)', async ({
  141 |   page,
  142 | }) => {
  143 |   const { courseRef } = await startModeCourse(page, 'e27-table', 'table');
  144 |   const unit = await firstUnit(page, courseRef);
  145 |   expect(unit.evaluation_mode).toBe('hybrid');
  146 |   await openAnswerTab(page, m('chapter.tab.code'));
  147 |   await expect(page.getByRole('button', { name: m('mode.resubmit'), exact: true })).toHaveCount(0);
  148 | 
  149 |   // Two Mentor rejections of one criterion (the check passes both times; the bytes differ so the
  150 |   // second review is fresh), then accept with note: the unit completes with the mark.
  151 |   await scriptMentor({ times: 2, verdicts: { substance: 'Failed' } });
  152 |   await writeAnswer(page, TABLE_RIGHT);
  153 |   await runRubricAction(page, m('action.check'));
  154 |   await expect.poll(() => mentorReviewCalls(), { timeout: 60_000 }).toBe(1);
  155 |   await expect(page.getByText('Проверка 1 из 2')).toBeVisible();
  156 |   await writeAnswer(page, `${TABLE_RIGHT}\n`);
  157 |   await runRubricAction(page, m('action.check'));
  158 |   await expect.poll(() => mentorReviewCalls(), { timeout: 60_000 }).toBe(2);
  159 |   const offer = page.getByRole('button', { name: 'Принять с пометкой…' });
  160 |   await expect(offer).toBeVisible({ timeout: 60_000 });
  161 |   await offer.click();
  162 |   await page
  163 |     .getByRole('dialog', { name: 'Принять без подтверждения ментора?' })
  164 |     .getByRole('button', { name: 'Принять с пометкой', exact: true })
  165 |     .click();
  166 |   await expect(page.getByText('ПРИНЯТО С ПОМЕТКОЙ', { exact: true })).toBeVisible({ timeout: 30_000 });
  167 |   await waitForCourseProgress(page, courseRef, { total_units: 2, completed_units: 1, percent: 50 });
  168 | 
  169 |   // State 15 is offered beside Next unit; a Mentor rejection keeps the mark and the progress.
  170 |   const again = page.getByRole('button', { name: m('mode.resubmit'), exact: true });
  171 |   await expect(again).toBeVisible();
  172 |   await scriptMentor({ verdicts: { substance: 'Failed' } });
  173 |   await again.click();
  174 |   await expect.poll(() => mentorReviewCalls(), { timeout: 60_000 }).toBe(3);
  175 |   await expect(again).toBeEnabled({ timeout: 60_000 });
  176 |   await expect(page.getByText('ПРИНЯТО С ПОМЕТКОЙ', { exact: true })).toBeVisible();
  177 |   expect((await unitCriteria(page, courseRef, unit.unit_ref)).some((c) => c.status === 'accepted_with_note')).toBe(true);
  178 |   await expect(page.getByRole('button', { name: 'Принять с пометкой…' })).toHaveCount(0);
  179 |   await waitForCourseProgress(page, courseRef, { total_units: 2, completed_units: 1, percent: 50 });
  180 | 
  181 |   // A later Mentor pass clears the mark; the unit stays completed and the new result is shown.
  182 |   await again.click();
  183 |   await expect(page.getByText('ПРИНЯТО С ПОМЕТКОЙ', { exact: true })).toHaveCount(0, { timeout: 60_000 });
  184 |   await expect(page.locator('.verdict__t .state-label').getByText('Passed', { exact: true })).toBeVisible();
  185 |   expect(await mentorReviewCalls()).toBe(4);
  186 |   expect((await unitCriteria(page, courseRef, unit.unit_ref)).every((c) => c.status !== 'accepted_with_note')).toBe(true);
  187 |   await waitForCourseProgress(page, courseRef, { total_units: 2, completed_units: 1, percent: 50 });
  188 | });
  189 | 
```