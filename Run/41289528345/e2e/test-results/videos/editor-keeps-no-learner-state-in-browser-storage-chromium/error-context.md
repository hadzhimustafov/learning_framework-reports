# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: editor.spec.ts >> keeps no learner state in browser storage
- Location: e2e/editor.spec.ts:164:1

# Error details

```
Error: expect(received).toEqual(expected) // deep equality

- Expected  - 3
+ Received  + 1

  Object {
    "databases": Array [
      "vscode-web-state-db-global-shared",
    ],
-   "keys": Array [
-     "monaco-parts-splash",
-   ],
+   "keys": Array [],
  }
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
                      - tree "Files Explorer" [active] [ref=e112]:
                        - generic [ref=e113]:
                          - treeitem "fixture" [level=1] [ref=e114] [cursor=pointer]:
                            - generic [ref=e115]:
                              - generic [ref=e116]: 
                              - generic "\\workspace\\fixture" [ref=e118]: fixture
                          - treeitem "src" [level=1] [selected] [ref=e122] [cursor=pointer]:
                            - generic [ref=e123]:
                              - generic [ref=e124]: 
                              - generic "\\workspace\\src" [ref=e126]: src
                          - treeitem "tests" [expanded] [level=1] [ref=e130] [cursor=pointer]:
                            - generic [ref=e131]:
                              - generic [ref=e132]: 
                              - generic "\\workspace\\tests" [ref=e134]: tests
                          - treeitem "test_app.py" [level=2] [ref=e138] [cursor=pointer]:
                            - generic "\\workspace\\tests\\test_app.py" [ref=e141]:
                              - text: 
                              - generic [ref=e142]: test_app.py
                          - treeitem "README.md" [level=1] [ref=e145] [cursor=pointer]:
                            - generic "\\workspace\\README.md" [ref=e148]:
                              - text: 
                              - generic [ref=e149]: README.md
                  - main [ref=e157]:
                    - generic [ref=e165]:
                      - generic [ref=e166] [cursor=pointer]:
                        - generic [ref=e167]:
                          - tablist [ref=e168]:
                            - tab "app.py" [selected] [ref=e169]:
                              - generic "\\workspace\\src\\app.py" [ref=e170]:
                                - text: 
                                - generic [ref=e171]: app.py
                              - toolbar "Tab actions" [ref=e175]:
                                - button "Close (Ctrl+F4)" [ref=e176]: 
                            - tab "test_app.py" [ref=e177]:
                              - generic "\\workspace\\tests\\test_app.py" [ref=e178]:
                                - text: 
                                - generic [ref=e179]: test_app.py
                              - toolbar "Tab actions" [ref=e183]:
                                - button "Close (Ctrl+F4)" [ref=e184]: 
                            - tab "README.md" [ref=e185]:
                              - generic "\\workspace\\README.md" [ref=e186]:
                                - text: 
                                - generic [ref=e187]: README.md
                              - toolbar "Tab actions" [ref=e191]:
                                - button "Close (Ctrl+F4)" [ref=e192]: 
                          - toolbar "Editor actions" [ref=e196]:
                            - button "Split Editor Right (Ctrl+\\) [Alt] Split Editor Down" [ref=e197]: 
                            - button "More Actions..." [ref=e200]: 
                        - list [ref=e203]:
                          - listitem [ref=e204]:
                            - generic "\\workspace\\src" [ref=e205]: src
                            - generic [ref=e208]: 
                          - listitem [ref=e209]:
                            - generic "\\workspace\\src\\app.py" [ref=e210]:
                              - text: 
                              - generic [ref=e211]: app.py
                            - text: 
                      - generic "app.py" [ref=e214]:
                        - code [ref=e215]:
                          - generic [ref=e216]:
                            - textbox "app.py"
                            - textbox [ref=e217]
                            - generic [ref=e220]:
                              - generic [ref=e221]: "1"
                              - generic [ref=e224]: "2"
                              - generic [ref=e226]: "3"
                              - generic [ref=e228]:
                                - generic [ref=e229] [cursor=pointer]: 
                                - generic [ref=e230]: "4"
                              - generic [ref=e231]: "5"
                              - generic [ref=e233]: "6"
                            - generic [ref=e245]:
                              - generic [ref=e246]: from fixture.domain import total_price
                              - generic [ref=e250]: "def checkout(order):"
                              - generic [ref=e252]: return undefined_name
                - status [ref=e257]:
                  - generic [ref=e259]:
                    - 'generic "Layout: US" [ref=e260]':
                      - 'button "Layout: US" [ref=e261] [cursor=pointer]'
                    - generic "Python" [ref=e262]:
                      - button "Python" [ref=e263] [cursor=pointer]
                    - generic "LF" [ref=e264]:
                      - button "LF" [ref=e265] [cursor=pointer]
                    - generic "UTF-8" [ref=e266]:
                      - button "UTF-8" [ref=e267] [cursor=pointer]
                    - 'generic "Spaces: 4" [ref=e268]':
                      - 'button "Spaces: 4" [ref=e269] [cursor=pointer]'
                    - generic "Ln 1, Col 1" [ref=e270]:
                      - button "Ln 1, Col 1" [ref=e271] [cursor=pointer]
              - generic:
                - alert
                - alert
              - text:         
    - complementary "Mentor panel" [ref=e273]:
      - generic [ref=e274]:
        - tablist "Support panel sections" [ref=e275]:
          - tab "Criteria" [selected] [ref=e276] [cursor=pointer]
          - tab "Mentor" [ref=e277] [cursor=pointer]
        - button "Collapse the Mentor panel" [expanded] [ref=e278] [cursor=pointer]
      - slider "Mentor panel width" [ref=e282]
      - text: +
      - tabpanel "Criteria" [ref=e283]:
        - region [ref=e284]:
          - heading "Criteria" [level=2] [ref=e285]
          - list [ref=e286]:
            - listitem [ref=e287]:
              - generic [ref=e288]:
                - generic [ref=e289]: R1
                - paragraph [ref=e290]: The tests pass
              - generic [ref=e291]: Awaiting a check
          - paragraph [ref=e294]: All files saved
          - button "Submit for a check" [ref=e295] [cursor=pointer]
```

# Test source

```ts
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
  97  |   await expect(page.locator('.quick-input-list .monaco-list-row', { hasText: 'domain.py' }).first()).toBeVisible({ timeout: 15_000 });
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
> 175 |   expect(stored).toEqual({ databases: ['vscode-web-state-db-global-shared'], keys: ['monaco-parts-splash'] });
      |                  ^ Error: expect(received).toEqual(expected) // deep equality
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
  198 |   await expect(page.locator('.search-view .monaco-list-row').first()).toBeVisible({ timeout: 20_000 });
  199 | 
  200 |   expect(await watched.violations()).toEqual([]);
  201 |   expect(watched.consoleErrors.filter((line) => /Content Security Policy|Refused to/.test(line))).toEqual([]);
  202 | });
  203 | 
```