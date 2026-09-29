# 超級數學瑪利歐 · 開發者文檔

> **三位數加減大冒險** — 一款以「瑪利歐」視覺風格包裝的國小數學練習遊戲。
> 單一 HTML 檔案、零後端、零建置工具，直接在瀏覽器開啟即可遊玩。

---

## 1. 專案總覽

| 項目 | 內容 |
|------|------|
| 檔名 | `index.html` |
| 型態 | 單一檔案網頁遊戲（Single-file web game） |
| 語言 | 純 HTML + CSS + JavaScript（Vanilla，無框架） |
| 目標對象 | 國小學童（練習二／三位數進位加法與退位減法） |
| 視覺風格 | 復古 8-bit 瑪利歐（pixel font、積木、金幣、磚塊地面） |
| 教學機制 | 積木視覺化 + 直式算盤 + 逐步引導 + 點珠提示 |
| 音效 | Web Audio API 即時合成（無外部音檔） |

**核心教學理念**：不直接讓孩子填答案，而是透過「積木搬移動畫」把抽象的
進位（Carry）與退位（Borrow）具象化 —— 10 顆個位珠合成 1 條十位長條、
10 條十位長條合成 1 塊百位方塊。

---

## 2. 技術棧與外部依賴

| 依賴 | 用途 | 載入方式 |
|------|------|----------|
| [Tailwind CSS 3.4.16](https://cdn.tailwindcss.com/3.4.16) | 版面配置、間距、響應式 | CDN（`<script>`） |
| Google Fonts · Press Start 2P | 像素字體（標題、數字、按鈕） | CDN（`<link>`） |
| Google Fonts · Noto Sans TC | 正體中文字體（內文、說明） | CDN（`<link>`） |
| Web Audio API | 全部音效即時合成 | 瀏覽器內建 |

> ⚠️ **依賴注意事項**：Tailwind 走 CDN、字體走 Google Fonts，**離線環境無法遊玩**。
> Tailwind CDN 版本是「即時編譯」模式，開啟時會先閃一下未套樣式的內容（FOUC）。

---

## 3. 檔案結構（單一檔案的邏輯分區）

檔案從上到下可分為六個邏輯區塊：

```
index.html
├── <head>
│   ├── meta / title
│   ├── 外部資源（字體、Tailwind CDN）
│   └── <style>            ← 全部自訂 CSS（設計代幣、積木、動畫）
├── <body>
│   ├── #app
│   │   ├── #screen-menu    ← 主選單（選類別 + 位數）
│   │   ├── #screen-level   ← 關卡選擇
│   │   ├── #screen-game    ← 遊戲主畫面
│   │   └── #screen-result  ← 結算畫面
│   ├── #bead-hint          ← 點珠提示（浮動）
│   ├── #report-modal       ← 學習診斷報告（Modal）
│   └── <script>            ← 全部遊戲邏輯
```

---

## 4. 畫面流程（狀態機）

遊戲在 4 個畫面間切換，由 `showScreen(name)` 統一控制（透過切換 `.hidden` class）。

```
        ┌──────────► menu ──► level ──► game ──► result ─┐
        │                              ▲        │        │
        └──────────────────────────────┘        └────────┘
                 (放棄/返回)            (再玩/下一關/選關卡)
```

| 畫面 | ID | 進入方式 | 離開方式 |
|------|-----|---------|---------|
| 主選單 | `#screen-menu` | 啟動預設 | `pickCategory()` |
| 關卡選擇 | `#screen-level` | `pickCategory()` | `startGame()` 或 `goMenu()` |
| 遊戲中 | `#screen-game` | `startGame()` | `finishLevel()` 或 `quitGame()` |
| 結算 | `#screen-result` | `finishLevel()` | 四個按鈕其中之一 |

**位數選擇**：主選單左右兩欄分別是「二位數」與「三位數」；每欄各有
「加法大關」「減法大關」「加減綜合」三個按鈕，共 **6 種進入路徑**。

---

## 5. 設計系統（CSS）

### 5.1 設計代幣（CSS 變數）

定義在 `:root`，全站配色都從這裡取，改色只需改這一處：

| 變數 | 值 | 用途 |
|------|-----|------|
| `--sky` | `#5c94fc` | 天空背景 |
| `--mario` | `#e84a23` | 瑪利歐紅（主按鈕、百位標頭、運算子） |
| `--grass` | `#00a800` | 草地綠（十位標頭、確定鍵） |
| `--pipe` | `#0058f8` | 水管藍（減法、答案字色） |
| `--coin` | `#f8d820` | 金幣黃（數字鍵盤、個位） |
| `--panel` | `#fcd8a8` | 面板木色 |
| `--ground` | `#882000` | 地面棕紅 |
| `--border` | `#b89800` | 邊框金棕 |
| `--ink` | `#3a2a00` | 深棕墨水字 |

### 5.2 關鍵 CSS Class

| Class | 用途 |
|-------|------|
| `.pixel` | 套用 Press Start 2P 像素字體 |
| `.btn-3d` | 3D 按鈕（厚底邊 + 按下位移） |
| `.ground` | 底部磚塊地面（重複漸層） |
| `.blk-h` / `.blk-t` / `.blk-u` | 積木三型：百位方塊(10×10格) / 十位長條 / 個位單元 |
| `.blocks-grid` | 積木區的 4 欄網格（標籤 + 百/十/個） |
| `.math-board` | 直式算盤容器 |
| `.digit-cell` | 直式算盤的單一數字格 |
| `.carry-input` | 進位標記框（虛線小框） |
| `.borrow-input` | 退位標記框（虛線框） |
| `.main-ans` | 主答案輸入框 |
| `.flying-block` | 飛行動畫中的積木（`position:fixed`） |

### 5.3 動畫關鍵影格

| @keyframes | 用途 |
|------------|------|
| `pop` | 元素彈出（選單、點珠） |
| `drop` | 積木落下降臨 |
| `blinkPair` | 相減時配對積木閃爍 |
| `shake` | 錯誤時搖晃 |
| `floatUp` | （保留，未大量使用） |
| `beadIn` | 點珠逐顆出現 |
| `pulseInput` | 提示「現在該填哪一格」的脈衝光 |
| `pulseBtn` | 「借位」按鈕脈衝提示 |

---

## 6. 全域狀態（`state` 物件）

所有遊戲狀態集中在單一 `state` 物件，**沒有用任何全域散落變數**（除了少數
`autoTimer`、`beadTimer` 計時器）。

```js
const state = {
  screen: 'menu',     // 目前畫面：'menu' | 'level' | 'game' | 'result'
  category: 'add',    // 類別：'add' | 'sub' | 'mixed'
  digits: 3,          // 位數：2 | 3
  op: '+',            // 運算：'+' | '−'
  level: 1,           // 關卡 1~4
  qNum: 1,            // 目前題號
  maxQ: 10,           // 每關題數（固定 10）
  score: 0, coins: 0, // 得分、金幣
  n1: 0, n2: 0,       // 兩個運算元
  d: {},              // 拆解後的各位數 {h1,t1,u1,h2,t2,u2}
  phase: 0,           // 加法階段機（0個位→1十位→2百位→3完成）
  step: 'u_check',    // 減法步驟機
  cur: { h: 0, t: 0, u: 0 }, // 減法被減數「目前剩餘值」（借位後會變）
  isAnimating: false, // 動畫鎖（避免輸入與動畫競態）
  attempts: 0,        // 本題嘗試次數
  errorDetails: [],   // 本題錯誤診斷文字
  animPlayed: { h: false, t: false, u: false }, // 減法各欄動畫是否已播
  report: [],         // 全關作答紀錄
  boardOp: null,      // 已渲染的算盤運算符（+ 或 −，換運算才重畫）
  activeIds: [],      // 加法目前可輸入的 input id 清單
  focusEl: null,      // 最後聚焦的 input（供數字鍵盤定位）
  lastCol: null       // 減法最後操作的欄位 {type, count}（供重播）
};
```

---

## 7. 關卡設定（`LEVELS`）

```js
const LEVELS = {
  add:   [ 無進位, 一次進位, 兩次進位, 綜合練習 ],
  sub:   [ 不退位, 一次退位, 兩次退位, 綜合挑戰 ],
  mixed: [ 基礎混合, 進退位混合, 雙重進退位, 全部混合 ],
};
const LEVEL_NAMES = { add: '加法大關', sub: '減法大關', mixed: '加減綜合' };
```

- 每類別 4 關，關卡顯示為「世界-關卡」格式（例 `1-2`）。
- **第 4 關 = 綜合**：在出題時把 level 隨機映射回 1~3。
- 出題難度由 `LEVELS` 描述 + `makeAdd`/`makeSub` 內的數值範圍共同決定。

---

## 8. 出題引擎

### `makeAdd(level, digits)` — 加法出題

- `level === 4` 時先 `randInt(1,3)` 隨機降階。
- 依位數（2 或 3）與關卡控制「個位／十位」的數字範圍，確保進位次數符合關卡：
  - **無進位**：個位和 ≤ 9、十位和 ≤ 9
  - **一次進位**：個位或十位其中一組會進位
  - **兩次進位**：個位和十位都進位
- 回傳 `{ n1, n2, d: {h1,t1,u1,h2,t2,u2} }`。

### `makeSub(level, digits)` — 減法出題

- 保證 **被減數 ≥ 減數**（結果不為負）。
- 依關卡控制哪一位需要退位（借位）：
  - 二位數不退位、一次退位、兩次退位（兩次退位時被減數補到三位，如 `100+`）。
  - 三位數 `h1` 固定 4~9 起跳，確保退位後仍有值可借。

> 💡 **設計重點**：出題引擎刻意讓「位數」與「進退位次數」可控，
> 是這個遊戲的教學分級核心。擴充關卡時，改 `LEVELS` 描述 + 這裡的數值範圍即可。

---

## 9. 積木視覺化系統

### 三種積木（對應「十進位」位值概念）

| 型別 | Class | 對應 | 視覺 |
|------|-------|------|------|
| 百位方塊 | `.blk-h` | 100 | 38px 方塊，內建 10×10 格線 |
| 十位長條 | `.blk-t` | 10 | 7px 寬長條，內建 10 格 |
| 個位單元 | `.blk-u` | 1 | 13px 金幣方塊 |

`createBlock(type)` 產生對應元素（`.blk-h` 塞 100 個 `<i>`、`.blk-t` 塞 10 個）。

### 積木區網格（`.blocks-grid`）

```
        │ 百位 │ 十位 │ 個位 │
────────┼──────┼──────┼──────┤
 加數1   │ blk-a-h │ blk-a-t │ blk-a-u │   ← row A
 加數2   │ blk-b-h │ blk-b-t │ blk-b-u │   ← row B（含進位停靠點 stop-h/stop-t）
 總和    │ blk-c-h │ blk-c-t │ blk-c-u │   ← row C（結果區）
```

- **加法**：row A = 加數1、row B = 加數2，動畫把 A+B 搬到 row C。
- **減法**：row A = 原有數量（被減數）、row B = 被減掉的數量，動畫把
  配對的積木畫叉消失，剩下的移到 row C。
- row B 的百位／十位格內各有一個 `carry-stop-point`（`#stop-h`/`#stop-t`），
  是加法進位時「10 合成 1」飛進去的停靠點。

---

## 10. 直式算盤（`#math-board`）

`renderBoard()` 依 `state.op` 與 `state.digits` 渲染不同版面。**只在運算符
改變時才重畫**（`boardOp` 快取），避免每次出題都重建 DOM。

### 加法版面

- **2 位數**：2 個主答案欄 + 2 個進位欄
- **3 位數**：3 個主答案欄（`in-th/in-h/in-t/in-u`）+ 3 個進位欄
  （`carry-in-th/carry-in-h/carry-in-t`）
- 進位欄（`.carry-input`）是主答案格右上方的小虛線框，孩子要「先填進位標記」。

### 減法版面

- 最上方兩列 `borrow-row` 是「退位標記框」：
  - `in-h-b`（百位退位後剩多少）、`in-t-b1`（十位退位後剩多少）
  - `in-t-b2`（十位借位後變多少）、`in-u-b`（個位借位後變多少）
- 中間兩列顯示被減數 `n1-*` 與減數 `n2-*`。
- 底部 `ans-row` 是答案欄 `ans-h/ans-t/ans-u`。

---

## 11. 加法流程（Phase 狀態機）

加法用 `state.phase` 做三階段逐步引導（從個位開始，因為進位要從低位往上算）。

```
phase 0（個位）─► phase 1（十位）─► phase 2（百位）─► phase 3（完成）
```

### 關鍵流程

1. `updateAddUI()`：依 phase 啟用/停用對應輸入框，並聚焦第一個。
   - `addActiveInputs()` 回傳該 phase 應啟用的 input id：
     - phase 0 → `['in-u', 'carry-in-t']`
     - phase 1 → `['in-t', 'carry-in-h']`
     - phase 2 → `['in-h', 'in-th', 'carry-in-th']`
2. `onAddInput()`：偵測主答案欄有值後，**1.2 秒後自動** `processAddCheck()`。
3. `processAddCheck()`：核心驗證。依 phase 計算正確答案與進位值，比對使用者輸入。

**進位計算**（在 `processAddCheck` 內）：
```js
carryT  = floor((u1 + u2) / 10)          // 個位往十位的進位
carryH  = floor((t1 + t2 + carryT) / 10) // 十位往百位的進位
carryTh = floor((h1 + h2 + carryH) / 10) // 百位往千位的進位（三位數加法的最高進位）
```

- 答對：播放該欄動畫 → `markCorrect()` → 進下一 phase（或 `finishQuestion()`）。
- 答錯：`AudioFX.error()` → `showBeadHint()` 提示 → 清空答案重試，並記錄錯誤。

---

## 12. 減法流程（Step 狀態機）

減法用 `state.step` 字串做狀態機，比加法更複雜（因為要「先處理退位、再逐欄作答」）。

```
u_check ──► u_borrow / u_ans ──► t_check ──► t_borrow / t_ans ──► h_ans ──► done
```

- `checkSubStep()` 判斷目前欄位「夠不夠減」：
  - **夠減** → 直接開啟答案欄讓孩子填。
  - **不夠減** → 顯示「🔨 不夠減！我要借位」按鈕，要求先完成退位。
- `onBorrowClick()`：進入退位流程，要求孩子填兩個框
  （例：十位退位後剩多少、個位加 10 後變多少）。
- `validateBorrow(type)`：驗證退位數字，正確則**播放積木借位動畫**
  （從十位拿走一條長條、在個位補 10 顆單元），並更新 `state.cur`。
- `validateSubAnswer(type)`：驗證答案欄，正確則播放該欄相減動畫並前進。

> 減法的 `state.cur` 是**動態的**：借位後被減數的各位會改變，因此出題後的
> `state.cur` 初始值 = `{h:d.h1, t:d.t1, u:d.u1}`，借位成功後才更新。

---

## 13. 動畫系統

### 加法動畫（把積木搬到結果區 + 進位合成）

| 函式 | 行為 |
|------|------|
| `gatherBlocks(srcIds, targetId, type)` | 收集來源格的積木 → 閃光高亮 → 以 `.flying-block` 克隆飛到目標格 |
| `playUnitAnimation(u1, u2)` | 個位相加；若 ≥10，把 10 顆單元合成 1 條十位長條飛進 `stop-t` |
| `playTenAnimation(t1, t2, carryT)` | 十位相加；若 ≥10，合成 1 塊百位方塊飛進 `stop-h` |
| `playHundredAnimation(h1, h2, carryH)` | 百位相加（僅高亮，不再往上進位） |

「10 合成 1」的實作：把結果區 10 個積木 `carry-highlight` 高亮 → 建立一個
飛行克隆 → 把原本 10 個設透明 → 在停靠點生成 1 個上一級積木 → 結果區重繪剩餘。

### 減法動畫（配對畫叉 → 剩餘落下）

| 函式 | 行為 |
|------|------|
| `playColumnAnimation(type, count)` | 來源兩列的 `count` 個積木 `blink-pair` 閃爍 → `cross-out` 畫叉 → 消失；剩餘積木 `drop-in` 落入結果列 |
| `replayColumn(type, count)` | 重播某欄動畫（重建該欄 DOM 再播一次） |

---

## 14. 點珠提示（`showBeadHint`）

錯誤時在畫面底部彈出的浮動提示，把「抽象的數字」變成「可數的珠子」。

```js
showBeadHint(value, color, title)
```

- `value`：要顯示的珠子數（上限 30 顆）。
- 每 10 顆換行（`bead-break`），第 11 顆起加 `.bead-overflow` 紅框，
  視覺化「滿 10 進一」的概念。
- 3 秒後自動隱藏（`beadTimer`）。

---

## 15. 音效系統（`AudioFX`）

用 **Web Audio API 即時合成**，無任何音檔。以 IIFE 封裝，回傳一組具名音效函式。

| 方法 | 實作 | 用途 |
|------|------|------|
| `coin()` | 兩個方波 tone（B5→E6） | 撿金幣（答對欄位） |
| `oneup()` | 6 音符上行琶音 | 進位成功（1-up） |
| `correct()` | 三角波雙音 | 整題完成 |
| `error()` | 鋸齒波下滑 | 答錯 |
| `pop()` | 短促正弦波 | 積木移動 |
| `fly()` / `drop()` | 正弦波滑音 | 飛行 / 落下 |
| `slash()` | 鋸齒波下滑 | 畫叉 |
| `click()` | 方波短音 | 按鍵 |
| `applause()` | 30 段白噪音 | 過關掌聲 |
| `fireworks()` | 4 段噪音 + 滑音 | 過關煙火 |
| `levelComplete()` | 4 音三角波 | 過關音階 |

底層原語：`tone()`（單音）、`glide()`（滑音）、`noise()`（噪音）。
`ac()` 惰性建立 `AudioContext` 並在 suspended 時 `resume()`（滿足瀏覽器自動播放政策）。

> 💡 首次使用者互動（`document.body` 的 `{once:true}` click 監聽器）會呼叫一次
> `AudioFX.click()`，用於解鎖 AudioContext。

---

## 16. 輸入系統

遊戲支援三種輸入方式，全部指向「目前聚焦/啟用的 input」：

1. **螢幕數字鍵盤**（`#numpad`）：12 個鍵（0-9、清除、確定），
   `numpadPress(k)` 把字元寫入 `state.focusEl` 或第一個啟用中的 input。
2. **實體鍵盤**：`keydown` 監聽（加法時 Enter = 送出；數字直接打進 input）。
3. **滑鼠/觸控**：直接點擊 input。

**輸入定位**：`document` 的 `focusin` 監聽器記錄 `state.focusEl`；
`numpadPress` 若找不到有效焦點，會退而找 `#math-board` 內第一個 `active` 或可用的 input。

**防呆機制**：
- `.main-ans` 被點擊但 `disabled` 時，會顯示「先完成前面的步驟」的提示。
- `maxlength="1"` 限制答案欄只能填 1 位（退位框則 `maxlength="2"`）。

---

## 17. 計分與結算

### `finishQuestion()`（單題完成）

```js
bonus = (attempts === 0) ? 50 : 0;   // 一次答對 +50 分
score += 100 + bonus;                // 每題基本 100 分
coins  += 1;
```

- 滿分 = `10 × 150 = 1500` 分、`10` 金幣。
- 每題把 `{level, q, attempts, errors}` push 進 `state.report`。
- 第 10 題完成 → 播過關音效/掌聲/煙火 → `finishLevel()`；否則顯示「下一題」按鈕。

### `finishLevel()`（關卡結算）

- 一次答對率 = `(attempts===0 的題數 / 總題數) × 100`。
- 評價 RANK：`S (100%)` / `A (≥80%)` / `B (≥60%)` / `C (<60%)`。
- 第 4 關的「下一關」按鈕改為「選關卡」（因為沒有第 5 關）。

---

## 18. 學習報告（`showReport` / `closeReport`）

結算畫面的「📊 學習報告」按鈕開啟 `#report-modal`：

- **摘要列**：一次答對率、總錯誤修正次數、完成題數。
- **明細表**：每題的關卡、算式、嘗試次數、錯誤診斷（依 `attempts` 上色：
  0=綠、≤2=黃、>2=紅）。
- 錯誤診斷文字由 `logError()` 累積在 `errorDetails`，例如「個位答案錯誤」
  「百位進位標記錯誤」「十位退位數值輸入錯誤」。

---

## 19. 函式索引

| 函式 | 類別 | 說明 |
|------|------|------|
| `$` / `sleep` / `randInt` | 工具 | DOM 查詢 / 延遲 / 隨機整數 |
| `showScreen` / `goMenu` / `pickCategory` | 畫面 | 畫面切換 / 回選單 / 選類別 |
| `startGame` / `quitGame` / `nextQuestion` | 流程 | 開始 / 放棄 / 出下一題 |
| `resetQuestionUI` / `fillDigits` / `renderBlocks` | 流程 | 重置 UI / 填數字 / 畫積木 |
| `makeAdd` / `makeSub` | 出題 | 加法 / 減法出題 |
| `createBlock` | 積木 | 建立積木元素 |
| `renderBoard` | 算盤 | 渲染直式算盤 |
| `bindAddInputs` / `addActiveInputs` / `updateAddUI` / `onAddInput` / `markCorrect` / `processAddCheck` | 加法 | 加法輸入與驗證 |
| `gatherBlocks` / `playUnitAnimation` / `playTenAnimation` / `playHundredAnimation` | 加法動畫 | 加法積木動畫 |
| `bindSubInputs` / `checkSubStep` / `onBorrowClick` / `validateBorrow` / `validateSubAnswer` / `playColumnAnimation` / `replayColumn` | 減法 | 減法流程與動畫 |
| `logError` | 工具 | 記錄錯誤診斷 |
| `showBeadHint` | 提示 | 點珠提示動畫 |
| `finishQuestion` / `finishLevel` | 結算 | 單題 / 關卡結算 |
| `showReport` / `closeReport` | 報告 | 開啟 / 關閉學習報告 |
| `numpadPress` | 輸入 | 數字鍵盤輸入 |
| `AudioFX.*` | 音效 | 13 種合成音效 |

---

## 20. 擴充指南

### 新增位數（例如四位數）

1. 主選單加一個「四位數」欄 + 三個按鈕（`pickCategory('add', 4)`）。
2. `makeAdd`/`makeSub` 增加 `dg === 4` 的數值分支。
3. `renderBoard` 的加法版面加一組 `in-thh` / `carry-in-thh` 欄位。
4. `state.cur` 加 `th` 欄位，`phase` 機加一個階段。
5. `ADD_INPUTS` 陣列與 `addActiveInputs()` 補上對應 id。

### 新增關卡難度

1. `LEVELS[cat]` 增加 `{lv:5, name, desc}`。
2. `makeAdd`/`makeSub` 增加對應的數值範圍分支。
3. `finishLevel` 的「下一關」上限判斷（目前是 `level < 4`）改為 `level < 5`。

### 新增運算（例如乘法）

1. `state.op` 增加 `'×'` 分支（但 `category` 值與 `LEVELS` 需同步擴充）。
2. 積木系統可直接重用（乘法可視為「重複加法」的積木累加）。
3. 直式算盤 `renderBoard` 需新增乘法版面。

### 調整題數

改 `state.maxQ`（目前硬編碼為 10）。若希望可配置，可改成從 `startGame` 參數帶入。

---

## 21. 已知限制與注意事項

| 限制 | 說明 |
|------|------|
| **離線不可用** | Tailwind CDN 與 Google Fonts 需連網；離線會失去樣式與字體。 |
| **單一檔案耦合** | 樣式、結構、邏輯全在一個 HTML，超過 1000 行；大改建議拆分 CSS/JS 檔。 |
| **全域計時器** | `autoTimer`（自動送答）、`beadTimer`（點珠隱藏）未做清理，快速切換畫面時可能有殘留計時器。 |
| **`isAnimating` 競態** | 動畫期間大量函式靠 `if (state.isAnimating) return` 防重入；若新增非同步流程須記得加。 |
| **進位上限 3 位數** | 加法最多算到「千位進位」（`carry-in-th`）；減法退位最多兩次。四位數需擴充（見第 20 節）。 |
| **`finishLevel` 除零風險** | `total = report.length || 1` 已防禦，但若未來改題數邏輯需注意。 |
| **音效需使用者互動** | AudioContext 需首次點擊才解鎖（已有 `{once:true}` 監聽器處理）。 |
| **無持久化** | 分數、進度不存 localStorage，重新整理即歸零。 |

---

## 22. 快速上手（開發者）


**除錯技巧**：程式內建全域 `window.addEventListener('error')` 錯誤橫幅，
JS 出錯時會在畫面底部顯示紅底錯誤訊息（含檔名與行號），方便即時定位。

**改色**：全部配色集中在 `:root` 的 CSS 變數（第 5.1 節），改主題色只需改一處。
