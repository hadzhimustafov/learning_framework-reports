# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: editor-lf627.spec.ts >> Enter keeps the indentation of the line above, and the first save leaves no unsaved marker
- Location: e2e/editor-lf627.spec.ts:28:1

# Error details

```
Error: expect(received).toContain(expected) // indexOf

Expected substring: "
def outer(x):
    y = x
    return y"
Received string:    "from fixture.domain import total_price
def checkout(order):
    return undefined_name
def outer(x):
        y = x
        return y"
```

# Page snapshot

```yaml
- generic [ref=e3]:
  - banner [ref=e4]:
    - link "Learning Framework" [ref=e6] [cursor=pointer]:
      - /url: /app
      - text: LF
    - tablist "Chapter sections" [ref=e8]:
      - tab "Theory" [ref=e9] [cursor=pointer]
      - tab "Practice" [selected] [ref=e10] [cursor=pointer]
    - generic [ref=e11]:
      - generic [ref=e12]:
        - button "Read" [ref=e13] [cursor=pointer]:
          - generic [ref=e16]: ReadR
        - status [ref=e17]
      - generic [ref=e18]:
        - button "Learner menu" [ref=e19] [cursor=pointer]
        - generic [ref=e26]: "Local server: available"
        - status [ref=e27]
  - generic [ref=e29]:
    - navigation "Course contents" [ref=e31]:
      - generic [ref=e32]:
        - heading "Contents" [level=2] [ref=e33]
        - button "Show the contents" [ref=e34] [cursor=pointer]
    - main [ref=e38]:
      - generic [ref=e39]:
        - paragraph [ref=e41]:
          - link "Back to the course overview" [ref=e42] [cursor=pointer]:
            - /url: /app/courses/e1
        - paragraph [ref=e43]:
          - generic [ref=e44]: Practice
          - generic [ref=e45]: Awaiting a check
        - heading "Total price" [level=1] [ref=e48]
        - generic [ref=e49]:
          - term [ref=e50]: Chapter goal
          - definition [ref=e51]: Complete checkout.
      - tabpanel "Practice" [ref=e52]:
        - region [ref=e53]:
          - heading "Practice" [level=2] [ref=e54]
          - group [ref=e55]:
            - generic "Task" [ref=e56] [cursor=pointer]
            - paragraph [ref=e58]: Complete checkout.
          - region "Code editor" [ref=e60]:
            - generic [ref=e61]:
              - paragraph [ref=e62]:
                - link "Choose an editor in Settings to open this folder in it" [ref=e63] [cursor=pointer]:
                  - /url: /app/settings/editor
              - status
            - application [ref=e66]:
              - generic [ref=e70]:
                - generic [ref=e75]:
                  - generic [ref=e77]:
                    - tablist "Active View Switcher" [ref=e80]:
                      - tab "Explorer (Ctrl+Shift+E)" [expanded] [selected] [ref=e81] [cursor=pointer]:
                        - generic "Explorer (Ctrl+Shift+E)" [ref=e82]: 
                      - tab "Search (Ctrl+Shift+F)" [ref=e83] [cursor=pointer]:
                        - generic "Search (Ctrl+Shift+F)" [ref=e84]: 
                    - toolbar "Manage" [ref=e87]: 
                  - generic [ref=e88]:
                    - generic [ref=e89]:
                      - heading "workspace" [level=2] [ref=e91]
                      - toolbar "workspace actions" [ref=e95]:
                        - button "New File..." [ref=e96] [cursor=pointer]: 
                        - button "New Folder..." [ref=e97] [cursor=pointer]: 
                        - button "Refresh Explorer" [ref=e98] [cursor=pointer]: 
                        - button "Collapse Folders in Explorer" [ref=e99] [cursor=pointer]: 
                        - button "Views and More Actions..." [ref=e102] [cursor=pointer]: 
                      - generic:
                        - generic:
                          - generic:
                            - toolbar
                    - generic [ref=e109]:
                      - text:      
                      - tree "Files Explorer" [ref=e112]:
                        - generic [ref=e113]:
                          - treeitem "fixture" [level=1] [ref=e114] [cursor=pointer]:
                            - generic [ref=e115]:
                              - generic [ref=e116]: 
                              - generic "\\workspace\\fixture" [ref=e118]: fixture
                          - treeitem "src" [expanded] [level=1] [ref=e122] [cursor=pointer]:
                            - generic [ref=e123]:
                              - generic [ref=e124]: 
                              - generic "\\workspace\\src" [ref=e126]: src
                          - treeitem "app.py" [level=2] [selected] [ref=e130] [cursor=pointer]:
                            - generic "\\workspace\\src\\app.py" [ref=e133]:
                              - text: 
                              - generic [ref=e134]: app.py
                          - treeitem "tests" [expanded] [level=1] [ref=e137] [cursor=pointer]:
                            - generic [ref=e138]:
                              - generic [ref=e139]: 
                              - generic "\\workspace\\tests" [ref=e141]: tests
                          - treeitem "test_app.py" [level=2] [ref=e145] [cursor=pointer]:
                            - generic "\\workspace\\tests\\test_app.py" [ref=e148]:
                              - text: 
                              - generic [ref=e149]: test_app.py
                          - treeitem "README.md" [level=1] [ref=e152] [cursor=pointer]:
                            - generic "\\workspace\\README.md" [ref=e155]:
                              - text: 
                              - generic [ref=e156]: README.md
                  - main [ref=e166]:
                    - generic [ref=e174]:
                      - generic [ref=e175] [cursor=pointer]:
                        - generic [ref=e176]:
                          - tablist [ref=e177]:
                            - tab "app.py" [selected] [ref=e178]:
                              - generic "\\workspace\\src\\app.py" [ref=e179]:
                                - text: 
                                - generic [ref=e180]: app.py
                              - toolbar "Tab actions" [ref=e184]:
                                - button "Close (Ctrl+F4)" [ref=e185]: 
                            - tab "test_app.py" [ref=e186]:
                              - generic "\\workspace\\tests\\test_app.py" [ref=e187]:
                                - text: 
                                - generic [ref=e188]: test_app.py
                              - toolbar "Tab actions" [ref=e192]:
                                - button "Close (Ctrl+F4)" [ref=e193]: 
                            - tab "README.md" [ref=e194]:
                              - generic "\\workspace\\README.md" [ref=e195]:
                                - text: 
                                - generic [ref=e196]: README.md
                              - toolbar "Tab actions" [ref=e200]:
                                - button "Close (Ctrl+F4)" [ref=e201]: 
                          - toolbar "Editor actions" [ref=e205]:
                            - button "Tests without submitting — nothing is saved" [ref=e206]: 
                            - button "Split Editor Right (Ctrl+\\) [Alt] Split Editor Down" [ref=e207]: 
                            - button "More Actions..." [ref=e210]: 
                        - list [ref=e213]:
                          - listitem [ref=e214]:
                            - generic "\\workspace\\src" [ref=e215]: src
                            - generic [ref=e218]: 
                          - listitem [ref=e219]:
                            - generic "\\workspace\\src\\app.py" [ref=e220]:
                              - text: 
                              - generic [ref=e221]: app.py
                            - text: 
                      - generic "app.py" [ref=e224]:
                        - code [ref=e225]:
                          - generic [ref=e226]:
                            - textbox "app.py" [active]
                            - textbox [ref=e227]
                            - generic [ref=e230]:
                              - generic [ref=e231]: "1"
                              - generic [ref=e233]: "2"
                              - generic [ref=e235]: "3"
                              - generic [ref=e237]:
                                - generic [ref=e238] [cursor=pointer]: 
                                - generic [ref=e239]: "4"
                              - generic [ref=e240]: "5"
                              - generic [ref=e242]: "6"
                              - generic [ref=e244]: "7"
                              - generic [ref=e246]:
                                - generic [ref=e247] [cursor=pointer]: 
                                - generic [ref=e248]: "8"
                              - generic [ref=e249]: "9"
                              - generic [ref=e251]: "10"
                            - generic [ref=e272]:
                              - generic [ref=e273]: from fixture.domain import total_price
                              - generic [ref=e277]: "def checkout(order):"
                              - generic [ref=e279]: return undefined_name
                              - generic [ref=e283]: "def outer(x):"
                              - generic [ref=e285]: y = x
                              - generic [ref=e287]: return y
                - status [ref=e294]:
                  - generic [ref=e296]:
                    - 'generic "Layout: US" [ref=e297]':
                      - 'button "Layout: US" [ref=e298] [cursor=pointer]'
                    - generic "Python" [ref=e299]:
                      - button "Python" [ref=e300] [cursor=pointer]
                    - generic "LF" [ref=e301]:
                      - button "LF" [ref=e302] [cursor=pointer]
                    - generic "UTF-8" [ref=e303]:
                      - button "UTF-8" [ref=e304] [cursor=pointer]
                    - 'generic "Spaces: 4" [ref=e305]':
                      - 'button "Spaces: 4" [ref=e306] [cursor=pointer]'
                    - generic "Ln 10, Col 17" [ref=e307]:
                      - button "Ln 10, Col 17" [ref=e308] [cursor=pointer]
              - generic:
                - alert
                - alert
    - complementary "Mentor panel" [ref=e310]:
      - generic [ref=e311]:
        - tablist "Support panel sections" [ref=e312]:
          - tab "Criteria" [selected] [ref=e313] [cursor=pointer]
          - tab "Mentor" [ref=e314] [cursor=pointer]
        - button "Collapse the Mentor panel" [expanded] [ref=e315] [cursor=pointer]
      - slider "Mentor panel width" [ref=e319]
      - text: +
      - tabpanel "Criteria" [ref=e320]:
        - region [ref=e321]:
          - heading "Criteria" [level=2] [ref=e322]
          - list [ref=e323]:
            - listitem [ref=e324]:
              - generic [ref=e325]:
                - generic [ref=e326]: R1
                - paragraph [ref=e327]: The tests pass
              - generic [ref=e328]: Awaiting a check
          - paragraph [ref=e331]: "Not saved: src/app.py"
          - button "Submit for a check" [ref=e332] [cursor=pointer]
```

# Test source

```ts
  1  | import { expect, test } from '@playwright/test';
  2  | import type { Page } from '@playwright/test';
  3  | import { FakeLanguageServer, STARTER, activeEditorText, navigateInApp, openUnitWithEditor, practiceFolder, stubUnit } from './editorSupport';
  4  | import type { FakeWorkspace } from './editorSupport';
  5  | 
  6  | /** LF-627 (R17-31/32): Enter auto-indents (editor default), the first save sticks, no «Leave site?» when saved, one chapter's files at a time. */
  7  | test.use({ viewport: { width: 1440, height: 900 }, locale: 'en-US' });
  8  | 
  9  | let workspace: FakeWorkspace;
  10 | 
  11 | test.beforeEach(async ({ page }) => {
  12 |   workspace = practiceFolder();
  13 |   await stubUnit(page);
  14 |   await workspace.install(page);
  15 |   await new FakeLanguageServer().install(page);
  16 | });
  17 | 
  18 | const editorArea = (page: Page) => page.locator('.editor-group-container.active .monaco-editor .view-lines').first();
  19 | /** Whether leaving now would raise the browser's «Leave site?» prompt (a cancelable `beforeunload` that a handler holds back). */
  20 | const asksOnLeave = (page: Page): Promise<boolean> =>
  21 |   page.evaluate(() => {
  22 |     const event = new Event('beforeunload', { cancelable: true });
  23 |     window.dispatchEvent(event);
  24 |     return event.defaultPrevented || (event as BeforeUnloadEvent).returnValue === false;
  25 |   });
  26 | const dirtyTabs = (page: Page) => page.locator('.tabs-container .tab.dirty');
  27 | 
  28 | test('Enter keeps the indentation of the line above, and the first save leaves no unsaved marker', async ({ page }) => {
  29 |   await openUnitWithEditor(page);
  30 |   await expect.poll(() => activeEditorText(page), { timeout: 30_000 }).toContain('def checkout');
  31 |   await editorArea(page).click();
  32 |   await page.keyboard.press('Control+End');
  33 |   // The editor's own auto-indent stays on (Owner 2026-11-03): Enter keeps the line's indentation, so the
  34 |   // next line needs no typed spaces.
  35 |   await page.keyboard.type('\n\ndef outer(x):\n    y = x\nreturn y');
  36 |   await expect.poll(() => activeEditorText(page)).toContain('def outer(x):');
> 37 |   expect(await activeEditorText(page)).toContain('\ndef outer(x):\n    y = x\n    return y');
     |                                        ^ Error: expect(received).toContain(expected) // indexOf
  38 |   await page.keyboard.press('Control+s');
  39 |   await expect.poll(() => workspace.text(STARTER)).toContain('def outer');
  40 |   await expect(dirtyTabs(page)).toHaveCount(0);
  41 | });
  42 | 
  43 | test('asks nothing on leaving when every file is saved', async ({ page }) => {
  44 |   const dialogs: string[] = [];
  45 |   page.on('dialog', (dialog) => {
  46 |     dialogs.push(dialog.type());
  47 |     void dialog.dismiss();
  48 |   });
  49 |   await openUnitWithEditor(page);
  50 |   await expect.poll(() => activeEditorText(page), { timeout: 30_000 }).toContain('def checkout');
  51 |   await editorArea(page).click();
  52 |   await page.keyboard.press('Control+End');
  53 |   await page.keyboard.type('\n# note');
  54 |   await page.keyboard.press('Control+s');
  55 |   await expect.poll(() => workspace.text(STARTER)).toContain('# note');
  56 |   await expect(dirtyTabs(page)).toHaveCount(0);
  57 | 
  58 |   expect(await asksOnLeave(page)).toBe(false);
  59 |   await page.close({ runBeforeUnload: true });
  60 |   expect(dialogs).toEqual([]);
  61 | });
  62 | 
  63 | test('moving to another chapter shows only that chapter\'s files', async ({ page }) => {
  64 |   await stubUnit(page, ['chapter2/main.py', 'chapter2/test_main.py', 'chapter2/README.md'], 'u2');
  65 |   workspace.externalWrite('chapter2/main.py', 'print(2)\n');
  66 |   workspace.externalWrite('chapter2/test_main.py', 'x = 2\n');
  67 |   workspace.externalWrite('chapter2/README.md', '# two\n');
  68 |   await openUnitWithEditor(page);
  69 |   await expect.poll(() => activeEditorText(page), { timeout: 30_000 }).toContain('def checkout');
  70 |   await navigateInApp(page, '/app/courses/e1/units/u2');
  71 |   await page.locator('#unit-tab-practice').click();
  72 |   await expect(page.locator('.tabs-container .tab', { hasText: 'main.py' }).first()).toBeVisible({ timeout: 30_000 });
  73 |   await expect(page.locator('.tabs-container .tab', { hasText: 'app.py' })).toHaveCount(0);
  74 |   await expect(page.getByText('Could not open', { exact: false })).toHaveCount(0);
  75 | });
  76 | 
```