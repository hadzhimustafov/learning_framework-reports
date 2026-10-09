# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: responsive-matrix.spec.ts >> narrow viewport: trust modal fits within screen bounds and dismisses on Escape
- Location: e2e/responsive-matrix.spec.ts:64:1

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
  1  | import { expect, test } from '@playwright/test';
  2  | import {
  3  |   enterTechCoursePracticeWorkspace,
  4  |   expectNoHorizontalOverflow,
  5  |   expectTouchTarget,
  6  |   gotoLearnerApp,
  7  | } from './support';
  8  | 
  9  | const VIEWPORTS = [
  10 |   { name: 'wide', width: 1440, height: 900 },
  11 |   { name: 'compact', width: 1024, height: 768 },
  12 |   { name: 'narrow', width: 375, height: 667 },
  13 | ];
  14 | 
  15 | for (const viewport of VIEWPORTS) {
  16 |   test.describe(`${viewport.name} viewport (${viewport.width}x${viewport.height})`, () => {
  17 |     test.use({ viewport: { width: viewport.width, height: viewport.height } });
  18 | 
  19 |     test('library and workspace fit without overflow and keep 44px targets', async ({ page }) => {
  20 |       await gotoLearnerApp(page);
  21 | 
  22 |       await expect(
  23 |         page.getByRole('heading', { name: 'Ваше обучение' }),
  24 |       ).toBeVisible();
  25 |       // Catalog listing shares the server with earlier suites (generation shelf + mentor jobs).
  26 |       // enterCourseWorkspace already waits 15s; the default 5s locator timeout loses the race
  27 |       // while "Loading course catalog" is still on screen.
  28 |       await expect(page.getByRole('status', { name: 'Loading course catalog' })).toHaveCount(0, {
  29 |         timeout: 15_000,
  30 |       });
  31 |       await expect(page.getByRole('heading', { name: 'Tech Course', exact: true })).toBeVisible();
  32 |       await expectNoHorizontalOverflow(page);
  33 | 
  34 |       const primaryAction = page
  35 |         .locator('article', { hasText: 'Tech Course' })
  36 |         .first()
  37 |         .getByRole('button', { name: /Start|Continue/ });
  38 |       await expectTouchTarget(primaryAction);
  39 | 
  40 |       await enterTechCoursePracticeWorkspace(page);
  41 |       await expectNoHorizontalOverflow(page);
  42 | 
  43 |       const lessonTab = page.getByRole('tab', { name: 'Theory' });
  44 |       const workspaceTab = page.getByRole('tab', { name: 'Code' });
  45 |       await expectTouchTarget(lessonTab);
  46 |       await expectTouchTarget(workspaceTab);
  47 | 
  48 |       await workspaceTab.click();
  49 |       await expect(page.locator('.cm-content')).toBeVisible();
  50 |       // Below 1200px the Support column (with the workspace actions) is only reachable through
  51 |       // the compact-view toggle; at >=1200px the toggle itself is hidden.
  52 |       const supportToggle = page.getByRole('button', { name: 'Support', exact: true });
  53 |       if (await supportToggle.isVisible()) {
  54 |         await supportToggle.click();
  55 |       }
  56 |       // Side-only Check work policy: before the backend authorizes completion the action is
  57 |       // Check work; Next unit replaces it once completion is authorized. Either way the primary
  58 |       // action must keep the 44px touch-target minimum.
  59 |       await expectTouchTarget(page.getByRole('button', { name: /Check work|Next unit/ }));
  60 |     });
  61 |   });
  62 | }
  63 | 
  64 | test('narrow viewport: trust modal fits within screen bounds and dismisses on Escape', async ({
  65 |   page,
  66 | }) => {
  67 |   await page.setViewportSize({ width: 375, height: 667 });
  68 |   await gotoLearnerApp(page);
  69 | 
  70 |   const startButton = page
  71 |     .locator('article', { hasText: 'Modal Fixture' })
  72 |     .first()
  73 |     .getByRole('button', { name: 'Start' });
> 74 |   await startButton.click();
     |                     ^ Error: locator.click: Test timeout of 300000ms exceeded.
  75 | 
  76 |   const dialog = page.getByRole('dialog', { name: 'Grant Host Trust' });
  77 |   await expect(dialog).toBeVisible({ timeout: 10_000 });
  78 | 
  79 |   const box = await dialog.boundingBox();
  80 |   expect(box).not.toBeNull();
  81 |   if (box !== null) {
  82 |     expect(box.x).toBeGreaterThanOrEqual(0);
  83 |     expect(box.y).toBeGreaterThanOrEqual(0);
  84 |     expect(box.x + box.width).toBeLessThanOrEqual(375);
  85 |     expect(box.y + box.height).toBeLessThanOrEqual(667);
  86 |   }
  87 | 
  88 |   await page.keyboard.press('Escape');
  89 |   await expect(dialog).toBeHidden();
  90 | });
```