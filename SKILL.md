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

## 📁 最終交付檔案分類

工作完成前，Agent 必須將交付檔依用途整理到下列四類資料夾，並同步修正 HTML、PPTX 或其他檔案中的相對連結：

1. `output/字幕檔/`：SRT、逐字稿與分段辨識結果。
2. `output/簡報/`：PPTX、HTML 與課例分析文字。
3. `output/影片/`：影片與其音訊檔。
4. `output/圖檔/`：社群圖、影片擷取畫面與橫式重繪插圖。

處理用程式、暫存檔不可混入上述交付分類。直式影片插圖必須獨立存放於 `output/drawings_vertical/`，不得混入簡報用圖檔。

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

### 1. 取得風格選擇與影片準備
- **主動列出 `learning`、`pastel`、`blue`、`modern` 四種風格與色彩說明，引導使用者做選擇。**
- 下載 YouTube 可取得的**最高畫質視訊**與最佳相容音訊，合併為 `output/影片/video.mp4`；不得以 360p 或便利格式取代最高可得格式。
- 自行使用 `ffmpeg` 擷取音軌為 `output/影片/audio.mp3`。使用 Whisper medium 或同等品質模型辨識影片中的中文與日文發音；長片可分段辨識後依原始時間碼合併。
- 最少輸出兩份字幕：`subtitles.original.multilingual.srt`（原語，保留中文與日文發音）與 `subtitles.bilingual.zh-TW.srt`（每段原語加上對齊的繁中內容）。同時保留完整逐字稿。**影片、原語字幕與雙語字幕均為最終成品，不得刪除**。

### 2. 詢問 NotebookLM 連接與課例分析架構
- **在開始撰寫課例分析報告前，必須主動詢問使用者：「請問您有沒有要連接 NotebookLM 的筆記來作為簡報分析的架構？如果有，請提供筆記的名稱（及筆記內容）。」**
- 如果使用者提供筆記，應將該筆記的主題結構、核心洞察或特定分析維度，作為簡報分析報告 (`analysis.txt`) 的主要架構，並對齊逐字稿進行「描述 ➔ 詮釋 ➔ 反思」三階層分析。

### 3. 逐頁影格擷取與 GPTimage2 重繪插圖

1. 先取得每一頁簡報的標題、內文與分析焦點；再對照 `output/字幕檔/subtitles.srt` 與 `output/字幕檔/transcript.txt`（或其等效分類後檔名），找出最貼近該頁內容的課堂段落與時間碼。
2. 使用 `ffmpeg` 依段落時間碼擷取原始影格，儲存至 `output/screenshots/slide_{N}.png`。
3. 使用 `generate_image`（GPTimage2）時，**必須**把該頁截圖以 `ImagePaths` 或 `referenced_image_paths` 傳入作為視覺基底；不得只以文字提示生圖。
4. 重繪的橫式圖統一輸出至 `output/drawings/slide_{N}.png`，採一致的**日本溫潤水彩繪本插畫風格**（warm Japanese watercolor picture-book illustration style），供十六比九簡報與網頁使用。
5. 畫面主體必須聚焦學生的學習、傾聽、指著圖表（指圖）與共同推理。完全移除外圍觀課教師；授課教師僅能在直接參與學生學習與小組互動時保留。若使用者提供教師角色截圖，必須將其作為教師出現頁面的視覺參考，並維持一致的外觀與穿著。黑板或展示板只可保留由原始影片影格而來的圖表、照片、示意圖或教材版面，不可產生可讀文字、數字、標籤或水印。人物、肢體、黑板、圖表、教材、桌椅與文具一律完整入鏡，不可在畫面邊緣裁切。
6. 使用者確認第一頁後，第一頁插圖即為**角色連續性母圖**：後續每頁必須同時引用母圖、該頁影片影格及教師角色參考圖，固定教師、學生外觀、座位、桌椅與教室配置；僅依該頁分析重點調整學生的動作、表情、眼神與教師的直接互動。
7. 另依每頁簡報內容產生直式影片插圖，獨立輸出至 `output/drawings_vertical/slide_{N}.png`。構圖為 9:16，最終尺寸必須為 **1080×1920**，並沿用相同的截圖基底、風格、學生聚焦、角色連續性與角色過濾規則。

### 4. 產生簡報與社群圖
- 依據使用者在步驟 1 中選擇的配色主題，執行 `classroom_analyzer_helper.py` 同時產生可編輯文字的 16:9 寬螢幕簡報 (`output/slides.pptx`) 與互動式 HTML 網頁簡報 (`output/slides.html`)。

### 5. 簡報母版、語意與 HTML 規格

1. 先只產生**第一頁 PPTX 預覽**供使用者確認；未獲確認前，不得批量生成其餘插圖或投影片。
2. 確認後，全套 PPTX 固定使用：上方獨立白色標題框、左側白色文字卡、右側 16:9 長方形插圖、淺綠背景、統一邊框與陰影。不得把 16:9 插圖強制拉成正方形或其他變形比例。
3. 文字須依分析架構重新編寫為完整語意的重點段落，**不得以截斷字數方式**留下不完整句子。移除所有錯誤問號或亂碼前綴。
4. 正常目標內文字級為 **26–28 Pt**；應先縮寫與重組句子，再調整文字框高度或插圖高度。文字不得超出文字框、頁面，或被插圖遮蔽。
5. 插圖高度應盡量與相鄰文字卡高度一致；若 16:9 比例與頁面邊界衝突，優先維持插圖比例、文字可讀性與頁面完整，再彈性調整高度與留白。
6. HTML 必須以**最終語意完整 PPTX**為唯一文字來源，使用同一組插圖與標題／文字卡／右側插圖的閱讀邏輯，輸出成一頁式、可向下捲動的網頁；提供固定頁碼導覽，並驗證沒有 `?`、亂碼或失效圖檔連結。

### 6. 完成前檢核

1. 驗證影片解析度為可取得最高格式；字幕原語與雙語檔都有完整時間碼。
2. 驗證每一頁都有對應的影格、橫式插圖，及需要時的直式 1080×1920 插圖。
3. 驗證 PPTX 每頁的文字框未與插圖重疊、文字未超框、插圖比例為 16:9、且人物與物件未被裁切。
4. 驗證 HTML 有與 PPTX 相同頁數、同一組插圖、單頁下捲格式、有效導覽、零問號與零失效圖片。
5. 最後將交付檔放入字幕檔、簡報、影片、圖檔四類資料夾；保留 `drawings_vertical` 作為獨立直式素材資料夾。

---

## Common Mistakes
* **刪除影片或字幕檔案**：工作流結束後不應為了清理而刪除下載的影片 (`output/video.mp4`) 或轉譯的字幕檔 (`output/subtitles.srt`)，這兩者是使用者要求的重要研究成果。
* **未詢問 NotebookLM 筆記**：直接套用預設架構而忽略了使用者已在 NotebookLM 整理好的現成結構，導致產出簡報與其教學研究脈絡不符。
