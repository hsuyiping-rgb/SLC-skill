# SLC-skill (學習共同體課例探究與 16:9 簡報生成技能)

這個儲存庫封裝了「學習共同體（Study of Learning Community, SLC）」課堂公開課影片的分析、轉譯、關鍵畫格擷取重繪、以及可編輯文字的 16:9 PowerPoint 簡報排版生成流程。

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
若是公開課私密影片，請先取得登入 Cookie 並匯出為 `cookies.txt`，接著下載影片音訊與轉譯為字幕：
```bash
# 下載影片音訊
yt-dlp --cookies cookies.txt --js-runtimes node --remote-components ejs:github -f "ba" -x --audio-format mp3 -o output/audio.mp3 "https://www.youtube.com/watch?v=VIDEO_ID"

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
執行腳本即可產出排版平衡、字級自動計算且不溢出頁面的 16:9 寬螢幕簡報檔：
```bash
# 產生簡報
uv run scripts/classroom_analyzer_helper.py generate-slides --analysis output/analysis.txt --output output/slides.pptx --style learning

# 產生 FB/IG 分享概念圖
uv run scripts/classroom_analyzer_helper.py generate-image --text output/concept.txt --output output/concept_post.png --style learning
```

## 📄 授權協議

本專案採用 [MIT License](LICENSE) 授權。
