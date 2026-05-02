# 冰火大戰 game.html 修復 Planning

## 目標
修復 `game.html` 嘅 JavaScript 問題——`playCard`、`showPopup` 等函數未被定義（`undefined`），導致互動試玩同 Popup 功能完全失效。

## 現有問題

### 症狀
- `typeof playCard` 返回 `undefined`
- `typeof showPopup` 返回 `undefined`
- Popup overlay 預設 hidden 但點擊合法卡牌後唔會彈出
- 卡牌 click event listeners 綁定咗但 `playCard` 函數唔存在

### 懷疑根因
HTML 結構有問題，`</footer>` 同 `<script>` 擺放位置可能導致瀏覽器唔執行 script。

### 現有 HTML 結構（懷疑有問題）
```html
</section></main>
<footer>冰火大戰 Ice & Fire · 整數策略卡牌遊戲 · 2–6 人適玩</footer>
<script>function scrollTo(id){...}...playCard...showPopup...</script>
<!-- Popup Overlay -->...popup HTML...</div></body></html>
```

## 修復方案

### 方案：將 JavaScript 移到獨立檔案

1. **創建 `game.js`**：將 `<script>` 內所有 JavaScript 代碼抽出，存到 `game.js`
2. **更新 `game.html`**：
   - 移除 `<script>...</script>` 塊
   - 在 `</body>` 前加入 `<script src="game.js"></script>`
3. **更新 `index.html`**（如有 reference）
4. **確保 Popup HTML 在 `</body>` 前**

### 驗證方法
修復後喺瀏覽器打開 `game.html`，Console 輸入：
```js
typeof playCard  // 應該返回 "function"
typeof showPopup // 應該返回 "function"
```

點擊 `-1` 卡牌，應該：
1. Popup 彈出顯示 `"-1"`
2. 卡牌變灰（played state）
3. 累積總和更新

## 具體代碼位置

### game.html 關鍵位置（修復前）
- Line ~27141: `</section></main>` 結束 main content
- Line ~27149: `<footer>...` 開始 footer
- Line ~27221: `</footer>` 結束 footer  
- Line ~27230: `<script>...` 開始 JavaScript
- Line ~37330: `<!-- Popup Overlay -->` 開始 popup HTML
- Line ~37762: `</body></html>` 結束文檔

### JavaScript 函數清單（需抽出到 game.js）
```javascript
// 必須函數
function scrollTo(id) {...}
function playCard(el, val) {...}
function showPopup(value) {...}
function closePopup(e) {...}
function closePopupDirect() {...}
function isValidPlay(current, delta) {...}
function resetDemo() {...}
function toggleHint() {...}
function showHint() {...}
function showHintText(text) {...}
function playSound(type) {...}
function loadStats() {...}
function saveStats(s) {...}
function updateStatsDisplay() {...}
function updateDisplay(n) {...}
function updateHistory() {...}
function burstParticles(x, y, type) {...}
function installApp() {...}

// 全域變數
let currentTotal = 5;
let history = [];
let hintVisible = false;
let deferredPrompt;

// 初始化代碼（立即執行）
updateStatsDisplay();
updateDisplay(currentTotal);

// Event listeners
document.querySelectorAll('.demo-card.dc-ice').forEach(...)
document.querySelectorAll('.demo-card.dc-fire').forEach(...)
document.querySelectorAll('.play-card,.demo-card').forEach(...) // magnetic hover
window.addEventListener('beforeinstallprompt', ...)
navigator.serviceWorker.register('sw.js')

// 滑動 progress bar
window.addEventListener('scroll', ...)

// Intersection observer for nav tabs
const observer = new IntersectionObserver(...)

// Custom cursor
const cursor = document.createElement('div'); ...
```

## 注意事項
- 唔好使用拼音做網址（本 project 用 `bingfo` 作為 URL slug，已係正確嘅英文）
- Popup overlay HTML 結構要保持完整，包含 `popupOverlay`, `popupCard`, `popupValue`, `popupLabel`, `popupEffect`, `popupClose` 等元素
- 確保 CSS class `.popup-overlay`, `.popup-overlay.active` 等保持對應
