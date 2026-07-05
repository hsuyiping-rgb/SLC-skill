---
name: slc-skill
description: >-
  學習共同體公開課影片分析與 16:9 自動插圖簡報生成技能。協助使用者下載 YouTube 影片、語音轉譯、進行「描述-詮釋-反思」課例研究分析，並依據影片截圖重繪抹茶綠繪本插圖，最終生成可編輯文字的 16:9 PPTX 簡報與社群分享圖。
---

# SLC-skill (學習共同體課例探究與簡報生成技能)

## Overview
本技能旨在提供一整套自動化且標準化的「學習共同體（Study of Learning Community, SLC）」課例研究與簡報排版工作流。能夠引導 Agent 與使用者從課堂錄影出發，經過語音轉譯、課例分析，進而自動擷取關鍵影像，並使用 AI 重繪技術（GPTimage2/`generate_image`）繪製溫潤手繪水彩風格的插圖，最終排版生成符合 16:9 比例的簡報檔及社群分享圖。

## Dependencies
- **Python 3.10+** (推薦使用 `uv` 執行環境)
- **FFmpeg** (用於音訊轉檔與影片畫面擷取)
- **python-pptx** (簡報產生工具)
- **Pillow** (社群圖生成與文字繪製工具)
- **yt-dlp** (影片下載工具，建議附帶 Node.js 以便順利執行 JS 解密)

---

## Quick Start

1. **下載影片音訊**：
   ```bash
   yt-dlp --cookies cookies.txt --js-runtimes node --remote-components ejs:github -f "ba" -x --audio-format mp3 -o output/audio.mp3 "https://www.youtube.com/watch?v=VIDEO_ID"
   ```
2. **語音轉譯（使用 Whisper）**：
   ```bash
   whisper output/audio.mp3 --model medium --language zh -o output/ -f srt
   ```
3. **執行簡報與概念圖生成**：
   ```bash
   uv run scripts/classroom_analyzer_helper.py generate-slides --analysis output/analysis.txt --output output/slides.pptx --style learning
   uv run scripts/classroom_analyzer_helper.py generate-image --text output/concept.txt --output output/concept_post.png --style learning
   ```

---

## Utility Scripts (scripts/classroom_analyzer_helper.py)

本技能附帶了一個排版與生圖輔助腳本，具備以下子命令：

### 1. `generate-slides`
分析提供的課例研究文本，自動生成 16:9 的 PowerPoint 簡報：
* **參數**：
  * `--analysis`: 課例分析報告文字檔路徑。
  * `--output`: 輸出的 `.pptx` 檔案路徑。
  * `--style`: 簡報風格，支援 `learning` (抹茶綠/學習共同體風), `pastel` (暖白粉彩), `blue` (科技藍), `modern` (極簡)。
* **版面規則**：
  * **動態標題**：15字內採用 40 Pt，20字內 36 Pt，否則 32 Pt，維持單行呈現。
  * **動態內文**：根據項目密度自動在 20 Pt - 24 Pt 之間微調，決不超出頁面。
  * **插圖排版**：程式會自動尋找 `output/images/slide_{N}.png` 檔案，若存在則將文字寬度自動縮小為 6.5 英吋，並在右側插入 4.2 x 4.2 英吋的精美插圖。

### 2. `generate-image`
利用 Pillow 自動將核心概念文案轉換成 FB/IG 分享圖片：
* **參數**：
  * `--text`: 概念文案檔案路徑。
  * `--output`: 輸出的 `.png` 檔案路徑。
  * `--style`: 圖片背景與文字配色，支援與簡報相同的四種風格。

---

## Workflow (Agent 執行指引)

當使用者請求分析公開課影片並生成簡報時，請遵循以下步驟：

### 1. 音訊與轉譯準備
- 下載 YouTube 影片的音訊，並轉譯為繁體中文 SRT 字幕與逐字稿。如果是私密影片，需引導使用者手動匯出 `cookies.txt` 以供 `yt-dlp` 使用。

### 2. 課例探究分析 (Describe ➔ Interpret ➔ Reflect)
- 依據逐字稿，撰寫深度分析報告。報告中必須涵蓋：
  - **描述 (Describe)**：記錄客觀發生的教學事件（如：某學生小聲說了什麼、某學生的眼神動作）。
  - **詮釋 (Interpret)**：分析事件背後的學習意義與聆聽連結（如：此發言如何串聯前一位同學的想法，促成認知跳躍）。
  - **反思 (Reflect)**：提出教學設計的修正建議，並探查對整體教育哲學的啟示。

### 3. 擷取影片畫面與 GPTimage2 生圖重繪
- 依據簡報中每一頁投影片的主題，使用 `ffmpeg` 在對應的時間點截取影片畫面存成 `screenshot_{N}.png`。
- 使用 `generate_image` (GPTimage2) 生圖工具，將擷取的畫面作為底圖 (`ImagePaths`)，輸入對應的主題 prompt（指定「抹茶綠色系、日系溫潤水彩繪本風格」），將截圖重繪為高質感的獨立插圖並儲存至 `output/images/slide_{N}.png`。

### 4. 產生簡報與社群圖
- 執行 `classroom_analyzer_helper.py` 產生可編輯文字的 16:9 寬螢幕簡報。
- 提取簡報核心金句，產生 FB/IG 社群概念圖卡。

---

## Common Mistakes
* **文字與插圖重疊**：在手動調整排版時，未將文字框寬度限縮，導致與右側插圖重疊。本技能的輔助腳本會自動處理此項避讓，請務必遵循其寬高設計。
* **生圖額度超限 (429)**：在短時間內大量呼叫 `generate_image` 易觸發限流。Agent 應採用分批（每次 3-4 張）的方式生成，或在額度用罄時先使用真實影片截圖作為占位圖，並設定定時任務（Cron）於額度恢復後重試更新。
