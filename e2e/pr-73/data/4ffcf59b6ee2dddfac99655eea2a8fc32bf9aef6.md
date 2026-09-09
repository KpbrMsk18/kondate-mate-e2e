# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: app-flows.e2e.ts >> E2E-WEB-05: 家庭の肉と魚の比率を保存できる
- Location: e2e/app-flows.e2e.ts:210:5

# Error details

```
Error: expect(received).toBe(expected) // Object.is equality

Expected: 70
Received: 50
```

# Page snapshot

```yaml
- generic [active] [ref=e1]:
  - banner [ref=e2]:
    - link "こんだてめいと" [ref=e3] [cursor=pointer]:
      - /url: /
      - img [ref=e5]
      - heading "こんだてめいと" [level=1] [ref=e7]
    - generic [ref=e8]:
      - navigation [ref=e9]:
        - link "献立" [ref=e10] [cursor=pointer]:
          - /url: /menu
        - link "買い物" [ref=e11] [cursor=pointer]:
          - /url: /shopping
        - link "レシピ" [ref=e12] [cursor=pointer]:
          - /url: /recipes
        - link "Pantry" [ref=e13] [cursor=pointer]:
          - /url: /pantry
        - link "ルール" [ref=e14] [cursor=pointer]:
          - /url: /rules
      - 'button "現在: システム設定に合わせる。クリックでライトモードに切り替え" [ref=e15] [cursor=pointer]':
        - img [ref=e16]
      - button "Open user menu" [ref=e19] [cursor=pointer]:
        - img "'s logo" [ref=e22]
  - main [ref=e24]:
    - generic [ref=e25]:
      - heading "家庭のルール（曜日の固定メニュー）" [level=2] [ref=e27]
      - paragraph [ref=e28]: 「金曜はカレー」のような家庭のルールです。献立生成時に AI はこのルールを破れません。同じ曜日に登録すると上書きされます。
      - generic [ref=e29]:
        - heading "肉と魚の比率" [level=3] [ref=e30]
        - paragraph [ref=e31]: 献立を自動生成するときの目安です。大豆・卵やその他の主菜も候補に含まれます。
        - generic [ref=e32]:
          - strong [ref=e33]: 肉 70%
          - slider "肉と魚の比率" [ref=e34]: "70"
          - strong [ref=e35]: 魚 30%
          - button "比率を保存" [ref=e36] [cursor=pointer]
      - generic [ref=e37]:
        - heading "曜日の固定メニュー" [level=3] [ref=e38]
        - generic [ref=e39]:
          - img [ref=e40]
          - paragraph [ref=e42]: ルールはまだありません。
        - generic [ref=e43]:
          - combobox "曜日" [ref=e44] [cursor=pointer]:
            - option "月曜"
            - option "火曜"
            - option "水曜"
            - option "木曜"
            - option "金曜" [selected]
            - option "土曜"
            - option "日曜"
          - combobox "レシピ" [ref=e45] [cursor=pointer]:
            - option "レシピを選ぶ…" [selected]
          - button "ルールを登録" [disabled] [ref=e46]
        - paragraph [ref=e47]:
          - text: 先に
          - link "レシピを登録" [ref=e48] [cursor=pointer]:
            - /url: /recipes/new
          - text: してください。
  - generic [ref=e50]:
    - img [ref=e51]
    - text: 比率を保存しました
  - button "Open Next.js Dev Tools" [ref=e58] [cursor=pointer]:
    - img [ref=e59]
  - alert [ref=e62]
```

# Test source

```ts
  148 |   await prepareAuthenticatedPage(page);
  149 |   const list = {
  150 |     weekStart: "2026-07-20",
  151 |     items: [
  152 |       {
  153 |         ingredientId: "onion",
  154 |         name: "玉ねぎ",
  155 |         amount: "1個",
  156 |         purchased: false,
  157 |       },
  158 |     ],
  159 |   };
  160 | 
  161 |   await page.route("http://localhost:3001/**", async (route) => {
  162 |     const request = route.request();
  163 |     const path = new URL(request.url()).pathname;
  164 |     if (request.method() === "GET") return json(route, list);
  165 |     if (path.endsWith("/items") && request.method() === "POST") {
  166 |       const input = request.postDataJSON();
  167 |       list.items.push({ ingredientId: "carrot", ...input, purchased: false });
  168 |       await route.fulfill({ status: 204 });
  169 |       return;
  170 |     }
  171 |     const item = list.items.find(({ ingredientId }) =>
  172 |       path.includes(`/items/${ingredientId}`),
  173 |     );
  174 |     if (item === undefined) return route.fulfill({ status: 404 });
  175 |     if (path.endsWith("/unpurchase")) item.purchased = false;
  176 |     else if (path.endsWith("/purchase")) item.purchased = true;
  177 |     else if (request.method() === "PUT")
  178 |       Object.assign(item, request.postDataJSON());
  179 |     else if (request.method() === "DELETE")
  180 |       list.items.splice(list.items.indexOf(item), 1);
  181 |     await route.fulfill({ status: 204 });
  182 |   });
  183 | 
  184 |   await page.goto("/shopping");
  185 |   await page.getByLabel("追加する食材名").fill("にんじん");
  186 |   await page.getByLabel("追加する食材の分量").fill("2本");
  187 |   await page.getByRole("button", { name: "追加", exact: true }).click();
  188 |   await expect(page.getByText("にんじん")).toBeVisible();
  189 | 
  190 |   const carrot = page.locator(".shopping-item").filter({ hasText: "にんじん" });
  191 |   await carrot.getByRole("button", { name: "編集" }).click();
  192 |   await page.getByLabel("にんじんの分量").fill("3本");
  193 |   await page.getByRole("button", { name: "保存" }).click();
  194 |   const updatedCarrot = page
  195 |     .locator(".shopping-item")
  196 |     .filter({ hasText: "にんじん" });
  197 |   await expect(updatedCarrot.getByText("3本")).toBeVisible();
  198 | 
  199 |   await updatedCarrot.getByRole("checkbox").click();
  200 |   await expect(updatedCarrot.getByRole("checkbox")).toBeChecked();
  201 |   await expect(
  202 |     updatedCarrot.getByRole("button", { name: "編集" }),
  203 |   ).toBeDisabled();
  204 |   await updatedCarrot.getByRole("checkbox").click();
  205 |   await expect(updatedCarrot.getByRole("checkbox")).not.toBeChecked();
  206 |   await updatedCarrot.getByRole("button", { name: "削除" }).click();
  207 |   await expect(page.getByText("にんじん")).toHaveCount(0);
  208 | });
  209 | 
  210 | test("E2E-WEB-05: 家庭の肉と魚の比率を保存できる", async ({ page }) => {
  211 |   await prepareAuthenticatedPage(page);
  212 |   let meatRatio = 50;
  213 | 
  214 |   await page.route("http://localhost:3001/**", async (route) => {
  215 |     const request = route.request();
  216 |     const path = new URL(request.url()).pathname;
  217 | 
  218 |     if (path === "/household/fixed-menu-rules") return json(route, []);
  219 |     if (path === "/recipes") return json(route, []);
  220 |     if (path === "/household/menu-preference") {
  221 |       if (request.method() === "PUT") {
  222 |         meatRatio = request.postDataJSON().meatRatio;
  223 |         await route.fulfill({ status: 204 });
  224 |         return;
  225 |       }
  226 |       await json(route, { meatRatio });
  227 |       return;
  228 |     }
  229 |     await route.fulfill({ status: 404 });
  230 |   });
  231 | 
  232 |   await page.goto("/rules");
  233 |   const ratio = page.getByLabel("肉と魚の比率");
  234 |   await expect(ratio).toHaveValue("50");
  235 |   await ratio.fill("70");
  236 |   // モバイル専用の表示用サマリー（.ratio-mobile-values、aria-hidden）にも
  237 |   // 同じ文言が複製されるため、実要素のクラスで一意に絞り込む。
  238 |   await expect(page.locator(".ratio-meat")).toHaveText("肉 70%");
  239 |   await expect(page.locator(".ratio-fish")).toHaveText("魚 30%");
  240 | 
  241 |   const saved = page.waitForRequest(
  242 |     (request) =>
  243 |       request.method() === "PUT" &&
  244 |       new URL(request.url()).pathname === "/household/menu-preference",
  245 |   );
  246 |   await page.getByRole("button", { name: "比率を保存" }).click();
  247 |   expect((await saved).postDataJSON()).toEqual({ meatRatio: 70 });
> 248 |   expect(meatRatio).toBe(70);
      |                     ^ Error: expect(received).toBe(expected) // Object.is equality
  249 | 
  250 |   // 保存できたことを画面上でも確認できる。
  251 |   await expect(page.getByTestId("toast").last()).toHaveText(
  252 |     "比率を保存しました",
  253 |   );
  254 |   // 保存が成功しているのにエラーが出ないこと（本文が空のレスポンスで失敗扱いに
  255 |   // なっていた回帰のガード）。
  256 |   await expect(page.locator(".form-alert")).toHaveCount(0);
  257 | });
  258 | 
  259 | test("E2E-WEB-07: 固定メニュールールを確認後に解除できる", async ({ page }) => {
  260 |   await prepareAuthenticatedPage(page);
  261 |   let rules = [{ ruleId: "rule-1", day: 4, recipeId: "recipe-1" }];
  262 |   let deleteRequests = 0;
  263 | 
  264 |   await page.route("http://localhost:3001/**", async (route) => {
  265 |     const request = route.request();
  266 |     const path = new URL(request.url()).pathname;
  267 | 
  268 |     if (path === "/household/fixed-menu-rules" && request.method() === "GET") {
  269 |       return json(route, rules);
  270 |     }
  271 |     if (
  272 |       path === "/household/fixed-menu-rules/rule-1" &&
  273 |       request.method() === "DELETE"
  274 |     ) {
  275 |       expect(request.headers()["x-household-id"]).toBe("e2e-household");
  276 |       deleteRequests += 1;
  277 |       rules = [];
  278 |       await route.fulfill({ status: 204 });
  279 |       return;
  280 |     }
  281 |     if (path === "/recipes") {
  282 |       return json(route, [{ id: "recipe-1", title: "カレーライス" }]);
  283 |     }
  284 |     if (path === "/household/menu-preference") {
  285 |       return json(route, { meatRatio: 50 });
  286 |     }
  287 |     await route.fulfill({ status: 404 });
  288 |   });
  289 | 
  290 |   await page.goto("/rules");
  291 |   const row = page.locator(".rule-row").filter({ hasText: "カレーライス" });
  292 |   await expect(row).toBeVisible();
  293 |   const removeButton = row.getByRole("button", {
  294 |     name: "金曜の固定メニューを解除",
  295 |   });
  296 | 
  297 |   // キャンセルすると行は残り、DELETEは送信されない。
  298 |   await removeButton.click();
  299 |   await expect(page.locator(".dialog")).toBeVisible();
  300 |   await page.getByRole("button", { name: "やめる" }).click();
  301 |   await expect(row).toBeVisible();
  302 |   expect(deleteRequests).toBe(0);
  303 | 
  304 |   // 確認して解除するとDELETEが送信され、行が消える。
  305 |   await removeButton.click();
  306 |   await expect(page.locator(".dialog")).toContainText("金曜: カレーライス");
  307 |   await page.getByRole("button", { name: "解除する" }).click();
  308 |   await expect(row).toHaveCount(0);
  309 |   expect(deleteRequests).toBe(1);
  310 |   await expect(page.getByTestId("toast").last()).toHaveText(
  311 |     "固定メニューを解除しました",
  312 |   );
  313 | 
  314 |   // モバイル幅でも解除ボタンが画面内にあり操作できる。
  315 |   rules = [{ ruleId: "rule-1", day: 4, recipeId: "recipe-1" }];
  316 |   await page.setViewportSize({ width: 320, height: 844 });
  317 |   await page.goto("/rules");
  318 |   const scrollWidth = await page.evaluate(
  319 |     () => document.documentElement.scrollWidth,
  320 |   );
  321 |   const clientWidth = await page.evaluate(
  322 |     () => document.documentElement.clientWidth,
  323 |   );
  324 |   expect(scrollWidth).toBeLessThanOrEqual(clientWidth);
  325 | 
  326 |   const mobileRemoveButton = page.getByRole("button", {
  327 |     name: "金曜の固定メニューを解除",
  328 |   });
  329 |   await expect(mobileRemoveButton).toBeVisible();
  330 |   const box = await mobileRemoveButton.boundingBox();
  331 |   expect(box).not.toBeNull();
  332 |   if (box !== null) {
  333 |     expect(box.x + box.width).toBeLessThanOrEqual(320);
  334 |   }
  335 | });
  336 | 
  337 | test("E2E-WEB-06: パントリーの追加・保存場所変更・削除ができる", async ({
  338 |   page,
  339 | }) => {
  340 |   await prepareAuthenticatedPage(page);
  341 |   const items = [
  342 |     {
  343 |       ingredientId: "egg",
  344 |       name: "卵",
  345 |       storageLocation: "refrigerated",
  346 |     },
  347 |   ];
  348 | 
```