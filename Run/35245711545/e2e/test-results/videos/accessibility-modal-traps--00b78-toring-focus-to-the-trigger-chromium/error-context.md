# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: accessibility.spec.ts >> modal traps focus and closes on Escape, restoring focus to the trigger
- Location: e2e/accessibility.spec.ts:52:1

# Error details

```
Test timeout of 300000ms exceeded.
```

```
Error: locator.click: Test timeout of 300000ms exceeded.
Call log:
  - waiting for locator('article').filter({ hasText: 'Modal Fixture' }).first().getByRole('button', { name: 'Start' })

```

# Page snapshot

```yaml
- main [ref=e3]:
  - alert [ref=e4]: A newer build of the learner workspace is available. Please reload the page.
```

# Test source

```ts
  1   | import { expect, test } from '@playwright/test';
  2   | import { AxeBuilder } from '@axe-core/playwright';
  3   | import {
  4   |   completeTheoryGateViaApi,
  5   |   ensurePracticeCheckWorkAuthorized,
  6   |   enterTechCoursePracticeWorkspace,
  7   |   enterTechCourseWorkspace,
  8   |   fillEditor,
  9   |   gotoLearnerApp,
  10  |   grantTrustIfPresent,
  11  |   openPracticeUnit,
  12  | } from './support';
  13  | 
  14  | test('keyboard navigation: Tab, Shift+Tab and Enter/Space activation', async ({ page }) => {
  15  |   await gotoLearnerApp(page);
  16  | 
  17  |   const firstAction = page
  18  |     .locator('article', { hasText: 'Tech Course' })
  19  |     .first()
  20  |     .getByRole('button', { name: /Start|Continue/ });
  21  |   await firstAction.focus();
  22  |   await expect(firstAction).toBeFocused();
  23  | 
  24  |   // Shift+Tab moves focus backwards out of the button; Tab returns to it.
  25  |   await page.keyboard.press('Shift+Tab');
  26  |   await expect(firstAction).not.toBeFocused();
  27  |   await page.keyboard.press('Tab');
  28  |   await expect(firstAction).toBeFocused();
  29  | 
  30  |   // Enter activates the focused button to start the course (confirming trust when required).
  31  |   await page.keyboard.press('Enter');
  32  |   await grantTrustIfPresent(page);
  33  |   await expect(page.getByRole('tab', { name: 'Theory' })).toBeVisible({ timeout: 15_000 });
  34  | 
  35  |   // The entry unit is the theory quiz gate (no editor); reach the practice unit's editor by
  36  |   // passing the gate through the real evaluation API, then advancing and verifying the backend
  37  |   // state so u1 (Practice one) is the server-tracked in-progress unit with Check work
  38  |   // authorized — never the completed u0 offering "Next unit" instead.
  39  |   await completeTheoryGateViaApi(page);
  40  |   await ensurePracticeCheckWorkAuthorized(page);
  41  |   await page.reload();
  42  |   await enterTechCourseWorkspace(page);
  43  |   await openPracticeUnit(page);
  44  | 
  45  |   // Workspace tabs are real buttons: Space activates the focused tab.
  46  |   const workspaceTab = page.getByRole('tab', { name: 'Code' });
  47  |   await workspaceTab.focus();
  48  |   await page.keyboard.press('Space');
  49  |   await expect(page.locator('.cm-content')).toBeVisible();
  50  | });
  51  | 
  52  | test('modal traps focus and closes on Escape, restoring focus to the trigger', async ({ page }) => {
  53  |   await gotoLearnerApp(page);
  54  | 
  55  |   const startButton = page
  56  |     .locator('article', { hasText: 'Modal Fixture' })
  57  |     .first()
  58  |     .getByRole('button', { name: 'Start' });
> 59  |   await startButton.click();
      |                     ^ Error: locator.click: Test timeout of 300000ms exceeded.
  60  | 
  61  |   const dialog = page.getByRole('dialog', { name: 'Grant Host Trust' });
  62  |   await expect(dialog).toBeVisible({ timeout: 10_000 });
  63  | 
  64  |   // Focus is held inside the dialog (the grant panel becomes interactive once the preview loads).
  65  |   await expect(
  66  |     page.getByRole('button', { name: 'Grant Trust & Continue' }),
  67  |   ).toBeEnabled({ timeout: 10_000 });
  68  |   await page.waitForFunction(() => {
  69  |     const el = document.querySelector('[role="dialog"]');
  70  |     return el !== null && el.contains(document.activeElement);
  71  |   });
  72  | 
  73  |   // Tab from the last focusable wraps back to the first focusable inside the dialog.
  74  |   await page.getByRole('button', { name: 'Decline' }).focus();
  75  |   await page.keyboard.press('Tab');
  76  |   await page.waitForFunction(() => {
  77  |     const el = document.querySelector('[role="dialog"]');
  78  |     return el !== null && el.contains(document.activeElement);
  79  |   });
  80  |   await expect(
  81  |     page.getByRole('button', { name: 'Close trust dialog without granting' }),
  82  |   ).toBeFocused();
  83  | 
  84  |   // Escape dismisses the dialog and returns focus to the invoking control.
  85  |   await page.keyboard.press('Escape');
  86  |   await expect(dialog).toBeHidden();
  87  |   await expect(startButton).toBeFocused();
  88  | });
  89  | 
  90  | test('ARIA live regions announce save status and evaluation progress', async ({ page }) => {
  91  |   // CI budget: entering the practice workspace plus the save round-trip can exceed the
  92  |   // default 30s timeout on loaded runners.
  93  |   test.setTimeout(60_000);
  94  | 
  95  |   await gotoLearnerApp(page);
  96  |   await enterTechCoursePracticeWorkspace(page);
  97  |   await page.getByRole('tab', { name: 'Code' }).click();
  98  | 
  99  |   // Wait for the editor and the Check work action while the Code tab is active: the side-only
  100 |   // policy hides Check work unless the Code tab is selected, and the entry helper has already
  101 |   // verified the server authorizes check_work (u1 in_progress), so the button must not have
  102 |   // been replaced by "Next unit".
  103 |   await expect(page.locator('.cm-content')).toBeVisible();
  104 |   const checkWork = page.getByRole('button', { name: 'Check work' });
  105 |   await expect(checkWork).toBeVisible({ timeout: 15_000 });
  106 | 
  107 |   const saveStatus = page.getByRole('status', { name: /Save status/ }).first();
  108 |   await expect(saveStatus).toBeVisible();
  109 | 
  110 |   // Paste (not type) the probe: DOM-level typing corrupts whitespace in CodeMirror.
  111 |   await fillEditor(page, "def value():\n    return ''  # a11y probe");
  112 |   await expect(page.getByText('Unsaved changes')).toBeVisible();
  113 |   await expect(page.getByText('Saved')).toBeVisible({ timeout: 10_000 });
  114 | 
  115 |   // Evaluation progress and its terminal result are both polite live regions.
  116 |   await checkWork.click();
  117 |   await expect(page.getByText('Checking your work…')).toBeVisible();
  118 |   await expect(page.getByText('UNMET CRITERIA')).toBeVisible({ timeout: 20_000 });
  119 | });
  120 | 
  121 | test('axe-core: course library page has zero accessibility violations', async ({ page }) => {
  122 |   await gotoLearnerApp(page);
  123 | 
  124 |   // Wait for the catalog to finish hydrating so the skeleton is not the audited surface.
  125 |   await expect(page.getByText('Tech Course')).toBeVisible({ timeout: 15_000 });
  126 | 
  127 |   const results = await new AxeBuilder({ page }).analyze();
  128 |   expect(results.violations).toEqual([]);
  129 | });
  130 | 
  131 | test('axe-core: workspace page has zero accessibility violations', async ({ page }) => {
  132 |   await gotoLearnerApp(page);
  133 |   await enterTechCoursePracticeWorkspace(page);
  134 |   await page.getByRole('tab', { name: 'Code' }).click();
  135 |   await expect(page.locator('.cm-content')).toBeVisible();
  136 | 
  137 |   const results = await new AxeBuilder({ page }).analyze();
  138 |   expect(results.violations).toEqual([]);
  139 | });
  140 | 
  141 | test('axe-core: trust modal has zero accessibility violations', async ({ page }) => {
  142 |   await gotoLearnerApp(page);
  143 | 
  144 |   const startButton = page
  145 |     .locator('article', { hasText: 'Modal Fixture' })
  146 |     .first()
  147 |     .getByRole('button', { name: 'Start' });
  148 |   await startButton.click();
  149 | 
  150 |   const dialog = page.getByRole('dialog', { name: 'Grant Host Trust' });
  151 |   await expect(dialog).toBeVisible({ timeout: 10_000 });
  152 |   await expect(page.getByRole('button', { name: 'Grant Trust & Continue' })).toBeEnabled({
  153 |     timeout: 10_000,
  154 |   });
  155 | 
  156 |   const results = await new AxeBuilder({ page }).include('[role="dialog"]').analyze();
  157 |   expect(results.violations).toEqual([]);
  158 | });
```