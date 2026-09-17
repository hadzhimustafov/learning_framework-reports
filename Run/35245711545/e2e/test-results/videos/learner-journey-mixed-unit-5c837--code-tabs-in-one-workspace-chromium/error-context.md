# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: learner-journey.spec.ts >> mixed unit serves theory, quiz and code tabs in one workspace
- Location: e2e/learner-journey.spec.ts:81:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: getByRole('heading', { name: 'Mixed Course', exact: true })
Expected: visible
Timeout: 15000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 15000ms
  - waiting for getByRole('heading', { name: 'Mixed Course', exact: true })

```

```yaml
- main:
  - alert: A newer build of the learner workspace is available. Please reload the page.
```

# Test source

```ts
  1   | /**
  2   |  * Shared e2e helpers: session entry and course entry that is robust to whether the host-trust
  3   |  * confirmation is still required for the shared tech-course fixture. The session cookie is
  4   |  * established once by ``global-setup.ts`` and injected as the run-wide storage state, so tests
  5   |  * simply navigate to ``/app/``.
  6   |  */
  7   | import { expect, type Page } from '@playwright/test';
  8   | 
  9   | /** Navigate to the already-authenticated SPA shell and wait for it to hydrate. */
  10  | export async function gotoLearnerApp(page: Page): Promise<void> {
  11  |   await page.goto('/app/');
  12  |   await expect(page.locator('#root')).not.toBeEmpty();
  13  | }
  14  | 
  15  | /** Grant host trust when the confirmation dialog is present; a no-op once trust is recorded. */
  16  | export async function grantTrustIfPresent(page: Page): Promise<void> {
  17  |   const dialog = page.getByRole('dialog', { name: 'Grant Host Trust' });
  18  |   try {
  19  |     await dialog.waitFor({ state: 'visible', timeout: 5_000 });
  20  |     await dialog.getByRole('button', { name: 'Grant Trust & Continue' }).click();
  21  |     await dialog.waitFor({ state: 'detached', timeout: 5_000 });
  22  |   } catch {
  23  |     // Already trusted for this course; there is nothing to confirm.
  24  |   }
  25  | }
  26  | 
  27  | /** Wait for the workspace shell to render (the adaptive Theory tab is always available first). */
  28  | export async function waitForWorkspace(page: Page): Promise<void> {
  29  |   await expect(page.getByRole('tab', { name: 'Theory' })).toBeVisible({ timeout: 15_000 });
  30  | }
  31  | 
  32  | /**
  33  |  * Start the course with the given catalog title from the library and reach its workspace,
  34  |  * confirming trust when needed. Robust to prior runs: the card action may be "Start" or
  35  |  * "Continue".
  36  |  */
  37  | export async function enterCourseWorkspace(page: Page, courseTitle: string): Promise<void> {
> 38  |   await expect(page.getByRole('heading', { name: courseTitle, exact: true })).toBeVisible({
      |                                                                               ^ Error: expect(locator).toBeVisible() failed
  39  |     timeout: 15_000,
  40  |   });
  41  |   const card = page.locator('article').filter({
  42  |     has: page.getByRole('heading', { name: courseTitle, exact: true }),
  43  |   });
  44  |   await card.getByRole('button', { name: /Start|Continue/ }).click();
  45  |   await grantTrustIfPresent(page);
  46  |   await waitForWorkspace(page);
  47  | }
  48  | 
  49  | /** Start the shared tech course (used by the accessibility and responsive suites). */
  50  | export async function enterTechCourseWorkspace(page: Page): Promise<void> {
  51  |   await enterCourseWorkspace(page, 'Tech Course');
  52  | }
  53  | 
  54  | /** Start the mixed-capability course (theory + quiz + code in one unit). */
  55  | export async function enterMixedCourseWorkspace(page: Page): Promise<void> {
  56  |   await enterCourseWorkspace(page, 'Mixed Course');
  57  | }
  58  | 
  59  | /** Start the mentor-only tech clone (theory unit, never mutated by other suites). */
  60  | export async function enterMentorCourseWorkspace(page: Page): Promise<void> {
  61  |   await enterCourseWorkspace(page, 'Mentor Course');
  62  | }
  63  | 
  64  | /** Start the mentor-only mixed clone (Theory/Quiz/Practice chips, never advanced to transfer). */
  65  | export async function enterMentorMixedCourseWorkspace(page: Page): Promise<void> {
  66  |   await enterCourseWorkspace(page, 'Mentor Mixed Course');
  67  | }
  68  | 
  69  | /** Start the mentor-review course whose Check work is ``mentor_review`` authority. */
  70  | export async function enterReviewCourseWorkspace(page: Page): Promise<void> {
  71  |   await enterCourseWorkspace(page, 'Review Course');
  72  | }
  73  | 
  74  | /** Start the isolated cold-transfer course (Mentor unavailable from the first unit). */
  75  | export async function enterColdTransferWorkspace(page: Page): Promise<void> {
  76  |   await enterCourseWorkspace(page, 'Cold Transfer Course');
  77  | }
  78  | 
  79  | /**
  80  |  * Open the Support-rail AI Mentor tab. Below 1200px the Support column is reached through the
  81  |  * compact view toggle; at desktop width the rail is already visible.
  82  |  */
  83  | export async function openMentorTab(page: Page): Promise<void> {
  84  |   const supportToggle = page.getByRole('button', { name: 'Support', exact: true });
  85  |   if (await supportToggle.isVisible()) {
  86  |     await supportToggle.click();
  87  |   }
  88  |   const mentorTab = page.getByRole('tab', { name: 'AI Mentor' });
  89  |   await expect(mentorTab).toBeVisible({ timeout: 15_000 });
  90  |   await mentorTab.click();
  91  |   await expect(mentorTab).toHaveAttribute('aria-selected', 'true');
  92  | }
  93  | 
  94  | /** Start the journey-only course clone whose progression state is pristine per run. */
  95  | export async function enterJourneyCourseWorkspace(page: Page): Promise<void> {
  96  |   await enterCourseWorkspace(page, 'Journey Course');
  97  | }
  98  | 
  99  | /** Minimal unit summary shape the e2e helpers read from the real workspace view. */
  100 | interface TechCourseUnit {
  101 |   unit_ref: string;
  102 |   unit_id: string;
  103 |   state: string;
  104 | }
  105 | 
  106 | /**
  107 |  * Drive a real API request with the same bounded 423 retry the SPA client performs: concurrent
  108 |  * learner requests serialize on the exclusive Course Lock, so a request that collides with a
  109 |  * milliseconds-scale critical section must retry per the published Retry-After contract instead
  110 |  * of failing the helper on a transient contention window.
  111 |  */
  112 | async function apiWithLockRetry(
  113 |   page: Page,
  114 |   method: 'get' | 'post',
  115 |   url: string,
  116 |   init?: { data?: unknown; headers?: Record<string, string> },
  117 | ): Promise<import('@playwright/test').APIResponse> {
  118 |   const LOCK_RETRY_LIMIT = 3;
  119 |   for (let attempt = 0; ; attempt += 1) {
  120 |     const response =
  121 |       method === 'get'
  122 |         ? await page.request.get(url)
  123 |         : await page.request.post(url, {
  124 |             data: init?.data,
  125 |             headers: init?.headers,
  126 |           });
  127 |     if (response.status() !== 423 || attempt >= LOCK_RETRY_LIMIT) {
  128 |       return response;
  129 |     }
  130 |     await page.waitForTimeout(250);
  131 |   }
  132 | }
  133 | 
  134 | /** Minimal workspace-view shape the e2e helpers read from the real API (no mocks). */
  135 | interface TechCourseWorkspaceView {
  136 |   course_ref: string;
  137 |   available_actions: string[];
  138 |   stages: Array<{ units: TechCourseUnit[] }>;
```