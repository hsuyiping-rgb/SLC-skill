---
name: slc-skill
description: >-
  學習共同體公開課影片分析與 16:9 自動插圖簡報生成技能。協助使用者下載 YouTube 影片、語音轉譯、進行「描述-詮釋-反思」課例研究分析，並依據影片截圖重繪抹茶綠繪本插圖，最終生成可編輯文字的 16:9 PPTX 簡報與互動式 HTML 網頁簡報。
---

# SLC-skill (學習共同體課例探究與簡報生成技能)

## Overview
本技能旨在提供一整套自動化且標準化的「學習共同體（Study of Learning Community, SLC）」課例研究與簡報排版工作流。引導 Agent 與使用者從課堂錄影出發，經過語音轉譯、課例分析，進而自動擷取關鍵影像，並使用 AI 重繪技術（GPTimage2/`generate_image`）將截圖重繪為高質感的獨立插圖，最終排版生成符合 16:9 比例的簡報檔。

## 🎯 最終產出成品包括：
1. **下載的完整影片** (`output/video.mp4`)
2. **影片的語音轉譯字幕檔** (`output/subtitles.srt` 及其對應逐字稿 `output/transcript.txt`)
3. **簡報投影片（雙格式輸出）**：
   - **PPT 簡報檔** (`output/slides.pptx`)：16:9 寬螢幕比例，文字完全可修正。
   - **HTML 網頁簡報檔** (`output/slides.html`)：基於網頁格式的互動式簡報，支援左右方向鍵與空白鍵切換、自動響應式排版，完美搭配重繪插圖與主題配色。

---

## 📝 課例研究分析與插圖生成規範 (SLC Analysis & Drawing Guidelines)

當執行本自訂技能時，Agent 必須嚴格遵守以下七大分析與生圖規範：

1. **時間戳記標記**：課例研究分析報告必須以影片的**時間碼（如 `12:34`）**精確標記具體的課堂教學事件與學生互動段落。
2. **三階層分析**：分析必須明確區分並落實**「描述－詮釋－反思」**三個分析層次，並在此基礎上呈現於簡報內容中。
3. **聚焦微觀行為**：重點聚焦學生的**語言、眼神、姿態、手勢、沉默**以及**與同伴間的互動**，進行深入的微觀行為分析。
4. **探究多維關係**：深入分析學生與**教材**、學生與**同伴**、以及學生**先前理解（既有認知）**之間的關係。
5. **SLC 哲學概念探討**：探究並凸顯課堂中的**傾聽關係、互惠學習、言談權力、伸展跳躍任務**（挑戰性任務）以及**質性時間（深度學習時刻）**。
6. **去評價、去建言立場**：採取客觀、描述性的**去評價與去建言立場**，聚焦於學生的「學習事實」本身，避免將觀課議課變成對授課教師的優缺點打分或指導。
7. **日系水彩／抹茶綠繪本風格生圖**：
   - 逐頁擷取畫格並使用 `generate_image` (GPTimage2) 進行重繪插圖時，必須指示繪圖引擎將畫面重繪為**一致的「日系水彩／抹茶綠繪本風格」**（Japanese watercolor / Matcha green picture book style）。
   - **畫面主體聚焦**：畫面必須聚焦在**學生的學習、傾聽、指著圖表（指圖）與共同推理**。
   - **角色過濾規則**：**完全移除外圍的觀課教師**；而**授課教師**只有在「直接參與學生學習與小組互動」時才得以保留於重繪畫面中。

---

## Dependencies
- **Python 3.10+** (推薦使用 `uv` 執行環境)
- **FFmpeg** (用於音訊轉檔與影片畫面擷取)
- **python-pptx** (簡報產生工具)
- **Pillow** (社群圖生成與文字繪製工具)
- **yt-dlp** (影片下載工具，建議附帶 Node.js 以便順利執行 JS 解密)

---

## 🎨 簡報色彩主題設定 (Slide Themes)
本技能內建支援四種不同視覺氛圍的簡報配色風格。**Agent 執行時必須主動提供這四個選項讓使用者進行選擇**：
1. **學習共同體哲學風格 (`learning`)**：有機茶白背景、深松針綠標題、溫潤石板灰內文。溫和、防眼部疲勞，營造聆聽對話氛圍。
2. **粉彩柔和教育風 (`pastel`)**：暖白背景、暖深褐標題、柔黑內文。呈現溫暖親和的教學質感。
3. **科技商務藍風 (`blue`)**：淡藍灰背景、深海軍藍標題、藍灰內文。專業、清晰、邏輯分明。
4. **現代極簡黑白風 (`modern`)**：極淡灰背景、曜石黑標題、中灰內文。極簡、俐落。

---

## Quick Start

1. **下載影片**（音訊與視訊一併下載）：
   ```bash
   yt-dlp --cookies cookies.txt --js-runtimes node --remote-components ejs:github -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]" -o output/video.mp4 "https://www.youtube.com/watch?v=VIDEO_ID"
   ```
2. **擷取音訊並轉譯（使用 Whisper）**：
   ```bash
   # 從影片中分離音訊
   ffmpeg -i output/video.mp4 -q:a 0 -map a output/audio.mp3
   # 語音轉譯
   whisper output/audio.mp3 --model medium --language zh -o output/ -f srt
   ```
3. **執行雙格式簡報與概念圖生成**（依據使用者所選風格指定 `--style`）：
   ```bash
   # 此指令會自動在 output 目錄下同時產生 slides.pptx 以及 slides.html！
   uv run scripts/classroom_analyzer_helper.py generate-slides --analysis output/analysis.txt --output output/slides.pptx --style learning
   
   uv run scripts/classroom_analyzer_helper.py generate-image --text output/concept.txt --output output/concept_post.png --style learning
   ```

---

## Utility Scripts (scripts/classroom_analyzer_helper.py)

本技能附帶了一個排版與生圖輔助腳本，具備以下子命令：

### 1. `generate-slides`
分析提供的課例研究文本，自動生成 16:9 的 PowerPoint 簡報與同名 HTML 網頁簡報：
* **參數**：
  * `--analysis`: 課例分析報告文字檔路徑。
  * `--output`: 輸出的 `.pptx` 檔案路徑（同名 `.html` 檔案會自動於同目錄下生成）。
  * `--style`: 簡報風格，支援 `learning`, `pastel`, `blue`, `modern` 四種選擇。
* **排版與儲存規則**：
  * **版面尺寸**：16:9 寬螢幕比例（寬 13.333 英吋，高 7.5 英吋）。
  * **動態字級與避讓**：標題單行原則（32-40 Pt）、內文不超出原則（20-24 Pt）。若偵測到插圖，文字框與圖片會自動左右對稱排版。
  * **鎖定安全機制**：若 `slides.pptx` 已被 PowerPoint 開啟而無法寫入，腳本會自動將其與 HTML 簡報儲存為 `*_copy.pptx` 與 `*_copy.html`，保障分析流程不中斷。

### 2. `generate-image`
利用 Pillow 自動將核心概念文案轉換成 FB/IG 分享圖片：
* **參數**：
  * `--text`: 概念文案檔案路徑。
  * `--output`: 輸出的 `.png` 檔案路徑。
  * `--style`: 圖片背景與文字配色，支援與簡報相同的四種風格。

---

## Workflow (Agent 執行指引)

當使用者請求分析公開課影片時，請遵循以下步驟：

### 1. 取得風格與段落選擇、影片準備
- **主動列出 `learning`、`pastel`、`blue`、`modern` 四種風格與色彩說明，引導使用者做選擇。**
- **段落分析與下載詢問**：在開始下載或分析影片前，必須先評估影片長度，**詢問使用者是否要擷取下載並輸出特定的影片段落（例如：引起動機、分組討論、發表分享等）**，還是要下載分析整段影片。
- 依據選擇，下載對應的影片檔案 (`output/video.mp4`)。如果是私密影片，需引導使用者手動匯出 `cookies.txt` 以供 `yt-dlp` 使用。
- 自行使用 `ffmpeg` 擷取音軌為 `output/audio.mp3`，並使用 Whisper 轉譯為 `output/subtitles.srt` 與完整逐字稿 `output/transcript.txt`。**確保這兩個檔案（影片與字幕）均保留為最終輸出成品**。

### 2. 詢問 NotebookLM 連接與課例分析架構
- **在開始撰寫課例分析報告前，必須主動詢問使用者：「請問您有沒有要連接 NotebookLM 的筆記來作為簡報分析的架構？如果有，請提供筆記的名稱（及筆記內容）。」**
- 如果使用者提供筆記，應將該筆記的主題結構、核心洞察或特定分析維度，作為簡報分析報告 (`analysis.txt`) 的主要架構，並對齊逐字稿進行「描述 ➔ 詮釋 ➔ 反思」三階層分析。

### 3. 逐字稿精確擷圖與二次確認重繪
- **段落語境對齊擷圖**：依據簡報中每一頁投影片的主題，對照 `output/subtitles.srt` 的時間戳記與逐字稿的對話段落，找出最能反映該頁簡報核心內容的精確時間點，使用 `ffmpeg` 截取影片畫面存成 `screenshot_{N}.png`。
- **擷圖二次確認互動**：在執行生圖重繪前，**必須在對話中以圖片形式展示擷取出來的原始影片畫面截圖給使用者確認**，並詢問：*「校長，這是為您擷取的公開課關鍵畫格截圖（畫面時間點：XX:XX），您是否滿意？如果不滿意，請告訴我需要調整的時間點，我將重新為您擷圖。」*。
- **重繪插圖**：唯有在使用者確認滿意後，才能使用 `generate_image` (GPTimage2) 生圖工具，將確認後的畫面作為底圖，重繪為高質感的獨立插圖並儲存至 `output/images/slide_{N}.png`。若使用者不滿意，則重新擷取並重複確認。

### 4. 產生簡報與社群圖
- **內容頁數規劃**：規劃簡報的內容頁數在大約 **15 頁至 20 頁** 之間（依據分析影片長短與深度動態調整）。
- 依據使用者在步驟 1 中選擇的配色主題，執行 `classroom_analyzer_helper.py` 同時產生可編輯文字的 16:9 寬螢幕簡報 (`output/slides.pptx`) 與互動式 HTML 網頁簡報 (`output/slides.html`)。

---

## Common Mistakes
* **刪除影片或字幕檔案**：工作流結束後不應為了清理而刪除下載的影片 (`output/video.mp4`) 或轉譯的字幕檔 (`output/subtitles.srt`)，這兩者是使用者要求的重要研究成果。
* **未詢問 NotebookLM 筆記**：直接套用預設架構而忽略了使用者已在 NotebookLM 整理好的現成結構，導致產出簡報與其教學研究脈絡不符。
* **未提供擷圖供使用者確認**：跳過擷圖確認步驟而直接重繪插圖，導致生成圖不符合使用者預期的教學事件畫面。
* **頁數太少**：生成的內容頁數少於 15 頁，未滿足 15-20 頁的課例研究深度要求。
