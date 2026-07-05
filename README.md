# SLC-skill (學習共同體課例探究與 16:9 簡報生成技能)

這個儲存庫封裝了「學習共同體（Study of Learning Community, SLC）」課堂公開課影片的分析、轉譯、關鍵畫格擷取重繪、以及可編輯文字的 16:9 PowerPoint 簡報排版生成流程。

## 🎯 最終產出成品包括：
1. **下載的完整影片** (`output/video.mp4`)
2. **影片的語音轉譯字幕檔** (`output/subtitles.srt` 及其對應逐字稿 `output/transcript.txt`)
3. **可編輯文字的 16:9 簡報投影片** (`output/slides.pptx`)

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

## 🛠️ 安裝依賴

本專案需要 Python 3.10+ 及 FFmpeg。推薦使用 Python 工具 `uv` 來快速安裝並執行。

```bash
# 安裝 Python 庫依賴
pip install python-pptx Pillow
# 或者使用 uv 執行時自動載入（參見 Quick Start）
```

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

### 2. 進行課例研究分析
對照逐字稿進行「描述 ➔ 詮釋 ➔ 反思」分析，並將結果儲存於 `output/analysis.txt`。

### 3. 擷取影片畫面與 GPTimage2 重繪
利用 `ffmpeg` 擷取影片關鍵影格：
```bash
ffmpeg -y -ss 00:01:12 -i output/video.mp4 -vframes 1 -q:v 2 output/screenshots/screenshot_4.png
```
隨後使用 AI 影像生成工具，以該截圖為基底（`ImagePaths`）重繪為「抹茶綠風格的日本溫潤水彩繪本插畫」，儲存至 `output/images/slide_{N}.png`。

### 4. 生成可編輯文字的 16:9 PPTX 簡報
執行腳本即可產出排版平衡、字級自動計算且不溢出頁面的 16:9 寬螢幕簡報檔（依據選擇風格傳入對應的 `--style` 參數）：
```bash
# 產生簡報 (例如使用 learning 配色)
uv run scripts/classroom_analyzer_helper.py generate-slides --analysis output/analysis.txt --output output/slides.pptx --style learning

# 產生 FB/IG 分享概念圖
uv run scripts/classroom_analyzer_helper.py generate-image --text output/concept.txt --output output/concept_post.png --style learning
```

## 📄 授權協議

本專案採用 [MIT License](LICENSE) 授權。
