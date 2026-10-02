# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: course-generation-epic25-recovery.spec.ts >> deterministic map refusal survives reload and retries only after explicit action
- Location: e2e/course-generation-epic25-recovery.spec.ts:196:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByRole('alert').filter({ hasText: 'Не удалось подготовить программу. Можно попробовать ещё раз.' })
Expected: visible
Timeout: 120000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 120000ms
  - waiting for getByRole('alert').filter({ hasText: 'Не удалось подготовить программу. Можно попробовать ещё раз.' })

```

```yaml
- banner:
  - button "Меню ученика"
- main:
  - button "В библиотеку"
  - list "Этапы создания":
    - listitem: ✓ Цель
    - listitem: ✓ Параметры
    - listitem: 3 Программа
    - listitem: 4 Создание
  - paragraph: Personal budget and reserve
  - text: Программа Готовим занятия
  - region "Проверьте программу":
    - paragraph: Программа
    - heading "Проверьте программу" [level=1]
    - text: Черновик v7
    - paragraph: Черновик готов. Уточните детали или создайте курс по этой программе.
    - heading "Personal budget and reserve draft" [level=2]
    - paragraph: Compute a free monthly balance and the months needed to reach a reserve target.
    - article:
      - paragraph: Настроим курс
      - paragraph: Необязательно
      - heading "Should the draft favor a deeper core or a shorter course?" [level=3]
      - paragraph: "Почему спрашиваем: The module depth follows this bounded personalization signal."
      - paragraph: "Что изменится: The chosen answer rewrites the affected plan modules."
      - group "Should the draft favor a deeper core or a shorter course?":
        - radio "Keep depth Keep the module shape."
        - text: Keep depth Keep the module shape.
        - radio "Shorten Trim the module scope."
        - text: Shorten Trim the module scope.
      - button "Оставить как в черновике"
      - button "Доверьте выбор программе"
    - article:
      - paragraph: Настроим курс
      - paragraph: Необязательно
      - heading "Should the draft add one guided practice unit?" [level=3]
      - paragraph: "Почему спрашиваем: The draft depends on this bounded personalization signal."
      - paragraph: "Что изменится: The chosen answer rewrites the affected plan modules."
      - group "Should the draft add one guided practice unit?":
        - radio "Keep No extra guided unit."
        - text: Keep No extra guided unit.
        - radio "Add one Add one guided unit."
        - text: Add one Add one guided unit.
      - button "Оставить как в черновике"
      - button "Доверьте выбор программе"
    - region "Модули программы":
      - heading "Модули программы" [level=3]
      - group: 01 Foundations of Personal budget and reserve 1440 мин Core ideas of Personal budget and reserve
    - paragraph: Принятые допущения
    - list:
      - listitem: Modules follow a foundations-first order picked by the program, not a stated learner preference.
    - region "Модули по этапам":
      - heading "Модули по этапам" [level=3]
      - list:
        - listitem: "Этап 1: Foundations of Personal budget and reserve"
    - paragraph: Compute a free monthly balance and the months needed to reach a reserve target.
    - paragraph: Как проверим
    - list:
      - listitem: "lf158-group-01-practice: implementation evidence via published criteria"
      - listitem: "lf158-group-01-transfer: transfer evidence via published criteria"
    - paragraph: Что не проверяем
    - list:
      - listitem: Free-form explanations are not graded as proof of solution quality.
      - listitem: No income, investment or financial outcomes are promised or checked.
      - listitem: Transfer is checked in one neighboring context only.
    - status:
      - paragraph: "Отмечено карточек: 0"
      - paragraph: Отметьте ответ, пропуск или выбор программы хотя бы в одной карточке.
    - button "Применить ответы" [disabled]
    - list
    - button "Изменить"
    - button "Назад"
    - complementary:
      - heading "Параметры курса" [level=2]
      - button "Изменить параметры"
      - term: Ваш опыт
      - definition: Есть небольшой опыт
      - term: Время в неделю, часов
      - definition: "31"
      - term: На сколько недель?
      - definition: "32"
      - term: Язык курса
      - definition: Русский
    - button "Подтвердить и создать курс"
```

# Test source

```ts
  124 | test.afterEach(async ({ page }) => {
  125 |   await drainAuthoringQueue(page);
  126 | });
  127 | 
  128 | test('first plan draft HTTP failure survives reload and offers an explicit retry', async ({ page }) => {
  129 |   await resetGenerationFake();
  130 |   await fakeControl('fail-next/propose_plan_draft');
  131 |   const description = subjectDescription('python');
  132 |   await openGenerator(page);
  133 |   await submitGoal(page, description);
  134 |   await expect(page.locator('#wizard-skill')).toHaveValue(description.split('\n')[0], {
  135 |     timeout: 60_000,
  136 |   });
  137 |   const continueButton = page.getByRole('button', { name: 'Показать программу' });
  138 |   await expect(continueButton).toBeEnabled({ timeout: 60_000 });
  139 |   await continueButton.click();
  140 | 
  141 |   // The provider call is a real failed operation with a durable recovery offer.
  142 |   await expect(page.getByText('Не удалось подготовить программу. Можно попробовать ещё раз.'))
  143 |     .toBeVisible({ timeout: 60_000 });
  144 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  145 | 
  146 |   await page.reload();
  147 |   await gotoLearnerApp(page);
  148 |   await openDraftByPhaseAndAction(
  149 |     page,
  150 |     description.split('\n')[0],
  151 |     'plan_draft',
  152 |     'draft_retry',
  153 |   );
  154 |   const retry = page.getByRole('button', { name: 'Повторить подготовку' });
  155 |   await expect(retry).toBeVisible({ timeout: 30_000 });
  156 |   await retry.click();
  157 | 
  158 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  159 |     timeout: 120_000,
  160 |   });
  161 |   expect(await countCalls('propose_plan_draft')).toBe(2);
  162 |   expect(await countCalls('propose_learning_map')).toBe(0);
  163 |   expect(await countCalls('materialize_unit')).toBe(0);
  164 | });
  165 | 
  166 | test('wrong planner schema version gives bounded failure and recovers on retry', async ({ page }) => {
  167 |   await resetGenerationFake();
  168 |   await fakeControl('wrong-version-next/propose_plan_draft');
  169 |   const description = subjectDescription('finance');
  170 |   await openGenerator(page);
  171 |   await submitGoal(page, description);
  172 |   await expect(page.locator('#wizard-skill')).toHaveValue(description.split('\n')[0], {
  173 |     timeout: 60_000,
  174 |   });
  175 |   const continueButton = page.getByRole('button', { name: 'Показать программу' });
  176 |   await expect(continueButton).toBeEnabled({ timeout: 60_000 });
  177 |   await continueButton.click();
  178 | 
  179 |   const failure = page.getByRole('alert').filter({
  180 |     hasText: 'Не удалось подготовить программу. Можно попробовать ещё раз.',
  181 |   });
  182 |   await expect(failure).toBeVisible({ timeout: 60_000 });
  183 |   await expect(page.locator('body')).not.toContainText('schema_version');
  184 |   await expect(page.locator('body')).not.toContainText('operation_id');
  185 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  186 | 
  187 |   await page.getByRole('button', { name: 'Повторить подготовку' }).click();
  188 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeEnabled({
  189 |     timeout: 120_000,
  190 |   });
  191 |   expect(await countCalls('propose_plan_draft')).toBe(2);
  192 |   expect(await countCalls('propose_learning_map')).toBe(0);
  193 |   expect(await countCalls('materialize_unit')).toBe(0);
  194 | });
  195 | 
  196 | test('deterministic map refusal survives reload and retries only after explicit action', async ({ page }) => {
  197 |   await resetGenerationFake();
  198 |   await fakeControl('map-refusal-next/propose_plan_draft');
  199 |   const planWatcher = watchSessionPlanDigest(page);
  200 |   let mapRetryPosts = 0;
  201 |   let mapConsentPosts = 0;
  202 |   let planConsentPosts = 0;
  203 |   page.on('request', (request) => {
  204 |     if (request.method() !== 'POST') return;
  205 |     const path = new URL(request.url()).pathname;
  206 |     if (/\/api\/v1\/authoring\/sessions\/[^/]+\/map-retry$/.test(path)) mapRetryPosts += 1;
  207 |     if (/\/api\/v1\/authoring\/sessions\/[^/]+\/map-confirmation$/.test(path)) mapConsentPosts += 1;
  208 |     if (/\/api\/v1\/authoring\/sessions\/[^/]+\/plan-confirmation$/.test(path)) planConsentPosts += 1;
  209 |   });
  210 | 
  211 |   const description = subjectDescription('finance');
  212 |   await openGenerator(page);
  213 |   await submitGoal(page, description);
  214 |   await expect(page.locator('#wizard-skill')).toHaveValue(description.split('\n')[0], {
  215 |     timeout: 60_000,
  216 |   });
  217 |   const continueButton = page.getByRole('button', { name: 'Показать программу' });
  218 |   await expect(continueButton).toBeEnabled({ timeout: 60_000 });
  219 |   await continueButton.click();
  220 | 
  221 |   const failure = page.getByRole('alert').filter({
  222 |     hasText: 'Не удалось подготовить программу. Можно попробовать ещё раз.',
  223 |   });
> 224 |   await expect(failure).toBeVisible({ timeout: 120_000 });
      |                         ^ Error: expect(locator).toBeVisible() failed
  225 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  226 |   expect(await countCalls('propose_learning_map')).toBe(0);
  227 |   expect(mapRetryPosts).toBe(0);
  228 |   expect(mapConsentPosts).toBe(0);
  229 |   expect(planConsentPosts).toBe(0);
  230 |   await expect(page.locator('body')).not.toContainText('schema_version');
  231 |   await expect(page.locator('body')).not.toContainText('operation_id');
  232 | 
  233 |   const failedRef = planWatcher.seenRef();
  234 |   if (failedRef === '') throw new Error('plan watcher never saw its authoring session');
  235 |   const failed = await apiGet(page, `/api/v1/authoring/sessions/${failedRef}`);
  236 |   const failedDraft = failed['draft'] as { draft_digest?: string } | null;
  237 |   expect(failedDraft?.draft_digest).toBeTruthy();
  238 |   expect(failed['available_actions']).toEqual(['map_retry', 'revise', 'delete']);
  239 |   expect(failed['error']).toEqual({
  240 |     code: 'failed:map_preparation',
  241 |     message: 'The program could not be prepared. Retry preparation.',
  242 |   });
  243 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toBeDisabled();
  244 | 
  245 |   await page.reload();
  246 |   await gotoLearnerApp(page);
  247 |   await openDraftByDigest(page, description.split('\n')[0], failedDraft?.draft_digest ?? '');
  248 |   const retry = page.getByRole('button', { name: 'Повторить подготовку' });
  249 |   await expect(retry).toBeVisible({ timeout: 30_000 });
  250 |   expect(await countCalls('propose_learning_map')).toBe(0);
  251 |   await retry.click();
  252 | 
  253 |   await expect
  254 |     .poll(async () => {
  255 |       const session = await apiGet(page, `/api/v1/authoring/sessions/${failedRef}`);
  256 |       return (session['available_actions'] as string[]).includes('map_retry');
  257 |     }, { timeout: 120_000 })
  258 |     .toBe(true);
  259 |   await expect(failure).toBeVisible({ timeout: 120_000 });
  260 |   expect(mapRetryPosts).toBe(1);
  261 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  262 |   expect(await countCalls('propose_learning_map')).toBe(0);
  263 |   expect(await countCalls('materialize_unit')).toBe(0);
  264 |   expect(mapConsentPosts).toBe(0);
  265 |   expect(planConsentPosts).toBe(0);
  266 |   const retried = await apiGet(page, `/api/v1/authoring/sessions/${failedRef}`);
  267 |   expect(retried['available_actions']).toEqual(['map_retry', 'revise', 'delete']);
  268 |   expect(retried['error']).toEqual(failed['error']);
  269 | });
  270 | 
  271 | test('missing core direction renders its concrete blocker and keeps Create unavailable', async ({ page }) => {
  272 |   await resetGenerationFake();
  273 |   await fakeControl('block-next/propose_plan_draft');
  274 |   const description = subjectDescription('management');
  275 |   await openGenerator(page);
  276 |   await submitGoal(page, description);
  277 |   await expect(page.locator('#wizard-skill')).toHaveValue(description.split('\n')[0], {
  278 |     timeout: 60_000,
  279 |   });
  280 |   const continueButton = page.getByRole('button', { name: 'Показать программу' });
  281 |   await expect(continueButton).toBeEnabled({ timeout: 60_000 });
  282 |   await continueButton.click();
  283 | 
  284 |   await expect(page.getByText('Нужен ваш ответ')).toBeVisible({ timeout: 60_000 });
  285 |   await expect(page.getByTestId('blocker-direction')).toContainText('Не хватает ключевого направления');
  286 |   await expect(page.getByText('Which course direction should the draft take?')).toBeVisible();
  287 |   await expect(page.getByRole('button', { name: 'Подтвердить и создать курс' })).toHaveCount(0);
  288 |   expect(await countCalls('propose_plan_draft')).toBe(1);
  289 |   expect(await countCalls('propose_learning_map')).toBe(0);
  290 |   expect(await countCalls('materialize_unit')).toBe(0);
  291 | });
  292 | 
  293 | test('revised draft refreshes current module coverage before consent', async ({ page }) => {
  294 |   await resetGenerationFake();
  295 |   const planWatcher = watchSessionPlanDigest(page);
  296 |   await driveToConfirmedPlan(page, 'python');
  297 |   await expect.poll(() => planWatcher.planDigest(), { timeout: 30_000 }).not.toBe('');
  298 |   const previousPlanDigest = planWatcher.planDigest();
  299 |   await expect.poll(() => planWatcher.digest(), { timeout: 30_000 }).not.toBe('');
  300 |   const previousBlueprintDigest = planWatcher.digest();
  301 |   const sessionRef = planWatcher.seenRef();
  302 |   if (sessionRef === '') throw new Error('plan watcher never saw its authoring session');
  303 |   const current = await apiGet(page, `/api/v1/authoring/sessions/${sessionRef}`);
  304 |   const currentPlan = current['plan'] as { plan_digest?: string } | null;
  305 |   expect(currentPlan?.plan_digest).toBe(previousPlanDigest);
  306 |   const revisedPlanWatcher = watchSessionPlanDigest(page, sessionRef);
  307 | 
  308 |   let confirmationPosts = 0;
  309 |   page.on('request', (request) => {
  310 |     if (
  311 |       request.method() === 'POST' &&
  312 |       /\/api\/v1\/authoring\/sessions\/[^/]+\/plan-confirmation$/.test(
  313 |         new URL(request.url()).pathname,
  314 |       )
  315 |     ) {
  316 |       confirmationPosts += 1;
  317 |     }
  318 |   });
  319 |   await answerFirstDraftQuestion(page);
  320 | 
  321 |   await expect
  322 |     .poll(
  323 |       () => {
  324 |         const digest = revisedPlanWatcher.digest();
```