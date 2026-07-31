# 活起來的圖卡 · 翠鳥 WebAR

用手機掃描一張翠鳥圖卡，卡片上就會浮現一隻會輕輕浮動的翠鳥、加上一塊解說面板，並播放翠鳥叫聲。純前端網頁，可直接部署到 Netlify 或 GitHub Pages。

---

## 這是什麼、怎麼運作

- **圖像辨識**：使用 [MindAR](https://github.com/hiukim/mind-ar-js)（image-tracking，A-Frame 版）。它會比對相機畫面與事先編譯好的 `assets/target.mind`，找到圖卡後回報位置。
- **顯示內容**：
  - 一張翠鳥去背圖 (`assets/bird.png`) 以 A-Frame `<a-plane>` 貼在圖卡上方，做「上下浮動 + 微幅縮放」的 2.5D 循環動畫。
  - 一塊深色半透明解說面板（HTML 疊層），文字來自 `js/content.js`。
  - 翠鳥叫聲 (`assets/call.mp3`) 循環播放。
- **事件**：掃到圖卡觸發 MindAR 的 `targetFound`（圖像與面板淡入、音效播放）；離開觸發 `targetLost`（淡出、音效暫停）。
- **開始按鈕**：iOS 必須由使用者手勢才能啟動相機與音效，所以進入頁面會先顯示全螢幕「開始」按鈕，點擊後才啟動 MindAR。

### 使用的版本（升級時請同步 `index.html`）

| 函式庫 | 版本 | CDN |
| --- | --- | --- |
| A-Frame | 1.5.0 | `https://aframe.io/releases/1.5.0/aframe.min.js` |
| MindAR (image-aframe) | 1.2.5 | `https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js` |

> 注意：本專案用官方 CDN 引入，**不需要** npm 或任何建置工具。

### 關於中文面板的設計說明

A-Frame 內建的 `<a-text>` 使用 MSDF 字型，**無法顯示中文**（會變成空白方塊）。為確保中文「清楚可讀」，解說面板改用 HTML/CSS 疊層呈現，並在 `targetFound / targetLost` 時淡入淡出。浮動的翠鳥圖像仍是貼在圖卡上的 A-Frame 3D 物件。

---

## 辨識目標 `.mind`：先測試、再換成自己的

`.mind` 是 MindAR 的辨識目標檔，**無法用程式自動產生**。本專案的做法是：**先用官方範例把流程跑通，再換成你自己的翠鳥卡。**

### (a) 目前狀態：用官方範例 target 測流程（已內建）

- `assets/example-target.mind`：MindAR 官方 image-tracking 範例的目標檔（取自 GitHub `hiukim/mind-ar-js` 的 `examples/image-tracking/assets/card-example/`）。
- `assets/example-target.png`：對應的可列印範例圖片。
- `index.html` 目前**暫時**把辨識目標指向 `assets/example-target.mind`（該行上方有明顯註解）。

測試方式：把 `assets/example-target.png` 印出來、或顯示在另一個螢幕上，開啟本網頁點「開始體驗」後把鏡頭對準它，就會浮現翠鳥與面板。

> 若檔案遺失需重新下載（用 curl 或直接在瀏覽器另存）：
> - 目標檔：<https://raw.githubusercontent.com/hiukim/mind-ar-js/master/examples/image-tracking/assets/card-example/card.mind> → 另存為 `assets/example-target.mind`
> - 範例圖：<https://raw.githubusercontent.com/hiukim/mind-ar-js/master/examples/image-tracking/assets/card-example/card.png> → 另存為 `assets/example-target.png`

### (b) 正式：用你自己的翠鳥圖卡編譯

1. 打開 MindAR 線上編譯器：<https://hiukim.github.io/mind-ar-js-doc/tools/compile>
2. 上傳你的翠鳥圖卡照片（建議：對比清楚、特徵豐富、非純色或重複紋理；正方形或接近的比例最穩）。
3. 按 **Start** 編譯，完成後 **Download** 得到 `targets.mind`。
4. 把它改名為 `target.mind`，覆蓋到 `assets/target.mind`。
5. **改回 `index.html`**：把 `mindar-image` 的 `imageTargetSrc` 從 `assets/example-target.mind` 改回 `assets/target.mind`（那行上方有 `← 測試用官方範例；正式改回…` 的註解）。

> 若一張 `.mind` 內含多張圖，`index.html` 的 `mindar-image-target="targetIndex: 0"` 代表使用第 0 張。

---

## 本機測試（電腦 Chrome）

`localhost` 被瀏覽器視為安全來源，允許使用相機，不需要 HTTPS。

在專案資料夾（`kingfisher-ar/`）執行：

```bash
python -m http.server 8000
```

然後用電腦的 **Chrome** 開啟：<http://localhost:8000>

點「開始體驗」→ 允許相機 → 把鏡頭對準印出或顯示在另一螢幕的圖卡。

> 沒有 Python？也可用 `npx serve` 或 VS Code 的 Live Server 擴充套件。

---

## 手機測試（必須 HTTPS）— 用 GitHub Pages 部署

手機瀏覽器只在 **HTTPS** 下才允許使用相機（`localhost` 不適用於手機）。用 GitHub Pages 免費取得 HTTPS 網址：

1. 在 GitHub 另開一個新的**公開（Public）** repo。
2. 把本專案推上去，**讓 `index.html` 位於 repo 根目錄**（也就是把 `kingfisher-ar/` 裡的檔案放在 repo 根目錄，而非再包一層資料夾）。
   ```bash
   cd kingfisher-ar
   git init
   git add .
   git commit -m "kingfisher webar"
   git branch -M main
   git remote add origin https://github.com/使用者名稱/repo名稱.git
   git push -u origin main
   ```
3. 在 repo 頁面：**Settings → Pages → Source** 選 **Deploy from a branch**，分支選 **main**，資料夾選 **/(root)**，按 **Save**。
4. 等約一分鐘，重新整理 Pages 設定頁，即可取得網址：
   `https://使用者名稱.github.io/repo名稱/`
5. 用手機開啟該網址即可測試。

> ⚠️ **安全警告：此 repo 為公開，切勿放入任何密碼、API 金鑰或資料庫連線字串。** 任何人都看得到 repo 內容。本專案純前端、不需要任何機密，維持這樣即可。

---

## 產生指向網址的 QR code

拿到部署後的 HTTPS 網址後，做一個 QR code 讓人掃描直接進入：

- **線上**：到 <https://www.qr-code-generator.com/> 或 <https://qrcode.tec-it.com/> 貼上網址即可下載。
- **命令列**（Node）：
  ```bash
  npx qrcode "https://使用者名稱.github.io/repo名稱/" -o qr.png
  ```
- **Python**：
  ```bash
  pip install qrcode[pil]
  python -c "import qrcode; qrcode.make('https://使用者名稱.github.io/repo名稱/').save('qr.png')"
  ```

（把上面的 `使用者名稱`／`repo名稱` 換成你 GitHub Pages 實際的網址。）把 `qr.png` 印在圖卡旁邊或展場文宣上，觀眾掃 QR 進網頁 → 再用網頁掃圖卡。

---

## 素材替換清單（要換成別的鳥就改這些）

| 檔案 | 改什麼 | 怎麼做 |
| --- | --- | --- |
| `assets/target.mind` | 圖卡辨識目標（你自己的翠鳥卡） | 依上面「(b) 正式」用線上編譯器重做一份，覆蓋此檔，並把 `index.html` 的 `imageTargetSrc` 改回 `assets/target.mind`。**換圖卡一定要換這個。** |
| `assets/example-target.mind` | 測試用官方範例目標 | 只是為了先跑通流程，正式上線可忽略或刪除。 |
| `assets/bird.png` | 浮現的去背圖 | 換成新物種的**去背 PNG**（透明背景）。建議正方形、512×512 以上。同名覆蓋即可；換檔名要同步改 `index.html` 的 `<img id="birdImg" src="...">`。 |
| `assets/call.mp3` | 叫聲 | 換成新物種的音檔。同名覆蓋即可；換檔名要同步改 `index.html` 的 `<audio id="call" src="...">`。 |
| `js/content.js` | 文字（中文名、學名、分類、保育等級、一句話解說）與音量 | 直接編輯檔內欄位，非工程人員也能改。 |

> 目前 `assets/` 內的 `bird.png`（藍色鳥形色塊）與 `call.mp3`（極短靜音檔）都是**佔位素材**，正式使用前請替換。`target.mind` 也需依上述步驟自行產生。

---

## 專案結構

```
kingfisher-ar/
├─ index.html          # 頁面、AR 場景、事件與音效邏輯
├─ css/style.css       # 版面與面板樣式（手機直式全螢幕）
├─ js/content.js       # 物種內容設定（集中管理，方便替換）
├─ assets/
│  ├─ example-target.mind  # 測試用官方範例目標（index.html 目前指向這個）
│  ├─ example-target.png   # 官方範例的可列印圖片
│  ├─ target.mind      # 你自己的翠鳥卡目標（佔位，請依 README 產生）
│  ├─ bird.png         # 翠鳥去背圖（佔位藍色鳥形，待替換）
│  └─ call.mp3         # 翠鳥叫聲（佔位靜音檔，待替換）
└─ README.md
```
