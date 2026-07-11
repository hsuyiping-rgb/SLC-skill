# SLC-skill (學習共同體課例探究與 16:9 簡報生成技能)

這個儲存庫封裝了「學習共同體（Study of Learning Community, SLC）」課堂公開課影片的分析、轉譯、關鍵畫格擷取重繪、以及 16:9 PowerPoint / 互動式 HTML 簡報排版雙重格式生成流程。

## 🎯 最終產出成品包括：
1. **下載的完整影片** (`output/video.mp4`)
2. **影片的語音轉譯字幕檔** (`output/subtitles.srt` 及其對應逐字稿 `output/transcript.txt`)
3. **簡報投影片（雙格式輸出）**：
   - **PPT 簡報檔** (`output/slides.pptx`)：16:9 寬螢幕比例，文字完全可修正。
   - **HTML 網頁簡報檔** (output/slides.html)：基於網頁格式的互動式簡報，專為平板與手機等載具進行響應式自適應排版，支援 **Touch Swipe 手勢左右滑動切換頁面** 與鍵盤切換，提供最優質的閱讀感受。

## 📝 課例研究分析與插圖生成規範

當執行本技能時，應遵循以下七大核心規範：

1. **時間戳記標記**：課例研究分析報告必須以影片的**時間碼（如 `12:34`）**精確標記課堂教學事件與學生互動。
2. **三階層分析**：分析明確區分並落實**「描述－詮釋－反思」**三個分析層次。
3. **聚焦微觀行為**：聚焦學生的**語言、眼神、姿態、手勢、沉默與同伴互動**。
4. **探究多維關係**：分析學生與**教材、同伴及先前理解**之間的關係。
5. **SLC 哲學探究**：探究**傾聽關係、互惠學習、言談權力、伸展跳躍任務及質性時間**。
6. **去評價、去建言立場**：採取客觀、描述性的**去評價與去建言立場**，聚焦於學生的「學習事實」本身，避免將觀課議課變成對授課教師的優缺點打分或指導。
7. **日系水彩／抹茶綠繪本生圖風格**：
   - 擷取畫格並使用 `generate_image` (GPTimage2) 進行重繪插圖時，必須指示繪圖引擎將畫面重繪為一致的**「日系水彩／抹茶綠繪本風格」**（Japanese watercolor / Matcha green picture book style）。
   - **畫面主體聚焦**：畫面必須聚焦在學生的**學習、傾聽、指著圖表（指圖）與共同推理**。
   - **角色過濾規則**：**完全移除外圍的觀課教師**；而**授課教師**只有在「直接參與學生學習與小組互動」時才得以保留於重繪畫面中。

## 📁 目錄結構

```
SLC-skill/
├── .gitignore
├── LICENSE
├── README.md
├── SKILL.md
└── scripts/
    └── classroom_analyzer_helper.py
```

## 🎨 支援的簡報風格選項（提供使用者選擇）：
1. **`learning`**：學習共同體哲學配色（抹茶綠/茶白背景、松針綠標題、石板灰內文）。
2. **`pastel`**：暖色教育風（暖白背景、深褐標題、柔黑內文）。
3. **`blue`**：科技藍商務風（淡藍灰背景、海軍藍標題、藍灰內文）。
4. **`modern`**：極簡黑白風（極淡灰背景、曜石黑標題、中灰內文）。

---

## 🚀 快速使用指南

### 1. 課堂影片準備
如果是私密影片，請先取得登入 Cookie 並匯出為 `cookies.txt`，接著下載完整影片檔案並擷取音訊進行轉譯：
```bash
# 下載完整影片 (mp4 格式)
yt-dlp --cookies cookies.txt --js-runtimes node --remote-components ejs:github -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]" -o output/video.mp4 "https://www.youtube.com/watch?v=VIDEO_ID"

# 分離出音訊檔
ffmpeg -i output/video.mp4 -q:a 0 -map a output/audio.mp3

# 使用 Whisper 轉譯生成 srt 字幕與逐字稿
whisper output/audio.mp3 --model medium --language zh -o output/ -f srt
```

### 2. 詢問 NotebookLM 連接與課例研究分析
- **詢問使用者**：「請問您有沒有要連接 NotebookLM 的筆記來作為簡報分析的架構？如果有，請提供筆記的名稱（與內容）。」
- 將 NotebookLM 筆記的結構與內容，作為生成簡報分析報告 (`analysis.txt`) 的核心架構，再對齊語音逐字稿進行「描述-詮釋-反思」三階層分析。

### 3. 擷取影片畫面與 GPTimage2 重繪
利用 `ffmpeg` 擷取影片關鍵影格：
```bash
ffmpeg -y -ss 00:01:12 -i output/video.mp4 -vframes 1 -q:v 2 output/screenshots/screenshot_4.png
```
隨後使用 AI 影像生成工具，以該截圖為基底（`ImagePaths`）重繪為「抹茶綠風格的日本溫潤水彩繪本插畫」，儲存至 `output/images/slide_{N}.png`。

### 4. 生成可編輯文字的 16:9 PPTX / HTML 雙格式簡報 (動態規劃 15-20 頁)
執行腳本即可產出排版平衡、字級自動計算且不溢出頁面的簡報檔。
- **動態頁數**：會依據分析內容長短，將投影片內容頁數自動規劃在 **15 至 20 頁** 之間。
- **行動自適應與手勢**：HTML 簡報針對手機/平板進行響應式佈局優化，並加入 **Touch Swipe 左右滑動觸控手勢** 換頁，確保不同載具上均有最優質的閱讀感受。
- 依據選擇風格傳入對應的 `--style` 參數，HTML 與 PPTX 會同步於同目錄下生成：
```bash
# 產生簡報 (例如使用 learning 配色，會自動產出 output/slides.pptx 以及 output/slides.html)
uv run scripts/classroom_analyzer_helper.py generate-slides --analysis output/analysis.txt --output output/slides.pptx --style learning

# 產生 FB/IG 分享概念圖
uv run scripts/classroom_analyzer_helper.py generate-image --text output/concept.txt --output output/concept_post.png --style learning
```

## 📄 授權協議

本專案採用 [MIT License](LICENSE) 授權。
