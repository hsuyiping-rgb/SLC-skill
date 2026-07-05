# SLC-skill (學習共同體課例探究與 16:9 簡報生成技能)

這個儲存庫封裝了「學習共同體（Study of Learning Community, SLC）」課堂公開課影片的分析、轉譯、關鍵畫格擷取重繪、以及 16:9 PowerPoint / 互動式 HTML 簡報排版雙重格式生成流程。

## 🎯 最終產出成品包括：
1. **下載的完整影片** (`output/video.mp4`)
2. **影片的語音轉譯字幕檔** (`output/subtitles.srt` 及其對應逐字稿 `output/transcript.txt`)
3. **簡報投影片（雙格式輸出）**：
   - **PPT 簡報檔** (`output/slides.pptx`)：16:9 寬螢幕比例，文字完全可修正。
   - **HTML 網頁簡報檔** (`output/slides.html`)：基於網頁格式的互動式簡報，支援左右方向鍵與空白鍵切換、自動響應式排版，完美搭配重繪插圖與主題配色。

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

### 4. 生成可編輯文字的 16:9 PPTX / HTML 雙格式簡報
執行腳本即可產出排版平衡、字級自動計算且不溢出頁面的簡報檔（依據選擇風格傳入對應的 `--style` 參數，HTML 與 PPTX 會同步於同目錄下生成）：
```bash
# 產生簡報 (例如使用 learning 配色，會自動產出 output/slides.pptx 以及 output/slides.html)
uv run scripts/classroom_analyzer_helper.py generate-slides --analysis output/analysis.txt --output output/slides.pptx --style learning

# 產生 FB/IG 分享概念圖
uv run scripts/classroom_analyzer_helper.py generate-image --text output/concept.txt --output output/concept_post.png --style learning
```

## 📄 授權協議

本專案採用 [MIT License](LICENSE) 授權。
