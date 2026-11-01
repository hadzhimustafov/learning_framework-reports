# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: editor.spec.ts >> keeps a course fixture read-only
- Location: e2e/editor.spec.ts:92:1

# Error details

```
Error: expect(locator).toBeVisible() failed

Locator: locator('.quick-input-list .monaco-list-row').filter({ hasText: 'domain.py' }).first()
Expected: visible
Timeout: 15000ms
Error: element(s) not found

Call log:
  - Expect "toBeVisible" with timeout 15000ms
  - waiting for locator('.quick-input-list .monaco-list-row').filter({ hasText: 'domain.py' }).first()

```

```yaml
- banner:
  - link "Learning Framework":
    - /url: /app
    - text: LF
  - tablist "Chapter sections":
    - tab "Theory"
    - tab "Practice" [selected]
  - button "Read"
  - status
  - button "Learner menu"
  - text: "Local server: available"
  - status
- navigation "Course contents":
  - heading "Contents" [level=2]
  - button "Show the contents"
- main "Total price":
  - paragraph:
    - link "Back to the course overview":
      - /url: /app/courses/e1
  - paragraph: Practice Awaiting a check
  - heading "Total price" [level=1]
  - term: Chapter goal
  - definition: Complete checkout.
  - tabpanel "Practice":
    - region "Practice":
      - heading "Practice" [level=2]
      - group:
        - text: Task
        - paragraph: Complete checkout.
      - region "Code editor":
        - paragraph:
          - link "Choose an editor in Settings to open this folder in it":
            - /url: /app/settings/editor
        - status
        - application:
          - tablist "Active View Switcher":
            - tab "Explorer (Ctrl+Shift+E)" [expanded] [selected]: 
            - tab "Search (Ctrl+Shift+F)": 
          - toolbar "Manage"
          - heading "workspace" [level=2]
          - toolbar "workspace actions":
            - button "New File...": 
            - button "New Folder...": 
            - button "Refresh Explorer": 
            - button "Collapse Folders in Explorer": 
            - button "Views and More Actions...": 
          - toolbar
          - tree "Files Explorer":
            - treeitem "fixture" [level=1]:  fixture
            - treeitem "src" [expanded] [level=1]:  src
            - treeitem "app.py" [level=2] [selected]:  app.py
            - treeitem "tests" [expanded] [level=1]:  tests
            - treeitem "test_app.py" [level=2]:  test_app.py
            - treeitem "README.md" [level=1]:  README.md
          - main:
            - tablist:
              - tab "app.py" [selected]:
                - text:  app.py
                - toolbar "Tab actions":
                  - button "Close (Ctrl+F4)": 
              - tab "test_app.py":
                - text:  test_app.py
                - toolbar "Tab actions":
                  - button "Close (Ctrl+F4)": 
              - tab "README.md":
                - text:  README.md
                - toolbar "Tab actions":
                  - button "Close (Ctrl+F4)": 
            - toolbar "Editor actions":
              - button "Split Editor Right (Ctrl+\\) [Alt] Split Editor Down": 
              - button "More Actions...": 
            - list:
              - listitem: src 
              - listitem:  app.py
            - code:
              - textbox "app.py"
          - status:
            - 'button "Layout: US"'
            - button "Python"
            - button "LF"
            - button "UTF-8"
            - 'button "Spaces: 4"'
            - button "Ln 1, Col 6"
          - alert
          - alert
- complementary "Mentor panel":
  - tablist "Support panel sections":
    - tab "Criteria" [selected]
    - tab "Mentor"
  - button "Collapse the Mentor panel" [expanded]
  - slider "Mentor panel width"
  - tabpanel "Criteria":
    - region "Criteria":
      - heading "Criteria" [level=2]
      - list:
        - listitem:
          - text: R1
          - paragraph: The tests pass
          - text: Awaiting a check
      - paragraph: All files saved
      - button "Submit for a check"
```

# Test source

```ts
  1   | import { expect, test } from '@playwright/test';
  2   | import {
  3   |   FakeLanguageServer,
  4   |   STARTER,
  5   |   activeEditorText,
  6   |   openUnitWithEditor,
  7   |   practiceFolder,
  8   |   stubUnit,
  9   |   watchViolations,
  10  | } from './editorSupport';
  11  | import type { FakeWorkspace } from './editorSupport';
  12  | 
  13  | /**
  14  |  * M38-015 (LF-388): the VS Code workbench in the unit's practice slot, against the built app and its real
  15  |  * Content-Security-Policy. The working folder and the language server are answered by the page.
  16  |  */
  17  | test.use({ viewport: { width: 1440, height: 900 }, locale: 'en-US' });
  18  | 
  19  | let workspace: FakeWorkspace;
  20  | 
  21  | test.beforeEach(async ({ page }) => {
  22  |   workspace = practiceFolder();
  23  |   await stubUnit(page);
  24  |   await workspace.install(page);
  25  |   await new FakeLanguageServer().install(page);
  26  | });
  27  | 
  28  | const rows = (page: import('@playwright/test').Page) => page.locator('.explorer-folders-view .monaco-list-row');
  29  | 
  30  | test('mounts the workbench in the practice slot and opens the starter file', async ({ page }) => {
  31  |   await openUnitWithEditor(page);
  32  | 
  33  |   const slot = page.locator('[data-slot="practice-editor"]');
  34  |   const box = await slot.boundingBox();
  35  |   expect(box?.width ?? 0).toBeGreaterThan(500);
  36  |   await expect(page.locator('.tabs-container .tab', { hasText: 'app.py' }).first()).toBeVisible({ timeout: 30_000 });
  37  |   await expect.poll(() => activeEditorText(page), { timeout: 30_000 }).toContain('def checkout');
  38  | });
  39  | 
  40  | test('explorer, tabs, quick open and search work over the working folder', async ({ page }) => {
  41  |   await openUnitWithEditor(page);
  42  | 
  43  |   // The three starter files are open already (their folders are expanded); `fixture` is not one of them.
  44  |   await rows(page).filter({ hasText: /^fixture$/ }).first().click();
  45  |   await rows(page).filter({ hasText: 'domain.py' }).first().click();
  46  |   await expect(page.locator('.tabs-container .tab', { hasText: 'domain.py' }).first()).toBeVisible();
  47  | 
  48  |   await page.keyboard.press('Control+p');
  49  |   await page.locator('.quick-input-widget input').waitFor();
  50  |   await page.keyboard.type('test_app');
  51  |   await expect(page.locator('.quick-input-list .monaco-list-row', { hasText: 'test_app.py' }).first()).toBeVisible({ timeout: 15_000 });
  52  |   await page.keyboard.press('Enter');
  53  |   await expect(page.locator('.tabs-container .tab', { hasText: 'test_app.py' }).first()).toBeVisible();
  54  | 
  55  |   await page.keyboard.press('Control+Shift+f');
  56  |   await page.locator('.search-widget textarea, .search-widget input').first().waitFor();
  57  |   await page.keyboard.type('total_price');
  58  |   await page.keyboard.press('Enter');
  59  |   await expect(page.locator('.search-view .monaco-list-row', { hasText: 'app.py' }).first()).toBeVisible({ timeout: 20_000 });
  60  | });
  61  | 
  62  | test('saves an edit of the starter file to the working folder, with Ctrl+S and when focus leaves the editor', async ({ page }) => {
  63  |   await openUnitWithEditor(page);
  64  |   await expect.poll(() => activeEditorText(page), { timeout: 30_000 }).toContain('def checkout');
  65  | 
  66  |   await page.locator('.editor-group-container.active .monaco-editor .view-lines').first().click();
  67  |   await page.keyboard.press('Control+End');
  68  |   await page.keyboard.type('\n# saved with ctrl+s');
  69  |   await page.keyboard.press('Control+s');
  70  |   await expect.poll(() => workspace.text(STARTER)).toContain('# saved with ctrl+s');
  71  | 
  72  |   await page.keyboard.type('\n# saved when focus left');
  73  |   await rows(page).filter({ hasText: 'README.md' }).first().click();
  74  |   await expect.poll(() => workspace.text(STARTER), { timeout: 10_000 }).toContain('# saved when focus left');
  75  | });
  76  | 
  77  | test('shows a change made outside the editor, in the open file and in the explorer', async ({ page }) => {
  78  |   await openUnitWithEditor(page);
  79  |   await expect.poll(() => activeEditorText(page), { timeout: 30_000 }).toContain('def checkout');
  80  | 
  81  |   workspace.externalWrite(STARTER, 'x = 1\n# changed externally\n');
  82  |   await expect.poll(() => activeEditorText(page), { timeout: 20_000 }).toContain('changed externally');
  83  | 
  84  |   // Opening the starter file already revealed `src`: a click on an expanded folder would collapse it.
  85  |   const src = rows(page).filter({ hasText: /^src$/ }).first();
  86  |   if ((await src.getAttribute('aria-expanded')) !== 'true') await src.click();
  87  |   await expect(src).toHaveAttribute('aria-expanded', 'true');
  88  |   workspace.externalWrite('src/created_elsewhere.py', 'y = 2\n');
  89  |   await expect(rows(page).filter({ hasText: 'created_elsewhere.py' }).first()).toBeVisible({ timeout: 20_000 });
  90  | });
  91  | 
  92  | test('keeps a course fixture read-only', async ({ page }) => {
  93  |   await openUnitWithEditor(page);
  94  |   await page.keyboard.press('Control+p');
  95  |   await page.locator('.quick-input-widget input').waitFor();
  96  |   await page.keyboard.type('domain.py');
> 97  |   await expect(page.locator('.quick-input-list .monaco-list-row', { hasText: 'domain.py' }).first()).toBeVisible({ timeout: 15_000 });
      |                                                                                                      ^ Error: expect(locator).toBeVisible() failed
  98  |   await page.keyboard.press('Enter');
  99  |   await page.locator('.editor-group-container.active .monaco-editor .view-lines').first().click();
  100 |   await page.keyboard.press('Control+End');
  101 |   await page.keyboard.type('# nope');
  102 |   await page.keyboard.press('Control+s');
  103 |   await page.waitForTimeout(2_000);
  104 | 
  105 |   expect(workspace.writes.filter((write) => write.path.startsWith('fixture/'))).toEqual([]);
  106 |   expect(workspace.text('fixture/domain.py')).not.toContain('# nope');
  107 |   const textarea = page.locator('.editor-group-container.active .monaco-editor textarea').first();
  108 |   expect((await textarea.getAttribute('readonly')) !== null || (await textarea.getAttribute('aria-readonly')) === 'true').toBe(true);
  109 | });
  110 | 
  111 | test('follows the colour scheme without a reload', async ({ page }) => {
  112 |   await page.addInitScript(() => {
  113 |     (window as unknown as { __marker: string }).__marker = 'same-document';
  114 |   });
  115 |   await openUnitWithEditor(page);
  116 |   const background = () => page.evaluate(() => getComputedStyle(document.querySelector('.monaco-workbench .part.sidebar') as Element).backgroundColor);
  117 |   await expect.poll(background).not.toBe('');
  118 |   const initial = await background();
  119 |   await page.evaluate(() => {
  120 |     document.documentElement.dataset.theme = 'dark';
  121 |   });
  122 |   // The scheme is applied asynchronously: wait for the workbench to leave the initial (light) colours first.
  123 |   await expect.poll(background, { timeout: 15_000 }).not.toBe(initial);
  124 |   const dark = await background();
  125 | 
  126 |   await page.evaluate(() => {
  127 |     document.documentElement.dataset.theme = 'light';
  128 |   });
  129 |   await expect.poll(background, { timeout: 15_000 }).not.toBe(dark);
  130 |   expect(await page.evaluate(() => (window as unknown as { __marker: string }).__marker)).toBe('same-document');
  131 | });
  132 | 
  133 | test('offers no terminal, extensions, accounts, sync, upload or download', async ({ page }) => {
  134 |   await openUnitWithEditor(page);
  135 | 
  136 |   // The activity bar keeps the explorer and search; the gear (Manage) and Accounts entries are not shown.
  137 |   await expect(page.locator('.monaco-workbench .part.activitybar .action-item:has(.codicon-accounts-view-bar-icon)')).toBeHidden();
  138 |   await expect(page.locator('.monaco-workbench .part.activitybar .action-item:has(.codicon-settings-view-bar-icon)')).toBeHidden();
  139 | 
  140 |   await page.keyboard.press('Control+`');
  141 |   await page.keyboard.press('Control+Shift+x');
  142 |   await page.waitForTimeout(500);
  143 |   await expect(page.locator('.terminal-outer-container, .xterm')).toHaveCount(0);
  144 |   await expect(page.locator('.extensions-viewlet')).toHaveCount(0);
  145 | 
  146 |   // The explorer's folder menu offers no import or export of files.
  147 |   await rows(page).filter({ hasText: /^src$/ }).first().click({ button: 'right' });
  148 |   const menu = page.locator('.monaco-menu');
  149 |   await expect(menu).toBeVisible();
  150 |   const items = (await menu.locator('.action-label').allInnerTexts()).join('|');
  151 |   expect(items).toMatch(/New File/i);
  152 |   expect(items).not.toMatch(/download|upload/i);
  153 |   await page.keyboard.press('Escape');
  154 | 
  155 |   // The command palette knows nothing of settings sync or the terminal.
  156 |   await page.keyboard.press('F1');
  157 |   await page.locator('.quick-input-widget input').waitFor();
  158 |   await page.keyboard.type('sync');
  159 |   await page.waitForTimeout(500);
  160 |   const commands = (await page.locator('.quick-input-list .monaco-list-row').allInnerTexts()).join('|');
  161 |   expect(commands).not.toMatch(/settings sync|turn on|backup and sync/i);
  162 | });
  163 | 
  164 | test('keeps no learner state in browser storage', async ({ page }) => {
  165 |   await openUnitWithEditor(page);
  166 |   await rows(page).filter({ hasText: /^src$/ }).first().click();
  167 |   await page.keyboard.press('Control+p');
  168 |   await page.keyboard.press('Escape');
  169 | 
  170 |   // Two chrome leftovers are known: the library's empty "new storage" marker database and the splash colours.
  171 |   const stored = await page.evaluate(async () => ({
  172 |     databases: (await indexedDB.databases()).map((database) => database.name),
  173 |     keys: Object.keys(localStorage).filter((key) => /vscode|monaco|workbench/i.test(key)),
  174 |   }));
  175 |   expect(stored).toEqual({ databases: ['vscode-web-state-db-global-shared'], keys: ['monaco-parts-splash'] });
  176 |   const contents = await page.evaluate(async () => {
  177 |     const request = indexedDB.open('vscode-web-state-db-global-shared');
  178 |     const db = await new Promise<IDBDatabase>((resolve) => (request.onsuccess = () => resolve(request.result)));
  179 |     const store = db.transaction('ItemTable').objectStore('ItemTable').getAllKeys();
  180 |     return new Promise<IDBValidKey[]>((resolve) => (store.onsuccess = () => resolve(store.result)));
  181 |   });
  182 |   expect(contents).toEqual(['__$__isNewStorageMarker']);
  183 | });
  184 | 
  185 | test('runs under the page policy without violations', async ({ page }) => {
  186 |   const watched = await watchViolations(page);
  187 |   const response = await page.request.get('/app/');
  188 |   const policy = response.headers()['content-security-policy'] ?? '';
  189 |   expect(policy).toContain("script-src 'self' 'wasm-unsafe-eval'");
  190 |   expect(policy).toContain("worker-src 'self' blob:");
  191 |   expect(policy).not.toContain("'unsafe-eval'");
  192 |   expect(policy).not.toMatch(/script-src[^;]*'unsafe-inline'/);
  193 | 
  194 |   await openUnitWithEditor(page);
  195 |   await page.keyboard.press('Control+Shift+f');
  196 |   await page.keyboard.type('checkout');
  197 |   await page.keyboard.press('Enter');
```