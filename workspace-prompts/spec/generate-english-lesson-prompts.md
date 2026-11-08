# 英文學習課程生成 Prompts

本文件由使用者指定的 `house\converted` 下單篇雙語 Markdown 產生英文學習課程，
輸出至 `house\lesson\[課程檔名]`。以繁體中文協助學習者理解來源內容、學習可遷移的英文，
並建立與來源領域相符的雙語知識與術語；無須讀取其他課程、規範或範本。

## 本 workspace 的流程

```text
house\source\[課程檔名].md
→ house\converted\[課程檔名].md
→ house\lesson\[課程檔名]\
```

- 來源轉換是獨立的前置階段，依 `workspace-prompts\spec\source-conversion.md` 執行。
- 本階段只讀取指定 converted 檔案，不讀取或修改 `house\source`，也不回寫 converted。
- 沿用 converted 的正文章節、中英配對及順序，不把兩種語言當成兩份正文重複處理。
- 本文既有契約中的 `raw`、`raw_path` 與 `raw_sha256` 均指指定的 converted 檔案及其
  SHA-256，僅保留作為內部資料欄位名稱，不代表另有 `raw` 目錄或需要讀取原始來源。
- 保留既有八檔課程、17 個 H2、HTML、TTS 與驗證契約。converted 採中文在前英文在後；
  課程 Reader 則沿用本文件的英文在前中文在後，兩者使用相同配對，不重新翻譯。

本規格不把每份來源硬套成一般閱讀測驗或完整文法教科書。課程有四個互相配合的主軸；其中
文法教學須由來源中確實值得學習的表達觸發，不為湊格式而生成：

1. **Communication English**：整理來源情境中可重複使用的溝通功能、句型與自然表達。
2. **Domain Knowledge**：理解來源所屬的人文、社會、自然、教育、生活、專業或其他知識脈絡。
3. **Bilingual Terminology**：建立可搜尋、可分類及可朗讀的來源領域中英文術語學習頁。
4. **Source-grounded Grammar**：辨識來源中有價值的文法、搭配與句構；原句不正確或不自然時，
   依可確認的原意提供教學修正版，說明修改原因，但不改動 raw。

所有 lesson 生成工作都將使用者指定的單一 raw 視為唯讀事實來源，不得修改、拆分、合併、重新命名或建立替代 raw，也不得讀取同一 raw 目錄中的其他正文或伴隨檔，除非使用者明確指定。若來源是 Email Content，raw 的 headers、正文、空白、換行、quoted history、簽名、URL、順序與格式均須完整保留；清理、翻譯、摘要、自然化改寫及教學註解只能寫入 lesson。

## 獨立執行契約與規範優先序

本文件本身包含產生一份單篇課程所需的完整內容、DOM、播放器、索引與驗證契約。獨立執行
時只需要指定 raw 路徑；不得要求讀取其他課程、範本或 prompt 才能開始。

若 workspace 另有專用規格，依下列優先序處理：

1. 使用者本次明示且不違反來源保真或資料保護的要求。
2. 本文件「本 workspace 的流程」與「輸入、輸出與責任邊界」。
3. 本文件的共用課程契約。
4. 同名既有 lesson 中仍符合前三項的人工內容與導覽。

低優先序不得默默覆蓋高優先序。遇到真正衝突時，保留來源與既有正確內容，停止衝突部分並
明列條款；不得自行挑選較容易實作的版本。本文後段若提供「內容骨架」，播放器控制列一律
以「TTS 與控制器鎖定契約」中的完整 DOM 為唯一準據，內容骨架不得重新定義控制列。

### 支援的執行模式

| 模式 | 來源 | 允許修改 | 必要結果 |
| --- | --- | --- | --- |
| `create` | 指定 raw，沒有同名 lesson | 同名 lesson 與必要 index | 全新八檔課程及驗證 |
| `update` | 指定 raw 與同名 lesson | 同名 lesson 與必要 index | 保留正確人工內容並修正缺漏 |
| `verify-and-fix` | 指定 raw 與同名 lesson | 僅可安全確認的同名 lesson 問題 | 問題、修正與重驗結果 |

- 一次只處理一個 converted 檔案，在專屬課程子目錄輸出八檔，不拆成多集課程。
- 不論模式，converted 都是唯讀。若尚未完成來源轉換，先依 `source-conversion.md`
  完成獨立前置階段，再指定 converted 路徑執行本文件；兩個階段不得交錯。
- 未指定輸入時先請使用者提供精確路徑。未指定模式時，只有輸出目錄不存在才使用 `create`；
  若已存在，先確認更新授權，不靜默覆寫人工內容。
- 使用者只要求「更新 index」時，只能在所有目標頁實際存在且連結驗證通過後新增入口，
  不得先建立會失效的索引連結。

## Canonical 課程資料模型

產生任何 Markdown 或 HTML 前，先建立單一 canonical 內容資料。可使用記憶體物件或
session 暫存檔，但不得提交中介資料。至少包含：

```text
source:
  raw_path, raw_sha256, title, source_type, source_url, metadata,
  primary_content_boundaries, uncertainty_markers
course:
  display_title, inferred_level, overview, goals, summary,
  domain_notes, communication_functions, sentence_patterns,
  natural_english, practice, application, review, answer_key
reader_sections[]:
  id, title, ordered_pairs[]
ordered_pairs[]:
  pair_id, section_id, section_pair_number, source_span, en, zh
terms[]:
  term, domain, part_of_speech, zh, plain_english, definition_zh,
  source_meaning, relationship, example, example_zh, example_origin, evidence
vocabulary[]:
  term, section, part_of_speech, meaning, zh, pronunciation,
  collocation, example, example_zh, memory_tip, lesson_meaning_zh
grammar[]:
  zh_name, en_name, source_example, rule, source_analysis,
  quick_check_a, quick_check_b, answer, answer_reason,
  teaching_rewrite, meaning_tone_boundary
teacher_points[]:
  id, title, zh_text, en_text
content_summary_points[]:
  id, title, zh_text, en_text, source_spans, editorial_notes (optional)
```

Canonical 資料規則：

- `source_span` 必須能回溯到來源中的連續內容；不得只有翻譯而沒有英文來源。
- Reader 的英文 token 依序聯集必須覆蓋全部 primary source content；允許的差異只有本文件
  明定的字幕硬換行合併及單元開頭 `>>`／`»` 顯示移除。
- `section_pair_number` 在每個 section 從 1 連續編號；`pair_id` 在輸出範圍內唯一且穩定。
- Markdown Reader、主要 HTML Reader、`bilingual.html` 只能投影同一份 `ordered_pairs`，
  不得各自重新分句、翻譯或編號。
- `ordered_pairs` 直接取自 converted 正文中的既有中英配對；來源資訊、轉換註記及其
  硬換行控制符不混入朗讀文字。中文原文的英文翻譯須標為轉換譯文，而非原始英文引文；
  英文原文的既有英文才可稱為來源英文。混合來源無法確認個別原語時，不聲稱是逐字引文。
- 術語、字彙、文法在 Markdown、主要 HTML 與專用頁的數量、順序及核心欄位完全一致。
- 術語的 `plain_english` 與 `definition_zh` 說明一般概念；`source_meaning` 只說明本課中的
  用法、場景或指涉。`part_of_speech` 與 `evidence` 和 domain 一起顯示為卡片小字資訊，
  不新增朗讀單元。一般定義不得直接複製來源體驗；新增背景不得冒充來源明示。
- 術語的 `example_origin` 記錄例句是來源原句或教學新增示例；來源原句須可回溯 raw，
  教學新增示例不得冒充來源逐字引文。此內部欄位不新增術語卡欄位或朗讀單元。
- `teacher_points` 僅承載補充英文教學；`content_summary_points` 承載本文摘要、重點與
  導讀，須以 `source_spans` 對應來源，並與 `course.summary`、內容筆記及 Reader 一致。
  兩者不得混用或將同一份講稿複製到兩頁。`editorial_notes` 僅存放不供讀者朗讀的編輯提醒，
  不混入 `zh_text`、`en_text` 或 `data-text`；輸出時置於對應主題的 HTML 註解。
- 翻譯須忠實保留人物歸屬、數字、否定、情態、時態、條件、語氣與不確定性；不得用摘要
  取代，也不得把一段中文複製給多個不同英文 pair。

## 工作狀態機與完成門檻

依序執行，前一階段未通過不得把後一階段標成完成：

1. **Scope locked**：解析唯一 raw、輸出目錄、允許 index、既有變更與模式。
2. **Source frozen**：完整讀取 raw，記錄 SHA-256、結構、primary content 與限制。
3. **Canonical ready**：建立並驗證 Reader、術語、字彙、文法、英文教學及本文導讀資料。
4. **Artifacts generated**：由 canonical 資料一次產生全部八檔，不手動分別維護重複內容。
5. **Static valid**：結構、數量、順序、DOM、連結、`data-text`、編碼及腳本均通過。
6. **Browser valid**：指定 viewport、鍵盤、響應式、搜尋、播放器與 TTS stub 均通過。
7. **Native smoke tested**：Microsoft Edge 原生 Web Speech 測試通過；若環境不可用，狀態只能
   記為未驗證，不得併入「全部通過」。
8. **Integrity closed**：raw hash 未變、限定 diff 無越界修改、暫存檔已清理、Git staging
   與 commit 未改動。

失敗後須修正根因並從最早受影響階段重跑。例如 Reader 切段改變時，必須重建 Markdown、
主要 HTML 與 Bilingual Reader，不能只修其中一頁；播放器修正時，須同步重建使用同一殼層
的所有頁並重跑共享控制器測試。

## 輸入、輸出與責任邊界

- 輸入是使用者指定的 `house\converted\[課程檔名].md`，須已有完整中英正文配對與來源
  資訊；若配對缺漏或格式無法可靠辨識，明確回報並停止受影響步驟，不在課程階段補做轉換。
  若上游提供 converted 的 SHA-256，開始與完成時均須核對；否則自行計算並記錄。
- 本流程不清理或修正 raw。若 raw 含無法安全用於課程的結構性問題，須明確回報，不得在
  lesson 生成期間改寫 raw 或悄悄改用其他來源。
- 輸出固定放在 `house\lesson\[課程檔名]`，包含本文件規定的一份課程 Markdown、七份
  HTML，以及課程目錄外必要的 `house\lesson\index.html` 建立或更新。
  不建立本文件未定義的中介規格、替代 raw 或額外課程頁。
- 本文件只依實際指定的 converted 與本文件內的完整契約生成並驗證課程，不要求讀取
  source 或轉換規格才能開始，也不依來源語言改變課程頁型或播放器。

## 工具契約與執行順序

本流程只使用本機工具處理 raw 與 lesson。工具名稱可依環境替換，但必須具備下列能力並依
固定順序執行；不得把 raw、lesson、diff、螢幕內容或測試資料上傳至外部服務。
公開通用術語的一般背景查核依 `Key Terminology` 規則辦理；只查必要的公開資料，不傳送
來源正文或其他受保護內容，也不以外部資料改寫 raw 或補造來源資訊。

必備能力：

1. **路徑、清單與工作樹檢查**：解析指定 raw、同名 lesson、允許讀取的 index 及輸出路徑；
   開始與完成時記錄工作樹狀態，以區分本次修改與既有變更。清單工具只列出索引及連結驗證
   所需的目錄／檔名，不得讀取其他 raw 正文。
2. **UTF-8 安全讀寫**：完整讀取指定 raw 及允許範圍內的既有檔案，並以明示 UTF-8 產生
   Markdown、HTML、CSS 與 JavaScript；不得經系統 ANSI code page 解碼後覆寫。
3. **雜湊與限定 diff**：開始與結束時以 SHA-256 驗證 raw 位元組未變；對本次輸出產生
   限定範圍的 diff，檢查人工修改、非預期刪除、舊檔遷移、index 變更及範圍外修改。
4. **結構化解析與靜態驗證**：使用能解析 Markdown 與 HTML DOM 的工具，不以正規表示式
   取代巢狀結構解析。依「完成前驗證」計算標題、section、ID、class、控制項、朗讀單元、
   語言順序、pair、卡片、連結及 `data-text`／可見文字一致性；另執行 HTML、CSS、
   JavaScript 及可用的檔案診斷。
5. **瀏覽器與可存取性測試**：在 320px、390px 與一般桌面 viewport 開啟實際輸出，
   檢查版面、水平溢出、sticky 控制列、焦點、Tab、Enter／Space、連結及互動。一般測試可用
   支援標準 DOM 與 Web Speech API 的本機瀏覽器或自動化工具；本文件明定的 Edge 相容性
   檢查必須使用 Microsoft Edge，不得以其他瀏覽器通過後宣稱 Edge 已驗證。
6. **TTS 控制器測試**：以可觀察 `speechSynthesis` 呼叫與事件的本機測試方式驗證 utterance
   順序、語言、模式、選聲、語速、暫停／繼續、取消、自然接續、首尾 clamp、狀態及錯誤。
   可使用不改動成品的暫時性 stub 進行可重現的狀態測試，但仍須在 Microsoft Edge 使用原生
   Web Speech API 完成播放、暫停／繼續、停止、選聲與語速的 smoke test；測試 stub 不得寫入
   最終 HTML。
7. **編碼、格式與清理檢查**：以 UTF-8 重讀八個輸出，掃描非預期的 `U+FFFD`、
   private-use 字元及連續三個以上 `?`，檢查 HTML 原始碼的縮排、換行及非壓縮結構，執行
   相當於 `git diff --check` 的格式檢查，並移除一次性腳本、截圖、測試報告與暫存檔。

執行順序固定為：解析唯一 raw 與允許範圍 → 記錄初始狀態及 raw SHA-256 → 完整讀取 raw
與同名既有 lesson → 建立 canonical 內容資料（包括 Reader pair、字彙、文法與術語清單）
→ 由同一份資料同步產生或更新八檔及必要 index → 執行靜態與跨檔一致性驗證 → 執行瀏覽器、
鍵盤、響應式與 TTS 測試 → UTF-8 重讀、診斷、限定 diff 與格式檢查 → 重算 raw SHA-256
及檢查最終工作樹 → 清理一次性檔案並回報。

工具缺失或測試失敗時，可改用具相同能力的本機工具重試；不得降低契約、靜默跳過或以
「檔案可開啟」代替指定驗證。若 Microsoft Edge、原生 Web Speech API、DOM 解析器或其他
必要能力確實不可用，須完成仍可執行的安全檢查，明列未驗證項目與原因，且不得宣稱
「完成前驗證」全部通過。

## UI 與課程格式基線

本文件是來源處理、八檔輸出、17 個 H2、學習情境、Email 不可變契約、事實
邊界、UI、DOM、CSS、Web Speech API、TTS、響應式行為及索引格式的唯一規格。不得要求
讀取其他課程或目錄中的檔案作為基準。

本文件中的「參考設計」、「參考頁」及「參考控制器」都只指本文件明列的 DOM、CSS 數值、
控制 ID、順序與行為，不指向任何外部檔案。不得為理解或補齊這些契約而讀取其他 prompt、
規格、範例課程或網路頁面。

更新既有 lesson 時，可讀取同名 lesson 內的目標檔案，以保留正確內容、人工修改及有效
導覽；建立新 lesson 時，直接依本文件的頁型與行為契約實作，不得因缺少外部參考檔案
而停止。任何既有檔案都只能提供該課本身的內容與版面資訊，不得覆蓋本文件規格，也不得
複製其他課程的人物、單字、例句、題目、答案或文法內容。

### 七種 HTML 頁型與索引契約

| 頁型 | 必須保留的鎖定殼層／內容單元 |
| --- | --- |
| 主要課程 | 單一 `main`、頂部返回 index、一個 H1、H1 後的雙語 `.ai-content-notice`、17 個 H2 section、`.table-wrap` |
| Bilingual Reader | body 直屬 `.controls > .row`、`main`、`.bilingual-section[data-group-index]`、`.bilingual-pair[data-group-index][data-pair-id]`、`.speech-unit` |
| 英文教學 | body 直屬 `.controls > .row`、`main`、`.teaching-point[data-point-id]`、每點先繁中後英文的兩個 `.speech-unit` |
| 本文導讀 | 與英文教學共用殼層及 `.teaching-point[data-point-id]` 結構，內容只取自 `content_summary_points` |
| 中文單字老師 | body 直屬淺綠 `.controls > .row`、`header`、最大 1000px 的 `main`、`.vocabulary-item[data-vocab-id][data-section][data-term]`、每項固定四個 `.speech-unit` |
| 文法課程 | body 直屬淺綠 `.controls > .row`、單一 `main`、導讀與總結 `.speech-unit`、`.grammar-point[data-grammar-id]`、每點固定五個 `.speech-unit`（首單元含 H2 標題與規則，另含快速檢核與答案理由） |
| 整合術語頁 | body 直屬 `.controls > .row`、`main.terminology-main`、`.tools`、`.term[data-term][data-domain]`、`.term-field[data-field]`、欄位級 `.speech-unit`，以及搜尋與 domain 篩選 |
| 索引 | `main.page`、資訊 header、`.lesson-list`、每課一個 `article`、`.links` 內恰有七個課程連結 |

建立或重建頁面時，依本文件統一 `<style>`、通用 JavaScript、語意層級、class、控制項
ID、按鈕順序與文字，只依課程替換標題、教材內容、項目數及相對路徑，不得建立規格外
版型。整合術語頁使用七鍵 compact player、中文單字老師的卡片視覺，並加入
本文件規定的搜尋、篩選與欄位分段。

### 跨課程鎖定殼層與內容替換白名單

- 原始來源的主題、長度、格式、語言比例、所在 workspace、既有頁面或生成工具都不得改變
  本文件的播放器與 UI。不得因內容較長、來源是旅遊逐字稿、既有 lesson 有另一套版面，
  或執行環境缺少範例課程，而簡化、重排、重新命名或自行設計控制列。
- 同一頁型必須由同一份 renderer template 產生。不同課程間只允許替換以下內容：
  `<title>`、H1、meta、canonical 教材文字、內容項目／群組數、穩定資料屬性值及經驗證的
  相對導覽路徑。播放器 DOM、控制文字、控制順序、ARIA、CSS selector 與 declaration、
  voice scoring、事件綁定、狀態文字及 Web Speech 控制流程不在替換白名單內。
- 完整播放器殼層固定供 `bilingual.html`、`teacher.html` 與 `content-summary.html` 使用；compact 播放器殼層固定
  供 `vocabulary.html`、`grammar.html` 與 `terminology.html` 使用。不得在任何課程中把
  完整與 compact 控制項混搭，也不得從既有同名 lesson 複製不符合本文件的播放器。
- 生成前先在記憶體建立鎖定的 `full-player-shell`、`compact-player-shell`、
  `full-player-css`、`compact-player-css`、`full-player-controller` 與
  `compact-player-controller`；所有適用頁面只能引用同一份字串或模板片段，不得逐頁
  重新撰寫。術語頁的搜尋與篩選使用獨立腳本，但不得改寫 compact 控制器。
- 產生後須以 DOM 正規化比較鎖定殼層，而不是以目視「相似」判定。正規化只可忽略 HTML
  縮排與屬性引號形式，不得忽略元素層級、class、ID、屬性、文字、option、控制順序或
  ARIA。CSS 比較只可忽略空白與註解；selector、property 及 value 必須相同。JavaScript
  比較只可忽略格式空白；識別符、字串、條件、事件與執行順序必須相同。
- 任一鎖定片段不一致即為生成失敗，必須回到共享 renderer 修正並重新產生所有受影響頁面；
  不得在單一 HTML 追加覆寫 CSS、頁面專屬事件處理器或補丁腳本來掩蓋差異。

### 頁面骨架與導覽契約

- 七份課程 HTML 的 `<title>` 與 H1 應包含來源標題；除文法頁固定以
  `中文文法老師 | [課程標題]` 開頭外，各功能頁以來源標題開頭並加入一致的頁型名稱：
  `Bilingual Reader／中英對照朗讀`、`英文教學`、`本文導讀`、`中文單字老師`、`文法與句構課程` 或
  `Vocabulary and Expressions`。術語頁 H1 固定為
  `[課程標題] — [來源內容類型] Vocabulary and Expressions`；`[來源內容類型]` 使用
  workspace 規格明定的類型（例如 `Drama` 或 `Travel`），若未明定則省略該欄及其後空格，
  不得自行猜測。不得使用同一個無法辨識頁型的標題。
- `lesson.html` 的單一 main 依序包含頂部返回課程索引連結、H1、雙語
  `.ai-content-notice`，以及依固定順序排列的 17 個 `<section>`；每個 section 以對應 H2
  開頭。通知只出現一次，明示教材由 AI 協助整理並以指定原始來源為準，不得遮擋或取代
  課程內容。不得把返回索引連結移到正文結尾或把 H2 直接散放在 main。
- `bilingual.html`、`teacher.html`、`content-summary.html` 與 `terminology.html` 的 H1 後放一段精簡 `.meta`，
  分別說明朗讀順序、英文補充教學範圍、本文導讀範圍或術語數量與來源；教學與導讀頁均
  說明先繁中後英文的朗讀順序。頁尾依序提供「返回主要課程」與
  「返回課程索引」。
- `vocabulary.html` 在控制列後依序使用無額外 class 的 `header` 與 `main`；H1 及
  「返回主要課程／返回課程索引」位於 header，所有單字卡片只放在 main。
  `grammar.html` 不建立 header：H1、meta、導讀、文法點、總結及頁尾返回導覽全部依序位於
  單一 main。不得把導覽文字放進任何朗讀單元。
- 所有內部連結使用正確相對路徑；返回連結、頁面說明、pair number、卡片 meta、篩選工具
  及狀態文字都不加入 TTS。

### 主要課程 Bilingual Reader 視覺契約

- Markdown 最後的 Bilingual Reader 必須在主要 HTML 明確解析為英中配對，不得把整課
  英中內容壓在單一 `<p>`，也不得只靠 Markdown 相鄰換行期待瀏覽器自動分組。每個 H3
  建立一個 `.lesson-bilingual-section[data-section-index]`；每組使用
  `.lesson-bilingual-pair[data-pair-index]` `<article>`。
- 每個 `.lesson-bilingual-pair` 內含且只含兩個 `.lesson-language-block`：先
  `.lesson-english` 的 `<p lang="en">`，再 `.lesson-chinese` 的
  `<p lang="zh-Hant">`。兩種語言不得混在同一個 `<p>`。
- Markdown Reader、主要 HTML 閱讀區及獨立 `bilingual.html` 必須由同一份已排序的 canonical
  pair list 產生，不得各自重新分句或翻譯。每個 pair 的英文、繁中、小節索引及小節內編號
  都是同一筆資料；修改切段後須一次重建三個表面，禁止只修其中一頁。
- 語言區塊分別顯示不進入正文的 `English` 與 `繁體中文` 標籤。英文使用淡藍底與藍色
  左框，繁中使用暖米色底與金色左框；不得使用固定高度。
- 桌面使用兩個等寬 `minmax(0,1fr)` 欄位，英文在左、繁中在右；660px 以下固定改為
  英文在上、繁中在下。主頁閱讀區不得加入 TTS 控制器、`.speech-unit`、`data-lang`
  或 `data-text`；朗讀功能只屬獨立 Bilingual Reader 頁。
- 若 Markdown 有 M 個小節與 N 組配對，主要 HTML 必須恰有 M 個
  `.lesson-bilingual-section`、N 個 `.lesson-bilingual-pair`、N 個英文區塊及 N 個
  中文區塊，文字與順序逐組一致。重新生成小節編號時，先移除既有編號再依目前順序加一次。

### 視覺與響應式契約

- 各頁使用 `17px/1.7 "Noto Sans TC","Segoe UI",sans-serif`、`#20302d` 文字、`#f7f7f2` 背景、
  `#d6dfdc` 框線及最大 1000px 置中內容。一般卡片白底、8px 圓角；一般控制列與表頭
  使用 `#e9f3f0` 淡綠底。
- 共用正文排版不得依賴瀏覽器表單預設值：body 的 `font-size`／`line-height`／`font-weight`
  固定為 `17px`／`1.7`／`400`；H1 為 `2em`／`1.25`／`700`，H2 為
  `1.5em`／`1.35`／`700`，H3 為 `1.17em`／`1.7`／`700`，一般 `p`、`ul`、`ol`
  與 `li` 為 `1em`／`1.7`／`400`，全部繼承 body 的 font family。660px 以下只把 body
  字級改為 `16px`，因此標題 em 值隨之縮放；320px、390px 不建立另一套正文或標題字級。
  單字頁 header H1、卡片 H2、meta、label 或工具文字若有頁型專用數值，以該明示值覆寫。
- `button`、`select`、`input`、`output` 與可見 label 一律繼承 body font family、font size
  與 line height；播放器按鈕與欄位使用本節後方鎖定的高度及 padding，不再套一般 44px
  索引連結尺寸。一般按鈕及 select 為白底、`#20302d` 文字與 `#91aaa4` 框線；hover 將框線
  改為 `#244f47`，disabled 使用 `opacity:.55` 且游標不得暗示可操作。所有 anchor、
  button、select 及 input 的 `:focus-visible` 固定使用 `3px solid #b96800` 與 `2px`
  outline offset；filter 的 pressed 狀態、active speech unit 及錯誤文字仍須另有非色彩
  的文字、ARIA 或結構狀態。
- 所有課程頁共用下列垂直節奏，不得由各課自行放大：`main` 使用
  `margin:1rem auto 3rem`；H1 使用 `margin:0 0 .75rem`；H2 使用
  `margin:.6rem 0 .35rem`；H3 使用 `margin:.5rem 0 .3rem`；一般 `p`、`ul`、`ol`
  使用 `margin:.35rem 0`；`li` 使用 `margin:.12rem 0`；一般 section 與卡片使用
  `margin:.55rem 0` 及 `.4rem .5rem` padding；`.speech-unit` 使用
  `margin:.22rem 0`、`.35rem .5rem` padding 及 `line-height:1.45`。頁型專用規格若提供
  更精確數值，以較精確值為準，但不得退回瀏覽器預設 margin。
- 一般頁首 `.meta` 使用 `color:#53635f;font-size:.9rem;line-height:1.7`、
  `margin:.35rem 0;padding:0;border:0;background:transparent`，作為緊湊的說明文字，
  不與 `.ai-content-notice` 共用提示框內距或框線。單字卡 `.meta` 與 `.term-meta`
  仍使用各頁型明定的 `.78rem` 字級、`1.35` 行高及 `.15rem 0` margin。
- 一般 `.speech-unit` 與 `.term-meta` 使用 `white-space:normal`，讓原始碼的縮排
  換行只作為可折疊空白，不形成正文前後的空白行。不得在整個朗讀單元套用
  `pre-line`／`pre-wrap` 來保留排版換行；只有 canonical 正文確實包含有意義換行的
  特定區塊才可局部保留，且不得將格式化新增的空白混入正文或 `data-text`。
- 卡片、語言區塊、朗讀單元、標題與正文只使用內容自然高度。除本文件明定尺寸的可操作控制項外，
  不得設定固定 `height`、只為對齊而設定 `min-height`、插入空 `<p>`／`<br>`／不可見元素，
  或用超過 `1rem` 的上下 padding 撐開內容。手機版可增加 `scroll-margin-top` 避開 sticky
  控制列，但不得增加正文 margin、卡片 padding 或空白佔位。
- 相同頁型中相同 selector 的 `margin`、`padding`、`line-height`、`height` 與
  `min-height` 必須逐字一致。完成前以 computed style 比較至少第一張、中間一張及最後一張
  卡片；內容長度不同只能改變自然高度，不得改變上下留白。
- Browser 驗證固定使用 320、390、660、1280px 四個 viewport；每個寬度都記錄 body、
  main/header、H1/H2/H3、一般 paragraph/list、第一／中間／最後卡片與 speech unit 的
  `font-family`、`font-size`、`font-weight`、`line-height`、`margin`、`padding`、`width`、
  `height`、`min-height`、`display`、grid/flex 及 wrapping。播放器另記錄所有可見 button、
  label、select、range、status/error 的同組屬性；工具頁另記錄 search、filters、結果數、
  map link 及 pressed/hidden 狀態。比較時只允許文字長度造成的自然 width/height 差異，
  不得放寬字型、留白、控制尺寸、欄數、gap、斷點或互動狀態差異。
- 每個 viewport 須同時檢查 `document.documentElement.scrollWidth <= innerWidth`、sticky
  控制列不遮住 focus/active 內容、文字沒有 clipping/overlap、卡片沒有固定內容高度。
  660px 要分別在斷點本身及相鄰兩側抽測：桌面欄位規則在 `>660px`，mobile 單欄／雙欄
  transport 規則在 `<=660px`；390px 與 320px 仍沿用 mobile 規則，不得新增會逐字直排
  label 或改變 transport 順序的例外。
- 主要頁的所有寬表格放入 `.table-wrap` 並只讓該容器水平捲動，不得讓表格撐開 viewport。
  標題、段落、清單、表格及卡片採緊湊但可讀的 margin、padding 與行距，正文須是主要
  視覺內容。
- 所有 `lesson.html` 使用同一份鎖定 CSS：17px/1.7 字體、1000px main、可水平捲動的
  `.table-wrap`、雙語 AI 提醒及本節規定的 Reader grid。頁面不建立 TTS 控制列或
  JavaScript；只替換來源標題、教材內容、表格數量、Reader 小節／配對及連結文字。
- 英文朗讀單元使用 `#eaf3f8`，中文使用 `#f5f0df`；`.active` 使用 `#ffe4a3`、
  `#b96800` 左框與陰影，所在卡片另加 `.active-group`。目前狀態不可只靠顏色表示。
- Bilingual Reader 的每個來源小節使用一個 `.bilingual-section` 與可見 H2；其內的
  `.bilingual-pair` 採緊湊間距，可顯示不進入朗讀正文的 pair number。英文教學與本文導讀的
  `.teaching-point` 同樣採緊湊卡片，標題、繁中單元及英文單元不可被過大留白分隔。
- 獨立 Bilingual Reader 的 `.speech-unit` 直接包含可見純文字，不額外包 `<p>` 或其他會帶
  瀏覽器預設 margin 的元素；`data-text` 必須逐字等於該容器的 `textContent.trim()`。
  `.bilingual-pair` 與 `.speech-unit` 只使用內容自然高度，不設固定或最小內容高度，也不得
  以空白、不可見元素或預設段落 margin 撐高卡片。
- Bilingual Reader、英文教學、本文導讀、中文單字老師、文法頁及術語頁使用淡綠色 sticky 控制列、
  白色按鈕與可換行的 `.row`。所有 TTS 頁皆提供 ±10 控制；Bilingual Reader、英文教學與本文導讀
  另提供 reading mode，中文單字老師、文法頁及術語頁使用七鍵 compact player，
  不得混用兩種播放器契約。
- 所有播放器的原生 `button`、`select` 與 `input` 必須明示 `font:inherit`，使控制文字、
  選項與語速欄位繼承頁面的 `17px/1.7 "Noto Sans TC","Segoe UI",sans-serif` 字型；不得讓
  Edge、Chrome 或其他瀏覽器套用不同的預設表單字型。完成前須以 computed style 比對
  body 與三種控制項的 `font-family`，並確認同一播放器殼層各頁完全一致。
- Bilingual Reader、英文教學與本文導讀的 `.row` 內先放一個佔滿整列且可換行的
  `.transport`，所有 transport button 明示 `type="button"`；其後依序放四個
  `.control-field`，最後放 `.status-line`。`.controls`、`.transport`、`.control-field`
  與其 select/input 均須有適當的 `width`、`min-width:0` 或 flex basis，並明示控制項前景色，
  不得依賴 Chrome 專屬的隱含尺寸或色彩。以 Microsoft Edge 實際驗證此 DOM，不得用另一套
  臨時控制列取代。
- 每個 `.control-field` 必須是包含獨立 `<label>` 與 form control 的 wrapper，不得把
  `.control-field` class、文字及 select/input 全部放在同一個 `<label>`。欄位 label 固定
  使用 `writing-mode:horizontal-tb;white-space:nowrap;word-break:keep-all;overflow-wrap:normal`，
  不得因 flex/grid 收縮而逐字換行或呈現直排；select/input 可縮放，但不得反向擠壓 label。
  660px 以下 `.control-field` 佔滿可用列寬，控制項使用容器安全的寬度並在 label 下方排列。
- Bilingual Reader、英文教學與本文導讀共用同一份完整播放器殼層。三頁的整個 `<style>`、下方固定
  `.controls` DOM 及 Web Speech JavaScript 必須逐字一致；共用 style 同時包含
  `.bilingual-section`、`.bilingual-pair` 與 `.teaching-point` 的內容樣式，不建立頁型
  專屬 CSS 或控制器分支。三頁只替換 `<title>`、H1、meta、教材內容、內容分組及返回連結，
  不得以簡化腳本、其他頁型控制器或課程專屬播放器取代。
- 共用完整控制器須追蹤目前內容群組：Bilingual Reader 使用 `.bilingual-pair`，英文教學與本文導讀
  使用 `.teaching-point`。兩種群組都使用 `scroll-margin-top:15rem`；首次播放或進入另一個
  群組時，依下方共用定位契約保留群組脈絡，並優先確保目前單元可見。同一群組內換段
  不反覆置頂，僅在目前單元被遮住或超出畫面時補捲動。停止、播放完成時須清除群組與單元
  定位追蹤；導覽或設定變更時須使舊定位失效，再依共用待播契約定位目標。
- 中文單字老師與文法頁共用下方文法播放器的視覺數值、`.controls > .row`、按鈕、欄位、
  `.speech-unit`、active 樣式、響應式規則及 JavaScript。單字頁可保留 `header` 與單字卡
  內容樣式；兩頁都不得覆蓋共享播放器選擇器或建立另一套控制列。
- 中文單字老師與文法頁的七個播放按鈕須包在獨立且佔滿一列的 `.transport`；
  `下 10 段` 是該列最後一個控制項。中文語音、英文語音及語速各自使用不可拆散的
  `.control-field` 包住 label 與對應 control，從 `.transport` 下一列開始排列；status 與
  error 使用獨立 `.status-line`。不得把裸露的 label 與 select 直接混排在按鈕之後，也不得
  讓瀏覽器在 label 與其 select 之間自行決定換行。
- 單字頁的 `header` 與 `main` 固定使用 `width:min(1000px,calc(100% - 1rem))`；
  `header` 使用 `margin:1rem auto 0;padding:0`，`main` 使用 `margin:1rem auto 3rem`。
  `header > h1` 固定為 `font-size:1.5rem;line-height:1.25;font-weight:700;margin:0 0 .75rem`，
  不套用一般 H1 的 `2em`，660px 以下仍為 `1.5rem`。header 內的導覽段落使用
  `margin:.35rem 0`；不得把 main 的 `3rem` 下 margin 套到 header，也不得加入空白佔位。
  單字頁連結使用 `color:#11695e;background:transparent`。
- 單字卡 H2 本身是第一個 `.speech-unit[data-lang="en-US"][data-text]`，其可見文字與
  `data-text` 都是英文字詞。H2 使用 `font-size:1.15rem`、`line-height:1.25`、
  `margin:.2rem 0`、2px `#91b9af` 下框及完整的 `padding:0 0 .25em`；另以
  `.vocabulary-item > h2.speech-unit` 移除上、左、右框與圓角並使用透明背景，使其維持
  標題外觀；`.active` 時才使用共用 active 背景、左框與 6px 圓角，並改為
  `padding:0 0 .25em .5rem`。不得殘留通用 `.speech-unit` 的上、左、右 padding。
  卡片直屬 meta 使用 `font-size:.78rem`、`line-height:1.35`、`margin:.15rem 0`、
  `padding:0;border:0;background:transparent`，不得套用 `.ai-content-notice` 的提示框。
  `.vocab-details` 是不朗讀的發音與搭配清單，
  使用 `font-size:.9rem`、`line-height:1.4`、`margin:.25rem 0` 及
  `padding-left:1.35rem`，其 `li` 使用 `.05rem 0` margin 及
  `font-size:inherit;line-height:inherit`，不套用一般清單的 `1.7` 行高。這四類資訊須比朗讀正文緊湊，
  但不得小於上述數值。
- 每張 `.vocabulary-item` 使用 `scroll-margin-top:15rem`。首次播放或進入另一張單字卡時，
  依共用定位契約保留卡片脈絡並確保目前單元可見；同卡四單元換段不反覆置頂，但須在
  目標被控制列遮住或超出畫面時補捲動。停止後須清除定位追蹤，使下次播放仍能定位。
- `.vocab-explanation` 使用 `display:grid;gap:.2rem;margin:0;padding:0`；每個
  `.explanation-row` 使用 `display:grid;grid-template-columns:5.5rem minmax(0,1fr)`、
  `.3rem` gap、`margin:0;padding:.2rem .4rem;font-size:inherit;line-height:inherit`，
  label 使用 `#654f16`。五列全部使用 `border:0;border-radius:0;background:transparent;box-shadow:none`。
  五個欄位在所有寬度均須由上到下各佔一列；兩欄 grid 只用於單列內的 label 與 value，
  不得在 `.vocab-explanation` 設定兩個等寬欄，將不同欄位排成左右兩欄。
  `.explanation-row.term` 的 `term` 只是欄位角色，不是術語卡；術語卡片樣式須限定在
  `article.term`，不得讓欄位繼承白底、框線、圓角、陰影或卡片留白。
- 單字頁第二個繁中單元使用
  `.vocabulary-item > .speech-unit[data-lang="zh-TW"]:not(.example-translation)` 明定
  `white-space:normal`，五欄及其 label/value 繼承此值；不得另行保留原始碼縮排，
  使其變成可見空白行。內部 `<p>` 使用 `margin:0;line-height:inherit`。
  660px 以下 explanation row 改單欄，header 與 main 使用
  `calc(100% - .5rem)` 寬度，speech unit scroll margin 18rem；卡片 padding、五欄 gap
  與上述緊湊留白不變。
- 文法頁鎖定參考頁變數與基線：`--ink:#20302d`、`--paper:#f7f7f2`、`--line:#d6dfdc`、
  `--en:#eaf3f8`、`--zh:#f5f0df`、`--active:#ffe4a3`、`--active-line:#b96800`；
  body 使用 `17px/1.7 "Noto Sans TC","Segoe UI",sans-serif`。main 使用
  `width:min(1000px,calc(100% - 1rem));margin:1rem auto 3rem`。
- 文法控制列固定為 `position:sticky;top:0;z-index:100`、背景 `#e9f3f0`、
  `border-bottom:1px solid var(--line)`、padding
  `.55rem max(.6rem,calc((100% - 1000px)/2))` 及 `0 3px 10px #0012` 陰影。
  `.row` 使用 flex、wrap、`align-items:center` 及 `.4rem` gap；button/select 使用
  `font:inherit`、`1px solid #91aaa4`、6px 圓角、白底及 `.38rem .58rem` padding；
  range input 同樣使用 `font:inherit`。
- 文法 `.speech-unit` 使用 4px 左框、6px 圓角、`.35rem .5rem` padding、
  `.22rem 0` margin、`line-height:1.45`、`scroll-margin-top:11rem`、`white-space:normal`；
  內部 `<p>` margin 必須為 0。英文背景 `#eaf3f8`，繁中背景 `#f5f0df`；active 使用
  `#ffe4a3`、`#b96800` 左框及 `0 4px 16px #7a4d1945` 陰影。
- `.grammar-point` 使用白底、`1px solid var(--line)`、8px 圓角、`.4rem .5rem` padding、
  `.55rem 0` margin；H2 使用 `margin:.05rem 0 .35rem`、`.3rem` 下 padding、
  `2px solid #91b9af` 下框、`1.18rem` 字級及 `#244f47`。active 所在卡片加
  `0 0 0 2px #b9680060`。
- 每個 `.grammar-point` 使用 `scroll-margin-top:15rem`。compact 控制器須追蹤目前捲動
  目標：導讀與總結以其 `.speech-unit` 本身為目標，文法內容以所在 `.grammar-point` 為
  目標。首次播放 standalone 單元或進入另一個文法點時，依共用定位契約保留群組脈絡並
  確保目前單元可見；同一文法點的五個單元換段不反覆置頂，但須在目標被遮住或超出
  畫面時補捲動。停止或播放完成時須清除定位追蹤，使下次播放能重新定位目前內容。
- 所有完整與精簡播放器的 `.transport` 固定
  `width:100%;display:grid;flex:1 1 100%;gap:.4rem`，欄數依下方 TTS 契約設定，不使用
  flex 排列按鈕；`.control-field` 固定
  `display:flex;flex:1 1 13rem;align-items:center;gap:.35rem;min-width:0`，label 固定
  `writing-mode:horizontal-tb;white-space:nowrap;word-break:keep-all;overflow-wrap:normal`，
  select／range 固定 `width:100%;min-width:0`。`.status-line` 使用
  `display:flex;flex:1 1 100%;gap:.5rem;min-width:0`，不得用 status 的 auto margin
  把欄位擠到非預期列。
- 文法頁 660px 以下 body 改 16px、controls padding `.4rem`、`.transport` 改兩個等寬欄，
  每個 `.control-field` 佔滿一列並改為單欄 grid，使 label 保持水平且 control 位於其下；
  status-line 獨占一列、main 寬度 `calc(100% - .5rem)`、speech unit scroll margin 18rem。
- 中文單字老師、文法頁與術語頁共用本節規定的 compact player CSS、控制列 DOM 及
  播放器 JavaScript；術語頁的搜尋／篩選必須放在播放器以外的獨立腳本。不得出現
  `.control-grid`、`.field`、
  `.buttons`、`.script-unit` 或另一組控制 ID／視覺數值。
- 術語頁 H1 固定使用 `font-size:1.5rem`、`line-height:1.3` 與
  `margin:0 0 .5rem`；長標題依內容自然換行，不得使用主要課程的
  `clamp(1.65rem,5vw,2.5rem)` 大標題、固定高度、縮放 transform 或裁切文字。
- 每個 `.vocabulary-item` 嚴格依序包含四個 `.speech-unit`：作為 H2 的英文單字、自然
  繁中五欄解說、英文例句、繁中例句翻譯。H2 後不得再建立內容相同的獨立英文
  `.speech-unit`；四者不得合併、缺漏、重排或加入 `Word:` 等額外朗讀標籤。
- 整合術語頁的 `article.term` 固定使用白底、`1px solid var(--line)`、8px 圓角、
  `.4rem .5rem` padding、`.55rem 0` margin 與 `scroll-margin-top:15rem`；
  `.term.active-group` 使用 `0 0 0 2px #b9680060`。術語 H2 固定使用
  `font-size:1.15rem`、`line-height:1.25`、`margin:.2rem 0`、`.25em` 下 padding
  與 `2px solid #91b9af` 下框；`.term-meta` 使用 `font-size:.78rem`、
  `line-height:1.35`、`margin:.15rem 0;padding:0;border:0;background:transparent`。
  `.term-meta` 位於 H2 下方，以同一小字列依序顯示 domain、完整詞性及「證據：[分類]」，
  以 ` · ` 分隔，窄螢幕可自然換行。不得另建證據卡片、提示框或獨立欄位。
- 每個 `.term-field` 使用 `display:grid`、`grid-template-columns:7rem minmax(0,1fr)`、
  `.3rem` gap 及 `align-items:start`；術語 H2 維持前述 `.2rem 0` margin，其餘四個
  `.term-field` 固定使用 `.12rem 0` margin。非術語欄位內的 `.speech-unit` 固定使用
  `margin:0` 與 `.18rem .4rem` padding，不得再套用 compact player 通用的上下 margin
  或較大 padding，避免「定義」、「來源語境」等相鄰欄位之間形成空白段落。
  `.field-label` 使用 `font-weight:700` 與 `color:#654f16`；`.field-value` 使用
  `min-width:0`。
  `.bilingual-value` 使用 flex、wrap、`align-items:baseline` 與 `.22rem` gap，使英文術語、
  `｜` 與繁中術語保持同一文字流；`.example-pair` 使用 grid 及 `.22rem` gap，使兩個例句
  各自成列。`.separator` 只在雙語術語欄中可見且不朗讀。660px 以下 `.term-field` 改為單欄，
  `.term` 保持自然高度，不得以固定高度或 `min-height` 對齊卡片。
- 搜尋與 domain 篩選放在獨立 `.tools` 卡片。`.tools` 固定使用白底、
  `1px solid var(--line)`、8px 圓角、`.5rem` padding 與 `.55rem 0` margin。
  `.search-field` 使用 `display:grid`、
  `grid-template-columns:max-content minmax(0,1fr)`、`align-items:center` 與 `.4rem` gap；
  搜尋 input 使用 `width:100%`、`min-width:0`、`1px solid #91aaa4`、6px 圓角及
  `.38rem .58rem` padding。`.filters` 使用 flex、wrap、`.3rem` gap 及 `.5rem` 上 margin；
  filter button 使用 `1px solid #91aaa4`、6px 圓角、白底、深色文字及
  `.3rem .5rem` padding，`[aria-pressed="true"]` 使用 `#244f47` 邊框與背景及白字。
  `.hidden` 固定為 `display:none!important`。660px 以下 `.search-field` 改為單欄。
  `#resultCount` 與 `#noResults` 使用 live status；無 JavaScript 時所有術語保持可見。
- 整合術語頁使用七鍵 compact player，包含 `previous10` 與 `next10`，不建立 `readingMode`。
  每個 `.term` 固定依英文術語、繁中術語、繁中定義、繁中來源語境、繁中流程關係、
  英文例句、繁中例句翻譯的七個欄位級 `.speech-unit` 自然接續；分類、詞性與證據只在
  `.term-meta` 顯示，不加入 TTS。不得建立
  內容重複的完整中文與英文說明。active unit 所在卡片使用 `.term.active-group`。
  播放器腳本不得包含搜尋／篩選邏輯，搜尋／篩選腳本也不得註冊或覆蓋任何 TTS 控制 ID。
- 術語頁控制器必須以 `unit.closest("article.term[data-term][data-domain]")` 取得卡片，
  依共用待播／播放契約為導覽目標或實際開始朗讀的單元加 `.active`、為卡片加
  `.active-group`，並追蹤已定位的卡片與
  單元。每張卡使用 15rem `scroll-margin-top`；依共用定位契約保留卡片脈絡，同卡七個
  單元換段不反覆置頂，但須在目標被遮住或超出畫面時補捲動。停止、播放完成及重新開始
  新的播放流程時須正確清除或重設定位追蹤，使下一次播放仍能定位目前內容。
- 長字、網址與標題使用 `overflow-wrap:anywhere`。TTS 控制列使用
  `position:sticky`、`top:0`、足夠 `z-index`、容器安全的 `width:100%` 及可換行布局，
  不得使用 `100vw` 搭配負 margin。660px 以下控制項可伸展換行，status 與錯誤訊息各自
  使用完整一列；朗讀單元保留足夠 `scroll-margin-top`。
- 所有頁面必須在 320px、390px 與一般桌面寬度實測，不得有整頁水平溢出、文字或控制項
  重疊、sticky 控制列遮住正文，或無法操作的按鈕與選單。

### TTS 與控制器鎖定契約

- 所有包含 TTS 播放器的 HTML 頁面（含 compact 頁與 workspace 額外輸出）皆須提供
  「往前 10 段」與「往後 10 段」按鈕；可見文字沿用下列鎖定值「⏪ 上 10 段」與
  「下 10 段 ⏩」，ID 分別為 `previous10` 與 `next10`，不得省略。
- 所有 TTS 頁都使用 `.speech-unit[data-lang][data-text]`。完整 `data-text` 必須在同一
  單元中可見，並
  逐字等於可見朗讀正文；英文固定 `en-US`，繁中固定 `zh-TW`。
- `.speech-unit` 是朗讀資料與高亮目標，不是播放控制項。不得以點擊正文或標題、
  聚焦後按 Enter／Space 啟動或切換朗讀，也不得在其祖先上委派這類事件。
  不為純朗讀單元加入 `tabindex`、`role="button"`、可點擊游標或點擊朗讀提示；
  單元內原有連結與其他互動元件保留原生操作及焦點，不觸發朗讀。
  播放、暫停／繼續、停止與段落導覽由播放器控制列操作；自然接續、高亮、進度、
  捲動與語言模式維持原契約。
- Bilingual Reader、英文教學與本文導讀的控制 ID 固定為 `previous10`、`previous`、
  `play`、`pause`、`stop`、`next`、`next10`、`readingMode`、`zhVoice`、`enVoice`、
  `rate`、`rateValue`、`status`、`error`。按鈕全部明定 `type="button"`；`rate` 為
  0.6–1.4、step 0.1、預設 1，`rateValue` 顯示一位小數，status 初始為 `準備完成`。
- Bilingual Reader、英文教學與本文導讀必須有 `readingMode`，固定提供 `中英皆讀`（預設）、
  `只讀中文`、`只讀英文`。compact 頁不建立 `readingMode`；單字頁依每卡
  `en-US → zh-TW → en-US → zh-TW`，文法頁依導讀、各文法點五單元及總結自然接續，
  術語頁依每卡 `en-US → zh-TW → zh-TW → zh-TW → zh-TW → en-US → zh-TW`
  自然接續，分類、詞性與證據只在小字列顯示，不朗讀。
- Bilingual Reader、英文教學與本文導讀的控制列固定使用下列 DOM。不得改動元素層級、class、ID、
  控制順序、按鈕／標籤文字、option 值、range 範圍、ARIA 或初始狀態：

```html
<div class="controls" aria-label="朗讀控制列">
  <div class="row">
    <div class="transport">
      <button id="previous10" type="button">⏪ 上 10 段</button>
      <button id="previous" type="button">⏮ 上一段</button>
      <button id="play" type="button">▶ 播放</button>
      <button id="pause" type="button">⏸ 暫停／繼續</button>
      <button id="stop" type="button">■ 停止</button>
      <button id="next" type="button">下一段 ⏭</button>
      <button id="next10" type="button">下 10 段 ⏩</button>
    </div>
    <div class="control-field">
      <label for="readingMode">朗讀語言</label>
      <select id="readingMode">
        <option value="both">中英皆讀</option>
        <option value="zh-TW">只讀中文</option>
        <option value="en-US">只讀英文</option>
      </select>
    </div>
    <div class="control-field">
      <label for="zhVoice">中文語音</label><select id="zhVoice"></select>
    </div>
    <div class="control-field">
      <label for="enVoice">English voice</label><select id="enVoice"></select>
    </div>
    <div class="control-field">
      <label for="rate">語速 <output id="rateValue">1.0x</output></label>
      <input id="rate" type="range" min=".6" max="1.4" step=".1" value="1">
    </div>
    <div class="status-line">
      <span id="status" data-ready="準備完成" role="status" aria-live="polite">準備完成</span>
      <span id="error" role="alert"></span>
    </div>
  </div>
</div>
```

- 控制列依播放與導覽、朗讀模式、語音選擇、語速、狀態與錯誤訊息分組，同類控制項不得
  交錯。文字、字級、內距與間距保持緊湊，但不得犧牲可讀性、點按範圍、完整功能或錯誤
  可見性。
- 所有 transport button 的可見內容必須同時包含本節鎖定 DOM 中的固定圖示與固定文字；
  圖示不得只存在於 `title`、`aria-label`、CSS pseudo-element、背景圖片或 hover 狀態，
  文字也不得以 visually-hidden 方式隱藏。按鈕的 accessible name 必須包含對應動作文字，
  且不得只有 `⏪`、`⏮`、`▶`、`⏸`、`■`、`⏭` 或 `⏩` 等符號。
- 逐按鈕鎖定圖示與文字配對：`⏪ 上 10 段`、`⏮ 上一段`、`▶ 播放`、
  `⏸ 暫停／繼續`（compact player 固定為 `⏸ 暫停 / 繼續`）、`■ 停止`、
  `下一段 ⏭`、`下 10 段 ⏩`。不得交換圖示、刪除文字、使用 emoji-only 變體或在不同頁型
  改寫同一動作。
- 中文單字老師、文法頁與術語頁共用 `.controls > .row` 及固定控制 ID：
  `previous10/previous/play/pause/stop/next/next10/zhVoice/enVoice/rate/rateValue/status/error`；
  不建立 `readingMode` 或別名 `prev10`。七個按鈕依下列 DOM 的順序、圖示與文字排列；
  接著依序為中文語音、英文語音、含 `rateValue` 的語速、status 與 error。
- 中文單字老師、文法頁與術語頁的控制列固定使用下列 DOM。不得改動元素層級、class、ID、
  控制順序、按鈕／標籤文字、range 範圍、ARIA 或初始狀態：

```html
<div class="controls" aria-label="朗讀控制列">
  <div class="row">
    <div class="transport">
      <button id="previous10" type="button">⏪ 上 10 段</button>
      <button id="previous" type="button">⏮ 上一段</button>
      <button id="play" type="button">▶ 播放</button>
      <button id="pause" type="button">⏸ 暫停 / 繼續</button>
      <button id="stop" type="button">■ 停止</button>
      <button id="next" type="button">下一段 ⏭</button>
      <button id="next10" type="button">下 10 段 ⏩</button>
    </div>
    <div class="control-field">
      <label for="zhVoice">中文語音</label><select id="zhVoice"></select>
    </div>
    <div class="control-field">
      <label for="enVoice">英文語音</label><select id="enVoice"></select>
    </div>
    <div class="control-field">
      <label for="rate">語速 <output id="rateValue">1.0x</output></label>
      <input id="rate" type="range" min=".6" max="1.4" step=".1" value="1">
    </div>
    <div class="status-line">
      <span id="status" data-ready="準備完成" role="status" aria-live="polite">準備完成</span>
      <span id="error" role="alert"></span>
    </div>
  </div>
</div>
```

- 上述 compact player 的 `.transport` 必須形成播放器第一列；`中文語音` 所在的第一個
  `.control-field` 必須從下一列開始。這是固定分組，不得依 viewport 剩餘空間碰巧換行。
  設定區域展開時，三個 `.control-field` 在桌面寬度可同列，在 660px 以下則各自佔滿一列。
- 所有 TTS 頁共用以下設定列顯示契約，不另建控制列 wrapper、切換按鈕或頁型專屬行為。
  上述固定 DOM 是初始展開狀態，`.control-field` 初始不帶 `hidden`；執行時只切換各欄位
  wrapper 的 `hidden` property，不移除、重建或重排控制項：

  | 播放狀態 | 設定區域 |
  | --- | --- |
  | 初始、未播放、已停止、導覽或設定變更後待播 | 展開 |
  | 播放中、繼續播放或自然接續 | 收合 |
  | 已暫停、播放完成或目前播放發生錯誤 | 展開 |

  收合範圍是 `.controls` 內全部 `.control-field`，包含適用頁面的 `readingMode`、
  `zhVoice`、`enVoice`、`rate`、`rateValue` 及其 label；不得只隱藏語音選單或只隱藏
  control 而留下 label。`.transport`、`.status-line`、`status` 與 `error` 必須維持可見，
  不得收合整個 `.row` 或 `.controls`，也不得影響正文、搜尋或 domain 篩選區域。
- 所有播放器殼層共用以下 helper，並在共用 CSS 的響應式規則之後加入
  `.controls .control-field[hidden] { display:none; }`，確保 `hidden` 不被欄位的 flex/grid
  覆蓋。收合後欄位不佔高度、列間距或鍵盤 Tab 順序；不得以 `visibility:hidden`、
  `opacity:0`、固定高度、`min-height`、空 wrapper 或佔位元素保留設定區域空白：

```javascript
function setPlaybackSettingsHidden(hidden) {
  document.querySelectorAll(".controls .control-field").forEach((field) => {
    field.hidden = hidden;
  });
}
```

- 開始播放前及從暫停繼續播放時呼叫 `setPlaybackSettingsHidden(true)`；暫停、停止、手動段落導覽、
  朗讀語言／中英文語音／語速變更、最後一個單元播畢或目前播放發生錯誤時呼叫
  `setPlaybackSettingsHidden(false)`。暫停後不論由 `play` 或 `pause` 繼續，都須再次收合。
  未播放或待播時按暫停不得啟動朗讀；導覽與設定變更須展開設定列，自然接續則維持收合。
  顯示狀態依控制器的播放流程更新，不以單次讀取 `speechSynthesis.speaking` 或短暫的
  utterance 間隙判定播畢。停止、錯誤及播畢後須同步清除 active 樣式與群組追蹤，狀態文字
  不得仍顯示播放中；真正錯誤仍顯示於可見的 `error`。
- 收合與展開本身不得重設 `readingMode`、中英文 `voiceURI`、`rate` 或 `rateValue`；
  使用者明確變更設定時，保留新值並依下述設定變更流程待播。不得改變未經導覽或設定變更的
  一般暫停／繼續、朗讀順序、語音評分與載入規則。所有
  `onend`／`onerror` 必須先依下述 token 規則排除舊回呼，再更新設定列與播放狀態；
  被停止、導覽或設定變更取消的舊 utterance 不得改變新流程的設定列或目前 utterance 引用。
- 所有 TTS 頁的 `.transport` 使用等寬 CSS Grid，按鈕不得依文字長度產生不同寬度。
  所有頁型在 1000px 以上固定為七個等寬欄，超過 660px 且未滿 1000px 為四個等寬欄，
  660px 以下一律為兩個等寬欄。所有 transport button 使用 `line-height: normal` 與
  `padding-block: 0.28rem`，不得設定 44px 最小高度；按鈕文字與圖示不得溢出或遭截斷。
- 所有 TTS 頁的 `.controls` 定義
  `--compact-control-height: calc(1.6em + 0.5rem)`。transport button、每個
  `.control-field select` 及 `.control-field input` 均使用此高度；完整播放器的朗讀模式
  選單也適用。不得以 `min-height` 覆蓋此值，語音選單與語速滑桿必須和播放按鈕等高。
- 所有 TTS 頁必須逐字共用同一個 voice scoring 與載入函式，不得依課程、頁型或 workspace
  改寫。評分輸入固定為 `(voice.name + " " + voice.voiceURI).toLowerCase()`，初始分數為
  0，並依序套用：符合 `/natural|online/` 加 20；符合
  `/xiaoxiao|曉曉|zira|aria|jenny|female/` 加 10；評選中文 voice 時，若同時符合
  `/microsoft/`、`/xiaoxiao|曉曉/` 與 `/natural|online/` 再加 100。不得加入地區、
  `default`、`localService`、供應商或其他隱藏加權。
- 因此固定優先層級為：中文 Microsoft Xiaoxiao／曉曉 Natural／Online 130 分；
  名稱命中偏好且為 Natural／Online 的中英文 voice 30 分；其他 Natural／Online 20 分；
  只命中 Xiaoxiao／曉曉、Zira、Aria、Jenny 或 female 的 voice 10 分；其餘 0 分。
  分數只決定同一語言候選清單內的預設 voice，不得讓中文 voice 進入英文選單或反向混用。
- 中文與英文候選清單分別只保留 `voice.lang.toLowerCase().startsWith("zh")` 與
  `.startsWith("en")` 的 voice，並維持 `speechSynthesis.getVoices()` 的原始順序；
  `<option value>` 固定使用 `voiceURI`。最高分以嚴格大於目前最高分時才替換，因此同分
  固定保留瀏覽器原始順序中的第一個，不得另以名稱、URI、locale 或字母順序破壞平手。
- 頁面初始化時立即呼叫一次共用 `loadVoices()`，並註冊 `voiceschanged` 後以同一函式重載。
  每次重載先記錄兩個 select 的舊 `voiceURI`：舊值仍存在於對應語言候選清單時必須保留；
  否則選取該語言最高分 voice。候選清單為空時保留空選單，不得跨語言塞入 voice、偽造
  option 或回退到未經評分的第一個 voice。之後 voices 出現時須自動完成正確選取。
- 每次新播放、停止、段落導覽或朗讀語言／中英文語音／語速變更，都先遞增 token 並清除
  舊 utterance 引用，再呼叫 `cancel()`，取消目前語音及待播佇列，使同步到達的取消回呼
  也失效。所有 start／boundary／end／error 回呼先核對 token、目前 utterance 與單元，
  避免 stale callback。四個手動導覽按鈕取消目前語音後只定位目標單元並待播，
  不建立或提交新語音；只有有效播放流程中的自然結束才自動接續。`canceled`／`interrupted`
  不顯示為錯誤，其他錯誤須顯示於
  可見 status／error 區域。所有 TTS 頁在目前 eligible units 的首段停用 `previous` 與
  `previous10`，末段停用 `next` 與 `next10`；停用按鈕不得取消或重播目前語音。
  進度格式為 `第 N 段 / 共 M 段`。
- **手動段落導覽**：所有 TTS 頁（含完整、compact 與 workspace 額外輸出）的
  `previous10`、`previous`、`next`、`next10` 共用同一導覽流程，位移分別為
  `-10/-1/+1/+10`。從未播放、播放中、已暫停、完成或錯誤狀態操作時，均先使舊播放失效並
  取消語音，再更新 clamp 後的 eligible index、展開設定列、定位並高亮目標，進入「待播定位」。
  此流程不得呼叫 `speak()`／`resume()`、建立新 utterance 或排程自動播放，也不得保留
  舊暫停語音供稍後繼續；須重設控制器的播放中／暫停旗標，不以引擎瞬間的 speaking／paused
  值恢復舊流程。停用的首尾按鈕維持無副作用。
- **設定變更**：所有 TTS 頁的 `zhVoice`、`enVoice` 與 `rate`，以及適用頁面的
  `readingMode`，共用停止並待播流程。使用者實際變更任一設定時，不論目前為未播放、
  待播、播放中、已暫停、完成或錯誤，均立即使舊播放失效、取消語音及佇列，重設播放中／
  暫停旗標、短 utterance 索引與文字 offset，清除舊逐字高亮並展開設定列。
  不得建立新 utterance、呼叫 `speak()`／`resume()` 或排程自動續播；變更非目前朗讀語言的
  語音選單也適用。語音或語速變更維持目前 eligible index 與目標單元，依展開後布局確認
  目標可見，保留或重建其待播段落／群組高亮；只更新使用者變更的設定，不重設其他選值。
  `readingMode` 變更沿用既有規則：依新模式重建 eligible units，將索引設為第一個可朗讀
  單元，再定位並高亮，不保留不符合新模式的舊目標。若沒有可朗讀單元，清除高亮與定位
  追蹤、停用播放及段落導覽並顯示 `沒有可朗讀內容`，不建立語音；有單元後恢復對應控制。
  各 select 使用 `change`，`rate` 使用 `input` 即時停止並更新 `rateValue` 為一位小數
  `Nx`，不得等滑桿放開或失焦才處理；同一數值的後續 `change` 不得重複重設待播狀態。
  初始化與 `voiceschanged` 自動重載不視為使用者設定變更，不得觸發此流程或自動開播，
  語音評分、選值保留與缺聲處理仍依既有載入規則。
- **待播與播放狀態**：導覽或設定變更後立即為目標單元加 `.active` 與 `aria-current="true"`，所在群組
  加 `.active-group`，移除舊單元／群組樣式與 `mark.spoken-word`；不以假 boundary 表示待播。
  status 固定顯示 `待播：第 N 段 / 共 M 段`，不得顯示播放中。只有使用者按 `play` 才從
  目標單元開頭、依目前語言模式、`voiceURI` 與語速開始新播放，不接續已取消的暫停內容；
  完整播放器須重設短 utterance 索引與文字 offset。
  待播時按 `pause` 不做任何播放或狀態切換。新語音提交後、有效 `onstart` 前仍不得宣稱
  已在朗讀；有效 `onstart` 才改成播放中狀態，並沿用該目標的段落／群組高亮。
  只有未經導覽或設定變更的既有暫停語音，才可由 `play` 或 `pause` 繼續。停止、真正錯誤
  或播畢仍清除高亮與定位追蹤；設定變更則清除舊狀態後建立新待播高亮，不得混為單純停止。
- 不支援 `speechSynthesis` 時必須顯示可見錯誤且完整內容仍可閱讀。所有控制提供
  `:focus-visible`，
  DOM 綁定不得依賴 ID 自動成為全域變數。
- 原生 button、select、range 與連結必須可用 Tab 導覽，並以 Enter／Space 或瀏覽器標準
  鍵盤操作啟用；不得用全域快捷鍵攔截輸入欄、搜尋欄或瀏覽器既有按鍵。
- Bilingual Reader 每組嚴格依 `en-US → zh-TW` 排列；英文教學與本文導讀每個
  `.teaching-point` 恰有一個 `zh-TW` 後接一個 `en-US` 單元，各單元依序包含對應語言的
  主題標題與說明，兩者皆須朗讀。整合術語頁不得套用此
  雙份說明結構，必須使用本文件定義的七個欄位級朗讀單元；`.term-meta` 不得加入 TTS。

### TTS 共用定位契約

所有 TTS 頁（含 workspace 額外輸出）共用同一套群組辨識、可視區計算與定位函式，
不得逐頁追加補丁或以播放延遲、盲目重試捲動掩蓋錯誤。

- **操作順序**：四個導覽按鈕與設定變更共用待播定位入口；導覽只改變
  `-10/-1/+1/+10` 位移並 clamp，設定變更的目標依前述規則選取。兩者均先遞增 token、
  清除舊 utterance 引用並取消語音及佇列，清除
  舊高亮與定位追蹤，再更新索引與待播狀態、展開設定列，依展開後的布局定位並高亮目標，
  不提交語音。按 `play` 開始新播放時才收合設定列，依收合後的布局重新確認目標可見，
  再提交目標語音；兩種操作都不得等待 `onstart` 才捲動。
- **可見性優先**：令 `top = controls.getBoundingClientRect().bottom + 8`、
  `bottom = window.innerHeight - 8`，以目標單元的 `getBoundingClientRect()` 驗證。
  高度不超過 `bottom - top` 的單元須滿足 `rect.top >= top` 且 `rect.bottom <= bottom`；
  超長單元須滿足 `top <= rect.top < bottom`，至少露出開頭。群組或卡片的
  `scroll-margin-top` 只供保留閱讀脈絡，不得以卡片已置頂代替目標單元可見。
- **捲動策略**：播放器定位（含自然接續）明示 `behavior:"instant"`，不得建立原生
  平滑動畫或依賴省略 `behavior` 的預設值。同群組可見的單元不反覆置頂；同一單元內
  接續短 utterance 不重複定位。停止、錯誤與完成須終止舊定位並重設追蹤；導覽與設定變更
  須使舊定位失效，再定位待播目標。語音／語速變更時，未變且仍可見的目標不得反覆置頂。
- **語音狀態分離**：導覽或設定變更的待播段落／群組高亮與進度立即更新，不依賴語音事件；
  沒有既有待播定位的新播放或自然接續，其新目標的 `.active`、`aria-current` 與 `.active-group`
  須等待有效 `onstart`。所有新語音的首次播放中狀態均須等待有效 `onstart`，不得將
  待播高亮視為已開始朗讀。未經導覽或設定變更的暫停保留已開始語音的高亮，
  繼續播放沿用先前確認的開始狀態，不重建語音或等待第二次 `onstart`。
- **回呼防護**：延後定位須先核對 token、目前索引、單元與控制器狀態；播放流程另核對
  目前 utterance，待播定位不得要求不存在的 utterance。通過並執行定位後才更新追蹤。
  過期回呼或未執行的捲動不得提前改變群組／單元追蹤，也不得拉回舊段；舊語音的
  start／boundary／end／error 不得覆蓋待播高亮、收合設定列或恢復朗讀。

### TTS 逐字高亮

- 所有含 TTS 的頁面均須保留段落 `.active` 與群組 `.active-group`，區分待播定位與朗讀中
  狀態。待播只高亮整個目標單元與群組，不建立 `mark.spoken-word`；朗讀時才以
  `SpeechSynthesisUtterance.onboundary` 同步高亮目前朗讀的文字範圍；「逐字」指語音引擎
  回報的字詞範圍，不保證每個中文字各有一次事件。不得用計時器推估或預先輪播字詞。
- 每次有效 boundary 先清除上一個 `mark.spoken-word`，再以 `charIndex` 與 `charLength`
  定位目前單元的可見文字。完整播放器須加上目前短 utterance 在原單元中的 offset，
  包含切句、去除前導空白及略過字幕標記的位移；compact 播放器直接使用單元內的索引。
  索引依 JavaScript 的 UTF-16 code unit 計算，範圍不得越界或切斷 surrogate pair。
  `charLength` 缺少或為 0 時，從 `charIndex` 取至下一個空白或中英文標點；無可取字詞時
  至少涵蓋一個完整字元。
- 使用 `TreeWalker` 與 `Range` 對應跨文字節點的範圍，包入唯一的
  `<mark class="spoken-word">`；不得重寫整個單元的 `innerHTML` 或 `textContent`。
  標題、段落、清單及行內語意結構須保留，`data-text` 與可見文字不得因高亮而改變。
  清除時解除 mark 包裝、保留其文字與行內子節點，並 normalize 相鄰文字節點，
  不得留下巢狀或殘留 mark，也不得把行內語意元素攤平成純文字。
- 共用樣式固定為 `.spoken-word { background:#ffcb60;text-decoration:underline; }`，
  不改變字級、字重、行距或尺寸；以底線輔助辨識，不得只靠顏色。完整與 compact
  播放器均納入同一套高亮邏輯與樣式，不建立課程專屬版本。
- `onboundary` 與其他語音回呼一樣，須確認 token、目前 utterance 強引用及目前單元仍相符，
  才可更新 DOM。暫停保留目前高亮，繼續後隨有效 boundary 更新；短 utterance 結束、
  切段、停止、自然播畢、真正錯誤、朗讀語言／中英文語音／語速變更及適用頁面的篩選切換
  均須清除逐字高亮。導覽或設定變更清除舊高亮後，立即保留或重建目標的待播段落／群組
  高亮，不等待 boundary 或 `onstart`，也不得讓舊 boundary 恢復逐字 mark。
- 語音引擎未提供 boundary 時，只保留段落與群組高亮，不偽造逐字位置、不阻斷朗讀；
  驗證回報須明列該環境無法實測逐字同步，不得將沒有事件誤報為逐字高亮已通過。

## 適用來源

本母版適用於人文、宗教、交通、教育、IT、愛情、友情、天文、地理、自然科學、社會科、
文學、藝術、生活、職場及其他主題。學習者年齡、程度、領域與教學重點應由來源內容及
使用者明示要求決定；未明示時只依來源合理推定，不得預設為成人、兒童、職場或特定產業。

raw 的來源及主要內容區塊名稱都不固定，可能包含：

- 書籍、教材、文章、故事、詩歌、經文、訪談、FAQ、政策、指南、報告、Email 或一般文件。
- Video、meeting、presentation、podcast、drama、lecture 或 training 的 transcript、captions 或 speaker notes。
- 對話、聊天、論壇、Teams、Slack 或其他訊息匯出的 conversation、thread 或 reader-friendly chat。
- 題目、答案、圖表描述、圖片替代文字、實驗步驟、案例、時間線、清單、表格、程式碼或公式。
- Summary、recap、digest、learning notes、action items 或繁體中文補充重點。
- 含有 `[Needs verification]`、`[unclear]`、`Source gap`、內部縮寫、程式碼、log、表格或附件說明的混合內容。

`Content`、`Transcript`、`Body`、`Message`、`Article` 等只是可能的標題，不是固定 schema，也不保證一定存在。不得只搜尋某個標題來判定正文，不得因區塊不叫 `Transcript` 而忽略內容，也不得把所有 raw 都描述成逐字稿。先依整份文件的語意及結構辨識來源類型、主要內容與補充內容，再使用本規格中適用的小節。

## Converted 內容區塊辨識

處理 raw 時，先為每個區塊判定下列角色；判定依內容功能，不依標題文字：

1. **Source metadata**：標題、寄件者、收件者、日期、場合、網址、文件版本、作者或講者。
2. **Primary source content**：實際要學習的 email 正文、文件內容、文章、訊息、聊天、逐字稿、政策、說明、簡報文字或其他原始材料。
3. **Source-provided supporting content**：來源本身提供的摘要、FAQ、表格、附件文字、caption、speaker notes 或補充說明。
4. **Prepared recap or learning aid**：後加的 recap、digest、中文重點、整理摘要、action items 或待確認清單。
5. **Noise or formatting artifacts**：字幕序號、無意義時間戳、頁碼、重複內容、匯出殘片、純導覽文字或不影響內容的系統事件。

converted 的 `本文 / Main Text` 內每組中文與英文共同表示一份正文，並非摘要與原文兩份
內容。`source_path`、`original_language` 等來源資訊歸入 metadata；`轉換註記 /
Conversion Notes` 歸入限制與待確認資訊，不當作正文或一般教學內容。

處理要求：

- `Primary source content` 是 Bilingual Reader 及語言分析的主要依據；可以有多個區塊，須依來源順序完整處理。
- Email raw 是不可變來源。生成 lesson 時不得修改其任何 headers、正文、空白、換行、條列、quoted history、簽名、URL 或格式；課程內引用時須忠實保留 subject、sender／recipient／date（若來源提供）、稱呼、正文、call to action 及結尾，且不得虛構缺少的 header。
- Email 的文法修正、自然英文版本、摘要、翻譯、action items、語氣分析及術語整理只出現在 lesson，並清楚標示為 teaching rewrite 或教學整理，不得冒充 Original wording。
- Transcript 須保留講者、發言順序及有意義的口語資訊；只有此類來源才使用「逐字稿」描述。
- Chat 須保留訊息順序、回覆關係、作者及系統事件的必要脈絡。
- Article、policy、specification、report 或一般文件須保留標題層級、段落、清單、表格及論述順序，不改寫成虛構對話或講者發言。
- `Source-provided supporting content` 若提供獨立資訊，放入 Reader 的相應位置；若只是主要內容的摘要，不重複朗讀全文。
- `Prepared recap or learning aid` 用於摘要、事實核對及課程結構，不冒充 primary source content；與主要內容衝突時，以可辨識的原始內容為準並標記差異。
- 只有確認為 noise 的內容才可移除。有疑義時保守保留並標示 `Needs verification`。
- 若無法可靠判定主要內容，須在課程中清楚記錄來源限制，不可自行選一段後假稱其為完整正文。

## 使用時替換

- `[raw 路徑]`：使用者指定的單一 converted Markdown 路徑。
- `[課程檔名]`：converted 檔名，不含 `.md`；不翻譯、縮寫或自行更名。
- `[主題目錄]`：本 workspace 固定為 `house`，包含 `source`、`converted` 與 `lesson`。

確認實際輸入位於 `house\converted` 後，由其檔名推導輸出：

```text
來源：house\converted\[課程檔名].md
輸出：house\lesson\[課程檔名]\
```

例如：

```text
house\converted\日本少子化房.md
→ house\lesson\日本少子化房\
```

## 處理範圍與資料保護

由單一 raw 生成、更新或驗證課程時，只能讀取：

- 本文件。
- 指定的單一 raw Markdown。
- 同名既有 lesson，以保留正確內容及人工修改。
- `house\lesson` 下的目錄名稱與必要檔名，只用於索引排序及連結驗證。
- `house\lesson\index.html`，只用於本次課程入口及連結驗證。

除非使用者明確要求，不得：

- 讀取或合併 `house\converted` 內其他教材的正文，或讀取 `house\source` 內的原文。
- 修改 raw、其他課程、Git staging、commit 或 push。
- 為補齊來源中未確認或未提供的人名、專名、代號、時程或未公開資訊而上網搜尋；
  已確認的公開通用術語或品牌的一般定義查核，僅依 `Key Terminology` 的限制辦理。
- 將私人、內部、受限制的連結或課程內容上傳到外部服務。
- 建立與本課無關的全域術語庫或推測其他課程內容。

即使使用者要求生成、更新或驗證 lesson，也不代表授權修改 raw。Email raw 在任何情況下都
不得由 lesson 生成流程改動；需要示範修正時，只能在固定八檔輸出的 lesson 內容中明確標示
為教學改寫，不得另建衍生檔或覆寫 raw。

來源可能含個人資料、未公開內容、程式碼、時程、存取資訊或內部網址。保留必要的隱私或 confidentiality 提醒，並依來源限制處理；不得把受限制內容改寫成可公開資訊。

## 來源保真與事實邊界

- raw 是本課內容的唯一事實依據。忠實保留講者、發言順序、日期、數字、版本、角色、決策、責任、條件、例外、風險、依賴、問題及承諾強度。
- 清楚區分：
  - `Source-stated`：來源明示。
  - `Directly inferred`：可由來源直接推得，但不是原句。
  - `General background`：協助學習的一般專業知識。
  - `Teaching example`：為語言練習新增的中性示例。
  - `Needs verification`：來源不足、字幕不清或內部名稱未確認。
- 不得把 proposal、plan、expectation、target、hope、assumption 或 hypothesis 改寫成已核准的 decision、commitment、result 或 root cause。
- 保留 `must`、`need to`、`should`、`may`、`can`、`plan to`、`expect to`、`aim to` 等強度差異。
- 未確認的縮寫不得猜測展開。可說明它在本次上下文中的功能，但須保留 `Needs verification`。
- 技術或科學問題適用時須區分現象、假設、證據、原因、處理方式、驗證結果及未知事項。
- 規則、政策、宗教、醫療、法律或合規來源須保留義務、禁止事項、適用條件、例外與觀點歸屬；教學整理不得冒充權威建議。
- converted 的中文正文是配對內容，不得誤當補充摘要而略過。中文原文轉出的英文可用於
  語言教學，但須說明是譯文，不冒充原作者的英文措辭或原始英文逐字稿。
  其他中文補充資料只用於核對概念及正式譯名，不補造來源未提供的英文引文或講者發言。

## 英文整理原則

- 對象是以繁體中文學習英文的學習者；年齡與程度依來源及使用者明示要求判定，CEFR 或年級只能標為教學推定。
- 使用自然的台灣繁體中文與來源領域常用譯法；首次出現重要概念時並列英文。
- 忠實版只修正能由上下文明確判定的拼字、標點、大小寫、斷句及基本文法錯誤，不改變語氣、責任或事實。
- 大幅度的自然化改寫放在 `Natural English`，並與來源表達並列；不得悄悄取代原話。
- 來源有值得學習的文法或句構時，必須在 `Tone, Nuance, and Grammar` 建立精簡、
  source-grounded 的文法教學；若原句不正確或不自然，可依上下文能確認的原意提出修正版。
- 每個文法修正都須並列 `Original wording`、`Teaching rewrite`、文法重點、修改理由及
  意義／語氣邊界。無法可靠判定原意時，不得擅自修正，改標 `Needs verification`。
- 正確但可更自然的句子須標為 style improvement，不得誤稱 grammar error；原句已佳時可直接
  擷取其文法價值，例如時態、分詞、被動、條件句、平行結構或修飾語。
- 不嘲諷講者、國籍或口音。非標準或即席英文應以「更自然／更清楚／更正式」說明，不把所有差異都標為文法錯誤。
- 新增例句使用符合學習者程度的中性或明確標示為虛構的情境，不得暗示來源人物、事件或組織有未記載的新狀態。
- 題目與答案分開；客觀題答案集中在 `Answer Key`。

## 每課固定輸出

每課建立以下八個檔案：

1. `[課程檔名].md`：完整課程講義。
2. `lesson.html`：與 Markdown 內容及順序一致的主要課程網頁。
3. `bilingual.html`：完整中英對照來源內容與交錯 TTS。
4. `teacher.html`：英文教學，提供雙語補充教學與學習方法的 TTS 講稿。
5. `content-summary.html`：本文導讀，提供雙語內容摘要、核心重點與理解方法的 TTS 講稿。
6. `vocabulary.html`：繁體中文核心單字與搭配老師 TTS 講稿。
7. `grammar.html`：來源導向的雙語文法與句構課程。
8. `terminology.html`：整合查詢、分類與雙語老師講解的來源領域術語頁。

每個 lesson 子目錄只保留上述一份 Markdown 與七份短檔名 HTML。不得另建獨立中文老師、
第二份文法老師或術語老師頁；相關職責已分別整合至 `teacher.html`、`grammar.html` 與
`terminology.html`。
更新既有課程時，將原 `teacher.html` 中的內容摘要、來源脈絡及理解本文的方法移至
`content-summary.html`，保留正確人工內容與來源限制；英文補充教學仍留在 `teacher.html`。
不得只複製舊講稿或改標題。同步更新導覽與 index，但僅為已生成且驗證通過的頁面新增入口。
更新舊課程時，必須將 `[課程檔名].html`、`[課程檔名]-bilingual-reading.html`、
`[課程檔名]-teacher-script.html`、`[課程檔名]-chinese-vocabulary-teacher-script.html`
及 `[課程檔名]-terminology.html` 遷移為上述短檔名，同步所有相對連結及 index，再刪除
舊長檔名。不得保留重複入口，亦不得建立名稱近似、拼字錯誤或其他規格外的課程檔案。

## Markdown 固定章節

完整講義包含一個 H1，並依下列順序使用 H2。來源確實不適用的 H3 可省略，但不得省略 H2 或虛構內容湊格式。

1. `## Source and Handling 來源與資料處理`
2. `## Course Overview 課程概覽`
3. `## Learning Goals 學習目標`
4. `## Executive Summary 重點摘要`
5. `## Domain Knowledge 領域知識`
6. `## Key Terminology 核心術語`
7. `## Key Vocabulary and Collocations 核心字彙與搭配`
8. `## Communication Functions 溝通功能`
9. `## Useful Sentence Patterns 實用句型`
10. `## Natural English 自然英文`
11. `## Tone, Nuance, and Grammar 語氣、細節與文法`
12. `## Content Notes and Recap 內容導讀與摘要`
13. `## Knowledge and Language Practice 理解與語言練習`
14. `## Real-world Application 真實情境應用`
15. `## Review 本課複習`
16. `## Answer Key 參考答案`
17. `## Bilingual Reader 中英對照全文`

### Source and Handling

- 保留來源實際提供的標題、網址、日期、場合、作者、寄件者／收件者或講者、來源類型及 raw 路徑；不適用或未提供的欄位不得猜測。
- Markdown 可用 `- **欄位名稱：** 值` 表示來源欄位；轉換至主要 HTML 時，必須將
  `**欄位名稱：**` 轉成語意化的 `<strong>欄位名稱：</strong>`，不得把 `**`、反引號或
  其他 Markdown 行內控制符當成可見正文。raw 路徑等程式碼樣式內容須轉成 `<code>`。
- raw 若提供 `Video link:`、`Video:` 或語意相同的影片網址，Markdown 的 Video 欄位固定
  使用可點擊連結，例如
  `- **Video：** [開啟原始影片 / Watch source video](https://www.youtube.com/watch?v=...)`。
  不得只輸出純文字 URL、把 URL 包在反引號中，或依賴 Markdown renderer 自動辨識網址。
- 主要 HTML 的同一欄位必須解析為原生 anchor，結構等同
  `<strong>Video：</strong> <a class="source-video-link" href="[來源的完整影片 URL]" target="_blank" rel="noopener noreferrer">開啟原始影片 / Watch source video</a>`。
  `href` 須逐字使用 raw 提供的完整 `http://` 或 `https://` URL，不得使用 `#`、空字串、
  JavaScript、按鈕事件、搜尋結果頁或自行重建的短網址。可見連結文字不得只寫成無法辨識
  目的的 `click here`。
- 任何 HTML 頁型若顯示 Video 或其他外部來源網址，都必須使用可鍵盤聚焦的原生 `<a
  href>`；一次點擊或按 Enter 即在新分頁開啟原始頁面。使用 `target="_blank"` 時固定加入
  `rel="noopener noreferrer"`，不得以 `window.open()` 或其他 JavaScript 取代正常連結。
- 標示推定 CEFR 程度，但不得說成來源提供。
- 內部材料加上僅供核准環境內學習的提醒。
- 說明 primary source content、supporting content、整理摘要、中文補充及待確認標記各自如何使用；只有來源確實為影音或會議逐字內容時才稱為 transcript。

### Course Overview 與 Learning Goals

- 概述內容範圍、目標學習者與適用學習情境，不取代原始來源。
- 提供 4 至 6 個可觀察目標，涵蓋內容理解、英文能力、術語及實際應用。

### Executive Summary

- 以簡潔文字說明目的、核心內容、結果與仍待確認事項。
- 不把摘要寫成直接引言。
- 長篇來源可先提供全局摘要，再依 episode 或主題分段。

### Domain Knowledge

依來源領域選用或自訂有意義的 H3，例如：

- `Domain Overview`
- `People, Places, and Relationships`
- `Historical and Cultural Context`
- `Scientific Concepts and Evidence`
- `Story, Theme, and Character`
- `Systems, Processes, and Design`
- `Rules, Values, and Responsibilities`
- `Decisions and Rationale`
- `Risks, Dependencies, and Tradeoffs`

每個重點回答「是什麼、彼此如何關聯、為何重要、會影響什麼」，並標示事實邊界。只加入理解來源所需且符合學習者程度的一般背景，不得新增來源人物、事件或組織的未記載事實。

### Key Terminology

整理理解本課不可缺少的領域概念與術語。表格至少包含：

| Term | Domain | Part of Speech | 繁體中文 | Plain English | 繁中定義 | Meaning in This Source | Relationship or Workflow | Example | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

要求：

- 優先選取來源實際出現或理解來源不可缺少的術語；數量依篇幅、難度與學習者程度決定，不為達到固定數量加入無關項目。
- `Domain` 使用來源內可理解且可篩選的分類，例如 `History`、`Religion`、`Transportation`、`Education`、`Relationships`、`Astronomy`、`Geography`、`Science`、`Literature`、`Business` 或 `Technology`。
- 一個術語可有多個 domain，但不得為了分類而扭曲概念。
- 中文名稱優先使用台灣常見或該領域通行譯法；必要時同列其他常見譯名。
- 一般定義須精確、簡潔，說明術語本身是什麼；來源語境另列本課的用法與指涉。
  不把單次體驗、價格、設備配置或講者評價當作通用定義，也不把品牌名稱當作一般物件的通稱。
- 公開品牌須區分公司／品牌、相關技術及本課實際指涉；相關技術只作必要的一般背景補充，
  不據此推定來源場所具備特定設備。短譯使用通行名稱，不以「品牌＋本課場所」取代品牌譯名。
- 必要時可用已確認的公開通用術語或品牌名稱查核官方或權威資料；只查一般定義、通行譯名
  與必要的概念關係，不提交 raw、課程正文、內部名稱或未公開資訊。無法確認的部分標
  `Needs verification`，不得採信其他 AI 的回答而省略查核。必要的背景參考連結可放在
  術語頁首的 `.meta`，使用原生 anchor，不納入 TTS。
- `Part of Speech` 使用完整英文詞性名稱，可附簡短繁中說明；區分名詞、名詞片語、
  專有名詞／品牌名稱、複數片語或集合名詞，依本課用法判定，不猜測未確認項目。
- 專名、古今異名、內部代號及未確認縮寫須標 `Needs verification`，不得自行展開或統一。
- 說明概念、人物、地點、事件、角色、因果或流程關係，以及容易混淆之處，而非只給字典翻譯。
- `Evidence` 只能是 `Source-stated`、`Directly inferred`、`General background` 或 `Needs verification`。
- `Evidence` 表示一般定義的依據；來源語境仍須回溯 raw，不因定義標為一般背景而失去來源
  歸屬。外部核對或補充的一般定義標 `General background`，未確認部分標 `Needs verification`，
  不以 `Source-stated` 包裝新增背景。它不代替例句來源分類。若含教學新增例句，在表格或
  術語頁前統一說明「例句可能包含教學新增示例，不代表來源逐字引文」，不逐列或逐卡重複
  聲明；需引用來源原句時，須依 canonical 的 `example_origin` 核對，不得誤標。
- Markdown、主要 HTML、`terminology.html` 與術語老師稿的術語項目及順序必須一致。

### Key Vocabulary and Collocations

依來源篇幅與學習者程度選取可重複使用的單字、片語及搭配。為了讓 Markdown、主要 HTML
及鎖定版型的 `vocabulary.html` 可逐項同步，表格至少包含：

| Word or Phrase | Section | Part of Speech | Meaning in Context | 繁體中文 | Pronunciation Tip | Collocation or Pattern | Example | 例句繁中 | Memory Tip |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

- 詞性使用完整英文名稱。
- `Section` 使用來源既有章節；來源沒有章節時，使用可確認的主題／情節段落名稱，不得虛構。
- 優先保留完整搭配，不把片語拆到失去來源語境。
- 簡單英文定義不得比目標字更難。
- `Pronunciation Tip` 只提供可可靠確認、適合學習者的重音、音節或常見字母發音提示；
  不得自造 IPA。無可靠提示時使用簡短的重音或音節說明，不可留空後在 HTML 猜測。
- `例句繁中` 必須只翻譯該列英文例句；`Memory Tip` 使用自然台灣繁中，連結字形、搭配、
  語意或本課情境，不得使用全課共用的空泛句型。
- Markdown、主要 HTML 與單字老師稿的項目及順序必須一致。

### Communication Functions

依來源類型與學習者需求整理實際溝通功能，例如：

- greeting, introducing, and building relationships
- asking for clarification or confirmation
- describing people, places, events, processes, or evidence
- narrating sequence, cause and effect, comparison, or change
- making a request or offering help
- agreeing, disagreeing, correcting, refusing, or setting a boundary
- expressing feelings, beliefs, plans, expectations, duties, possibility, or uncertainty
- explaining a rule, concept, opinion, conclusion, or next step

每個功能提供來源例句、繁中說明、語氣及可重複使用的中性句型。

### Useful Sentence Patterns

- 使用可替換槽位，例如 `Could you explain why [event]?` 或 `If [condition], [result].`。
- 說明句型適用的口語、書面、敘事、學術、生活或專業情境。
- 至少包含一個較直接與一個較委婉的版本，並解釋差異。
- 不使用新增句型捏造來源中的人物、事件、立場或狀態。

### Natural English

- 只挑選值得學習的非標準、不自然、冗長或不夠清楚的來源表達。
- 使用 `Source wording | Natural version | Why | Tone` 對照。
- `Source wording` 必須從不可變 raw 忠實引用；`Natural version` 只存在於 lesson，不得回寫 raw。
- 若原句已自然，保留並說明其優點，不為改寫而改寫。
- 不改變事實、時態、角色、責任、確定性、禮貌程度及領域含義。

### Tone, Nuance, and Grammar

- 文法只服務於來源內容與實際溝通，不做與來源無關的完整文法課；但來源中有實用重點時不得只列
  語氣差異而省略文法教學。
- 優先說明 modality、hedging、certainty、politeness、active/passive voice、tense、articles、prepositions 及常見搭配。
- 明確區分「不合文法」、「意思不同」、「較不自然」、「較直接／委婉」及「正式／口語」。
- 視來源需要加入 `Source-grounded Grammar Clinic` H3。為支援鎖定版型的文法教學與
  「快速檢核」，表格整理
  `Grammar point | Source example | Rule | Source analysis | Quick check A | Quick check B | Answer | Answer reason | Teaching rewrite（適用時） | Meaning / tone boundary`。
- `Quick check A` 與 `Quick check B` 必須只有一個符合該文法重點的最佳答案；錯誤選項只測
  本點規則，不得因拼字、人物、事實或其他無關線索而明顯錯誤。`Answer` 固定為 A 或 B，
  `Answer reason` 以自然繁中指出決定答案的規則，不只重複選項。
- 優先挑選 3 至 10 個可遷移到口語、寫作、閱讀或來源領域溝通的重點；品質優先。
  來源不足 3 個可靠重點時，使用實際數量並說明限制，不得虛構錯誤或加入來源無關文法。
- 可教授 noun stack、verb pattern、participial clause、modifier attachment、parallelism、
  article、preposition、tense/aspect、active/passive voice、conditionals、modality 與 hedging，
  但只選來源實際支持者。
- 修正版必須保留人物、事實、時態、條件、責任、數字、領域含義與承諾強度。若同時有多個
  自然版本，說明正式度或語氣差異，不武斷宣稱只有一個正解。
- 英文教學頁須至少用一個 `teaching-point` 概述本課最重要文法的學習或應用方法，並連結
  `grammar.html`，不重複完整講解；完整文法教學放在
  唯一的 `grammar.html`，不得另建第二份 grammar teacher 檔。

### Content Notes and Recap

依來源類型選用：

- 故事／小說／戲劇：Characters、Setting、Plot、Conflict、Theme、Point of View、Uncertainty。
- 教材／演講／科普：Key Concepts、Definitions、Examples、Evidence、Questions、Review。
- 歷史／人文／宗教：Time and Place、People、Source Perspective、Beliefs、Events、Interpretive Boundaries。
- 地理／交通／自然科學：Location、System、Process、Cause and Effect、Data、Constraints、Unknowns。
- Email／公告／memo：Purpose、Key Message、Requests、Required Actions、Audience、Deadline、Tone、Follow-up。
- 會議／聊天：Topics、Confirmed Findings、Decisions、Action Items、Risks、Blockers、Dependencies、Open Questions。
- 文章／訪談／評論：Claims、Evidence、Quoted Views、Perspective、Implications。
- Specification／report／政策：Purpose、Scope、Requirements、Findings、Recommendations、Constraints、Open Questions。

只有來源確實包含任務或決策時才建立 `Action Items` 或 `Decisions`；未提供的欄位填 `Not specified`，不可猜測。不得把所有來源強制整理成工作摘要。

### Practice、Application、Review 與 Answer Key

- 題目可涵蓋內容理解、順序、因果、觀點、術語配對、語氣強度、句型選擇、文法辨識／修正及情境應用，並依學習者程度調整。
- 每題提供足夠上下文；若多個答案合理，列出全部可接受答案及差異。
- 適用時要求區分事實、觀點、推論、假設、計畫、期望、決定或承諾；不得為湊題型強加來源沒有的類別。
- 科學或技術來源適用時練習區分現象、原因、證據、處理方式及驗證。
- Real-world Application 使用明確標示為虛構或中性的情境，練習把本課英文遷移到生活、學校、社交、研究、旅行、工作或其他適用場景。
- Review 摘要本課知識、術語、句型、語氣及自我檢查。
- Answer Key 逐題對應；開放題標示 `Sample answer`。

### Bilingual Reader

- 放在 Markdown 最後，以 raw 中辨識出的 primary source content 為基準，適用於 email、文件、文章、公告、政策、chat、transcript 及其他文字；完整依原順序呈現，不以摘要取代。
- 從 converted 既有的中文／英文配對建立 Reader，只將顯示順序改為英文／繁體中文。
  每個英文完整句對應同組中文，保留配對邊界，不再次翻譯、合併不同配對或重複加入中文正文。
- 來源為 transcript 或 captions 時，只在 lesson 衍生內容中合併可由上下文明確確認的字幕
  斷行，整理成完整句或合理短段；不得回寫 raw、自然化改寫原話、猜測講者或移除有意義的
  停頓、音效及不確定標記。
- 優先沿用來源既有章節；沒有章節時，才依可確認的主題、事件、場景或說話目的轉折建立
  連續 H3 小節。小節須完整涵蓋來源且不得重疊、重排或遺漏；無法可靠命名時使用帶順序
  編號的中性 `Story Segment`，不得自行補寫情節。
- 每個 H3 小節內的 Markdown 配對固定依序使用 `#### Pair N`、`**English**`、英文正文、
  `**繁體中文**`、繁中正文；每個小節的 N 都從 1 連續編號。這份結構須與主要 HTML 及
  `bilingual.html` 的小節、配對編號、文字及順序逐組一致。
- recap、digest、summary 或 learning notes 與主要內容重複時，不重複加入 Reader；前者放在工作摘要，primary source content 作為 Reader 基準。
- 每個英文完整句或合理短段後立即放對應繁體中文，不以一段中文概括多段英文。
- 保留適用的 email header、稱呼、結尾、作者、講者、訊息標籤、episode、小節、清單、表格、程式碼、命令、錯誤訊息、網址及重要格式。
- 程式碼、log、URL 及命令不翻譯其內容；在相鄰中文說明其用途。
- 未知講者使用來源既有標記；不得猜測姓名。
- `[Needs verification]`、`[unclear]`、`Source gap` 及不確定拼字不得在翻譯中消失。

## terminology.html 專用規格

`terminology.html` 同時是本課的中英文領域術語查詢頁及雙語術語老師頁，不是單純複製 Markdown 表格，也不另建術語老師檔。

必須包含：

- 依頁面骨架契約輸出的完整 H1，不得縮短、改寫來源標題或改回
  `整合術語／Terminology`；另提供來源、適用的資料或內容提醒及術語總數。
- 搜尋欄，可搜尋英文、繁中、定義、例句及 domain。
- 固定的 Domain 篩選按鈕群組、預設啟用的 `All`、目前結果數及無結果訊息。
- 每個術語的英文名稱、繁中名稱、domain、繁中定義、繁中來源語境、繁中關係／流程、
  英文例句、繁中例句翻譯、完整詞性及繁中 Evidence。
- `定義` 使用精確、簡潔的一般定義；`來源語境` 保留老師或來源對當次場景的描述。
  兩者不得互相替代；品牌及其技術的背景補充依 `Key Terminology` 規則處理。
- 每個 `.term` 只呈現一份整合式內容；不得另建完整英文說明，也不得把定義、來源語境、
  流程關係或 Evidence 逐欄翻成英文後重複顯示。
- 欄位順序固定為 `術語`、`定義`、`來源語境`、`流程關係`、`例句`。每個欄位使用
  `.term-field[data-field]`，包含獨立 `.field-label` 與 `.field-value`；不得把所有欄位
  串成單一長段落，也不得用 `<br>` 堆疊版面。四個非術語欄位必須使用鎖定的緊湊 margin
  與欄位內容 padding；不得因通用 `.speech-unit` 樣式在「定義」與「來源語境」或其他
  相鄰欄位間產生看似空白行的額外垂直間距。
- `術語` 欄同一行顯示英文名稱、可見分隔符 `｜` 及繁中名稱；`例句` 欄先顯示英文例句，
  下一行顯示其繁中翻譯。其餘三欄只使用自然、完整的繁體中文；必要的角色名、產品名、
  縮寫、未確認專名與技術符號可保留英文。
- `例句` 欄已有「例句：」標籤，英文句子正文及其 `data-text` 不得再加上
  `Teaching example:` 前綴，繁中翻譯也不得重複加入對應標籤；兩個朗讀單元只包含句子本身。
  保留 Evidence 分類與五欄結構，不逐卡附加教學示例聲明；教學新增例句不得標成
  來源逐字引文。統一說明依 `Key Terminology` 規則置於卡片清單前，不加入 TTS。
- Evidence 固定使用 `來源明示`、`直接推論`、`一般背景` 或 `待查證`。沒有資料的選用欄位
  不得虛構內容；不確定資訊須保留 `待查證` 或其他來源既有的不確定標記。
- domain、詞性與「證據：[繁中分類]」合併於 H2 後唯一的 `.term-meta`，使用鎖定的小字、
  緊湊行高與透明背景。三者均可見、可搜尋，但不含 `.speech-unit`、`data-lang` 或
  `data-text`，不得以 `hidden` 或 `aria-hidden` 隱藏。不建立獨立 `data-field="evidence"`、
  `.evidence-value` 或第六個欄位；證據移至小字列後，七個朗讀單元的內容職責與順序不變。
- 每張卡固定有七個 `.speech-unit[data-lang][data-text]`：英文術語、繁中術語、繁中定義、
  繁中來源語境、繁中流程關係、英文例句及繁中例句翻譯。英文單元使用
  `en-US`，繁中單元使用 `zh-TW`；每個 `data-text` 必須逐字等於該單元可見內容。
  `.field-label`、純視覺分隔符與整個 `.term-meta` 不加入 TTS；播放、自然接續與前後段導覽
  均須排除分類、詞性與證據，不得在送入語音引擎後才過濾。
- 適用時可把 `Often confused with`、縮寫、同義詞或相關術語自然整合進既有的繁中定義或
  流程關係，不得新增第六個欄位或另一份英文說明。
- 提供上一筆、下一筆、播放／暫停、停止、語速及語音選擇。
- 不使用假的 IPA。若無可靠發音資料，只使用瀏覽器英文 TTS。
- 搜尋及篩選使用內嵌 JavaScript，無 JavaScript 時仍可閱讀全部內容。
- 可列印、鍵盤操作、焦點清楚、色彩對比足夠，並支援 320px、390px 及桌面寬度。

搜尋工具固定使用下列 DOM；只能依本課實際 domain 重複篩選按鈕，按卡片首次出現順序
去重排列。`All` 固定為第一個按鈕，初始 `aria-pressed="true"`；其餘按鈕初始為
`false`。搜尋欄、篩選按鈕、結果狀態及無結果訊息必須存在於初始 HTML，不得由 JavaScript
動態建立：

```html
<section class="tools" aria-label="術語搜尋與篩選">
  <div class="search-field">
    <label for="termSearch">搜尋術語</label>
    <input id="termSearch" type="search"
      placeholder="搜尋英文、繁中、定義、例句或 domain">
  </div>
  <div class="filters" aria-label="Domain 篩選">
    <button type="button" data-domain="All" aria-pressed="true">All</button>
    <button type="button" data-domain="[domain]" aria-pressed="false">[domain]</button>
  </div>
  <p id="resultCount" role="status" aria-live="polite">[N] 個結果</p>
  <p id="noResults" role="status" hidden>沒有符合的術語。</p>
</section>
```

搜尋與篩選腳本固定遵守：

- `activeDomain` 初始為 `All`。點擊篩選按鈕時，只把該按鈕設為
  `aria-pressed="true"`，其餘設為 `false`，再重新套用篩選。
- 搜尋字串使用 `termSearch.value.trim().toLocaleLowerCase()`；每張卡以完整
  `textContent.toLocaleLowerCase()` 比對，因此英文、繁中、定義、來源語境、流程關係、
  Evidence、例句及可見 domain 均可搜尋。
- 卡片同時符合目前 domain（`All` 或 `card.dataset.domain === activeDomain`）及搜尋字串時
  才顯示。隱藏只可切換 `.hidden`，不得移除、重建、重排卡片或改寫內容。
- `#resultCount` 每次更新為 `[可見數] 個結果`；可見數為 0 時顯示 `#noResults`，否則隱藏。
  初始 HTML 的結果數須等於實際 `.term` 數量，載入後立即可讀，不得先顯示空白狀態。
- 搜尋／篩選腳本不得改寫 compact player 的朗讀單元清單、目前索引、控制 ID 或事件；
  TTS 仍依原始 DOM 順序巡覽全部術語內容。

標準內容如下；方括號內容須依本課來源替換：

```text
術語：[English term]｜[繁中術語]
[domain] · [完整詞性與繁中說明] · 證據：[來源明示／直接推論／一般背景／待查證]
定義：[精確、簡潔的一般繁中定義]
來源語境：[本課中的繁中來源語意]
流程關係：[與其他概念、事件、角色、地點或流程的繁中關係]
例句：[英文來源句或教學例句，僅含句子正文，不加 Teaching example: 前綴]
      [繁中例句翻譯]
```

`術語` 與 `例句` 是雙語欄位；`定義`、`來源語境`、`流程關係` 是繁中欄位。
domain、詞性與繁中證據分類是共用的小字資訊列，不屬於五個欄位，也不朗讀。
不得在卡片下方再輸出一份英文 Definition、Source Context、Workflow Relationship 或
Evidence。

每張 `.term` 固定使用下列 DOM；只能替換方括號內容及重複卡片，不得改變欄位數量、
class、`data-field`、語言屬性或順序：

```html
<article class="term" data-term="[English term]" data-domain="[domain]">
  <h2 class="term-field" data-field="term">
    <span class="field-label">術語：</span>
    <span class="field-value bilingual-value">
      <span class="speech-unit" lang="en" data-lang="en-US" data-text="[English term]">[English term]</span>
      <span class="separator" aria-hidden="true">｜</span>
      <span class="speech-unit" lang="zh-Hant" data-lang="zh-TW" data-text="[繁中術語]">[繁中術語]</span>
    </span>
  </h2>
  <p class="term-meta">[domain] · [完整詞性與繁中說明] · 證據：[繁中 Evidence]</p>
  <div class="term-field" data-field="definition">
    <span class="field-label">定義：</span>
    <div class="field-value speech-unit" lang="zh-Hant" data-lang="zh-TW" data-text="[繁中定義]">[繁中定義]</div>
  </div>
  <div class="term-field" data-field="source-context">
    <span class="field-label">來源語境：</span>
    <div class="field-value speech-unit" lang="zh-Hant" data-lang="zh-TW" data-text="[繁中來源語境]">[繁中來源語境]</div>
  </div>
  <div class="term-field" data-field="workflow-relationship">
    <span class="field-label">流程關係：</span>
    <div class="field-value speech-unit" lang="zh-Hant" data-lang="zh-TW" data-text="[繁中流程關係]">[繁中流程關係]</div>
  </div>
  <div class="term-field" data-field="example">
    <span class="field-label">例句：</span>
    <div class="field-value example-pair">
      <div class="speech-unit" lang="en" data-lang="en-US" data-text="[English example]">[English example]</div>
      <div class="speech-unit" lang="zh-Hant" data-lang="zh-TW" data-text="[繁中例句翻譯]">[繁中例句翻譯]</div>
    </div>
  </div>
</article>
```

單篇課程生成時只建立該課術語頁。只有使用者明確要求彙整多課術語時，才可讀取指定範圍內既有 `terminology.html` 或課程 Markdown，去重後建立獨立總表；不得自行掃描所有 raw。

## HTML 與 TTS 共通規格

- HTML 原始碼是可維護的成品，不是壓縮產物。新增或修改的 HTML 必須使用一致縮排與合理
  換行，讓 `html`、`head`、`body`、主要容器及巢狀區塊的結構可直接閱讀；不得將整份文件、
  主要容器或大量相鄰元素寫在同一行。
- 內嵌 CSS 與 JavaScript 也必須保持可讀結構。CSS selector、規則區塊及 declaration 須有
  清楚分行與縮排；JavaScript 的函式、條件、迴圈、物件及事件處理流程不得壓縮成單行。
- HTML、CSS 與 JavaScript 的每層巢狀結構統一增加兩個空格，不使用 tab。開始與結束標籤
  對齊；主要容器、卡片、欄位、控制項及 script block 各自換行。CSS 每個規則區塊的
  `{`、每個 declaration 及 `}` 分行；JavaScript 每個 statement 分行。短文字元素可保持
  單行，但不得把兩個同層卡片、兩個 `.term-field`、兩個 CSS 規則或兩個控制流程擠在同一行。
- 格式須支援人工審查與精準 patch。更新既有檔案時沿用其合理格式，只格式化本次目標檔案
  及達成本規格所需的範圍，不得為偏好差異重排無關程式碼或製造純格式 diff。若目標 HTML
  原為單行或含壓縮區塊，須在修改時將該檔案整理成符合本節的可讀格式。
- 格式化不得改變可見文字、空白語意、DOM 順序、CSS cascade、JavaScript、TTS、
  可存取性或頁面行為。`pre`、`textarea`、程式碼範例、字串、template literal 及其他
  空白敏感內容保持原樣；不得為縮短行數拆改 text node 或朗讀內容。
- 優先沿用專案既有格式。可使用本機 formatter，但必須明示 UTF-8、限制於目標檔案、先
  檢查 diff 並重新執行結構與功能驗證；不得只為格式化新增 dependency 或外部服務。
- 所有 HTML 都是可離線開啟的 UTF-8 獨立檔案，使用 viewport、語意化 HTML、內嵌 CSS；互動頁使用內嵌 JavaScript。
- 不依賴 CDN、外部字型、框架、分析服務或網路資源才能顯示及操作。
- 主要 HTML 完整保留 Markdown 的 H1、17 個 H2、內容、表格、題目、答案及 Reader 順序。
- Markdown 轉成 HTML 時須解析標題、粗體、斜體、行內程式碼、連結、清單及表格等行內與
  區塊語法；可見正文不得殘留用作格式控制的 `**`、`__`、反引號或 Markdown 連結語法。
  不得只轉換外層 `<li>`、`<p>` 或 `<td>` 而把其中的 Markdown 原樣塞入 HTML。
- 所有 `id` 唯一；長網址、長字、程式碼、log、表格及控制列不得造成整頁水平溢出。
- 連結需可辨識；外部或內部網址不得自動傳送內容到第三方。
- TTS 使用 Web Speech API；英文 `en-US`，繁中 `zh-TW`。
- 每個朗讀單元有明確容器、`data-lang` 與純文字，不從整頁 `textContent` 猜測。
- Bilingual Reader、英文教學與本文導讀提供播放、暫停／繼續、停止、上一段、下一段、上 10 段、
  下 10 段、朗讀模式、語音、語速、進度及狀態。中文單字老師、文法頁與術語頁使用鎖定的
  compact player：含上／下 10 段的七個按鈕、兩種語音、語速、進度、狀態與錯誤訊息。所有 TTS 頁依
  「TTS 與控制器鎖定契約」於播放時收合整個設定區域，導覽或設定變更後待播、暫停、
  停止、播畢或錯誤時展開。
- 所有 TTS 頁的 transport 按鈕都須形成單一且佔滿第一列的 `.transport` 群組。完整播放器的
  欄位順序固定為 `朗讀語言`、`中文語音`、`English voice`、`語速`；compact player 的
  欄位順序固定為 `中文語音`、`英文語音`、`語速`。每個欄位都使用 `.control-field`
  包住 label 與對應 control，三種 compact 頁面的播放器 DOM 與播放器 CSS 必須相同；
  三種 compact 頁面的播放器 JavaScript 也必須相同。
- 所有 TTS 頁的 transport button、朗讀模式、語音選單與語速滑桿依共享鎖定 CSS 使用相同
  的緊湊高度。所有頁型的響應式換行均不得造成水平溢出、文字截斷、遮擋或無法鍵盤聚焦。
- 所有 TTS 頁的 `上 10 段`／`下 10 段` 分別使用 `#previous10`／`#next10`。
  跳轉依目前 eligible units 的索引減／加 10，先取消目前語音，再定位並高亮目標單元，
  維持待播，按 `play` 才從該單元開頭朗讀；
  到達首尾時將目標索引限制在第一或最後一個可朗讀單元，不得越界。有 `readingMode`
  時僅計算目前模式允許的單元；compact 頁依原有朗讀順序計算全部單元，術語頁仍排除
  分類、詞性與證據小字列。
- 除三種 compact 頁外，同時包含英文與繁中的 TTS 頁必須提供
  `#readingMode`，選項和值依序為
  `中英皆讀 (both)`、`只讀中文 (zh-TW)`、`只讀英文 (en-US)`；切換後立即停止目前語音，
  取消佇列，將目前段落設為該模式的第一個可朗讀單元，定位並高亮、顯示待播而不自動播放。
- 所有 TTS 頁變更 `zhVoice`、`enVoice` 或 `rate` 時，立即停止並取消佇列，維持目前段落、
  定位與待播高亮；`rate` 的 `input` 即時生效並更新 `rateValue`。只有按 `play` 才依新設定
  從該單元開頭朗讀，不 resume 舊暫停內容；控制流程與空清單處理依共用設定變更契約。
- 有 reading mode 的頁面，其播放、上一段、下一段及自然接續只能巡覽目前模式允許的語言；
  進度分子與分母也只計算該模式的可朗讀單元。語言模式只影響 TTS，不得隱藏頁面內容。
  中文單字老師固定巡覽全部四語序單元；文法頁固定巡覽導讀、各點五單元及總結。
- 除三種 compact 頁外，顯示與前後段導覽仍以 `.speech-unit` 為單位，但送入 Web Speech API 前，
  須先依 `. ! ? 。 ！ ？` 句末符號切成短 utterance；超過 140 字元的單句再優先於逗號、
  分號、冒號或空白處切分。不得把含多句的長段落交給單一 `SpeechSynthesisUtterance`。
- 中文單字老師、文法頁與術語頁嚴格依共享參考控制器，實際播放時每個 `.speech-unit`
  直接建立一個 `SpeechSynthesisUtterance(el.dataset.text)`；四個手動導覽按鈕 clamp 後
  只定位並高亮，語音／語速變更則維持目前段落並待播，兩者均不建立語音。有效播放自然
  結束後依 DOM 朗讀下一單元，不加入句級拆分或額外佇列。
- 控制器須在模組層保留目前 `SpeechSynthesisUtterance` 的強引用，直至其 `end` 或真正
  `error`，避免瀏覽器回收區域變數導致朗讀中途停止。單一顯示單元的短 utterance 全部完成後，
  才可依目前語言模式接續下一顯示單元。
- `>>` 或 `»` 等開頭字幕發言標記若只表示字幕換人、沒有可靠講者身分或教學功能，須保留
  在 raw，但從衍生課程中每個單元的開頭移除，包括 Markdown、主要 HTML、Reader 畫面與
  `data-text`。不得因此虛構講者、合併發言或移除正文中的標點、停頓及其他內容。
- TTS 控制器仍須在建立朗讀副本時防禦性略過殘留的開頭 `>>`／`»`，避免舊課程或其他來源
  的非語言標記被語音引擎誤讀；此正規化不得改寫 DOM 或 raw。
- 手動切換段落時取消目前語音，只定位並高亮目標段，待使用者按 `play` 才朗讀；
  有效播放流程中自然播畢仍自動接續，最後一段結束則完成播放。
- `canceled` 與 `interrupted` 不顯示成錯誤；真正錯誤須清楚顯示。
- 目前段落須醒目顯示；Bilingual Reader、英文教學與本文導讀依共用內容群組追蹤規則捲動，其他頁型
  依各自鎖定契約將目前段落或所在卡片捲動至可見範圍。
- 不朗讀按鈕、導覽、答案、狀態、CSS、JavaScript、URL、程式碼或 log。
- DOM 綁定使用明確查詢，不依賴元素 `id` 自動成為全域變數。

## 各 TTS 頁內容

### Bilingual Reading

- 完整呈現 Bilingual Reader。
- 每組先朗讀一個英文句或合理短段，再朗讀只對應該單元的繁中翻譯。預設以來源的一個完整
  句子或一個不可再分的短發言為一組；不得按固定數量把多個連續句子批次合併成大段。只有
  同一發言中拆開會破壞語意的片段才可合併，且須保持短小。
- 對對話型 transcript 統計英文 pair 的句末標點數：中位數不得超過 2，至少 90% 的 pair
  不得超過 2 個完整句，且不得出現系統性的固定句數群組。超過 2 句者須確認是同一發言中
  語意不可分的短段；超過 6 句或 280 個英文字符視為疑似過度合併，須回查來源後拆分，
  或記錄無法拆分的具體理由。這些數值只用於偵測過度合併，不得把完整語意硬切到上限。
- 英文與中文顯示單元及 TTS 單元必須一對一、順序一致。
- 中文標點與英文不一致時，依語意邊界拆分既有譯文，使兩語一對一；不得為了湊數省略、
  重排或新增意思，也不得把一個中文長句複製給多個英文單元。
- 每個來源小節使用可見雙語 H2；每組卡片顯示不朗讀的順序編號，並以穩定且唯一的
  `data-group-index` 與 `data-pair-id` 對應該小節及配對。
- 每個 `.bilingual-pair` 鎖定為：pair number、直接含英文純文字的 `en-US` `.speech-unit`、
  直接含繁中純文字的 `zh-TW` `.speech-unit`；兩個 speech unit 不建立內層 `<p>`。每個小節
  的 pair number 從 1 連續編號，不重複、不跳號。
- 標題、講者標籤、程式碼、log 及 URL 可顯示但不加入 TTS。

### English Teacher（teacher.html）

- 聚焦可遷移的英文學習重點、學習方法、句型、發音、語氣與實際應用，補充其他課程頁面
  未涵蓋的內容；以本課英文為依據，不擴寫成與來源無關的通用教材。
- 依序安排英文學習目標、補充教學、練習方法、情境應用與學習總結；內容數量依來源調整，
  不為湊數生成。發音可教重音、連音、節奏等有依據的特徵，不臆測來源人物的實際發音。
- 字彙、文法及術語的完整教學仍分別放在對應專用頁；此頁只補充學習策略、易錯處或應用，
  並提供相關頁面連結，不複製整份卡片或講解。來源有值得學習的文法時，依文法規格提供概述。
- 本文摘要、情節／論點整理、來源脈絡與理解本文的方法放在 `content-summary.html`；
  此頁僅引用補充英文教學所需的最少上下文，不重述完整本文。

### Content Summary（content-summary.html）

- 聚焦本文本身：提供忠實摘要、核心重點與適合本文的理解及學習方法，不取代完整 Reader。
- 依來源類型安排閱讀目標、必要脈絡、內容摘要、核心重點、理解引導及回顧；可使用事件
  順序、因果、人物關係、論點與證據或概念關係等適合來源的組織方式，不強加不適用的欄位。
- 理解引導應說明如何掌握本文，例如追蹤轉折、辨識主張與證據、釐清指代或自我檢查理解，
  而非提供與本文無關的通用英文技巧。完整字彙、文法、術語教學仍留在各專用頁。
- 從 canonical 的摘要、內容筆記與 Reader 整理導讀；每個重點均須可回溯來源。明確區分
  來源明示、直接推論、一般背景與待查證資訊，不補寫情節、動機、結論或未提供的事實。
- 評論須自然銜接上下文，聚焦敘事節奏、人物選擇、衝突轉折與理解方法，避免突兀的說教、
  道德評斷或反覆免責聲明。例如以「故事以幾個快速片段帶過創業歷程」說明敘事壓縮，
  不在段尾生硬追加「不是人人創業都會成功的證明」。
- 角色的鼓舞、宣言與願望以其劇中作用說明，例如「這句話傳達鼓舞」或「表達幫助他人的
  願望」，自然區分已敘述事件與未來期望，不改寫成已實現的成果或普遍保證，也不臆測講者。
- 僅供編輯參考的提醒（如勿概括成功結果、勿將逮捕補成判刑、勿補造缺字或講者資訊）
  放入 `editorial_notes`，輸出為 `.speech-unit` 外的 HTML 註解，不顯示、不朗讀。
  若缺字、講者不明或其他來源疑義會影響讀者理解當前敘述，仍須在雙語正文中簡潔交代；
  不影響導讀的細節可留在編輯註解，不補造內容，也不刪除 Reader 的原有不確定標記。
- 本頁是固定輸出，不等同 workspace 選用的 `work-summary.html`，也不取代其契約。

### 英文教學與本文導讀共用內容結構

- 兩頁分別使用 `teacher_points` 與 `content_summary_points`，沿用同一完整播放器、
  卡片樣式及控制器，不因內容職責不同而建立另一套 UI。
- 每個主題只建立一個 `.teaching-point[data-point-id]`，依序安排一次自然繁體中文說明及
  一次自然英文說明。兩語須傳達一致的核心重點與事實邊界，中文可補充適合繁中學習者的
  解釋，但不得與英文矛盾；不另建中文老師檔。
- 每個主題保留兩個 `.speech-unit`：繁中單元內依序放中文主題標題與中文說明，英文單元內
  依序放英文主題標題與英文說明。標題使用語意化 heading，與說明之間保留文字空白；
  `data-text` 逐字等於該單元的 `textContent.trim()`，不得漏掉標題或另建重複標題單元。
  中英皆讀時依序朗讀中文標題、中文說明、英文標題、英文說明；單語模式只朗讀對應語言。
  標題與說明共用單元的高亮、進度及前後段導覽，不改動共用播放器；頁面 H1 與導覽仍不朗讀。
- 相同事實、例句或練習不可在同一頁反覆講解。英文使用 `en-US`，繁中使用 `zh-TW`。
- 來源行號、Reader pair ID、待確認問題 ID、raw 路徑與雜湊只作 canonical 內部對應及驗證。
  兩頁不輸出逐點或頁尾的來源註記、來源清單或 Qxx 連結，頁首也不顯示 raw 路徑與雜湊；
  既有原始來源連結仍保留為原生 `<a>`。影響讀者理解的來源疑義與限制在雙語正文中自然
  說明，不以來源註記或問題 ID 取代；本文導讀中僅供編輯參考的提醒依前節移至 HTML 註解。
  本條僅限制兩頁的呈現，不移除 canonical 的來源對應，
  也不改動主要課程的 `Source and Handling`、Reader 識別屬性或來源完整性驗證。
- 不逐字重讀完整 Reader、單字表、術語表或 Answer Key。

### 課程重點小節標題朗讀（work-summary.html）

本節僅適用於 workspace 專用規格要求的 `work-summary.html`，不新增預設輸出檔案。
本頁沿用完整播放器；以下標題朗讀規則不適用於原文 Bilingual Reader。

- 每個 `.bilingual-section` 的小節標題須含英文與繁中，置於該節正文配對之前。
  使用獨立 `.bilingual-pair` 包住一個 H2；H2 內依序放英文 `en-US` 與繁中 `zh-TW`
  `.speech-unit`，各自只含對應語言的可見標題文字，`data-text` 等於其
  `textContent.trim()`。語言間的分隔符置於朗讀單元外，不朗讀，也不另留重複標題。
- 標題 pair 與其兩個單元均保留所屬小節的 `data-group-index`，並共用穩定且唯一的
  `data-pair-id`，使用 `summary-heading-` 前綴，與正文及原文 pair ID 分開。
  標題不顯示 Pair 編號，不占正文的連續編號，也不得偽造來源 pair 對應。
- 中英皆讀依序播放英文標題、中文標題，再依序朗讀該節的英文／中文正文配對；
  單語模式只播放對應語言的標題與正文。標題單元參與前後段、±10、進度、高亮與
  群組捲動，不修改共用播放器或把標題硬塞進第一段正文；每節只朗讀一次標題。
- 頁面 H1、提示、來源位置、問題 ID、導覽、圖片、完整 URL、code 與 log 仍不加入 TTS。
  正文配對、翻譯、來源對應及順序均保持不變。

### Chinese Vocabulary Teacher Script

- 與 Markdown `Key Vocabulary and Collocations` 的項目及順序完全一致。
- 每項包含英文名稱、來源 section、完整詞性、中文意思、本課語意、可靠發音提示、搭配、
  一個英文例句、該例句的繁中翻譯及記憶提示。
- header 只放課程標題與返回導覽；不得加入頁面說明、項目數或其他參考頁沒有的區塊。
  每張 `.vocabulary-item[data-vocab-id][data-section][data-term]` 依序放：
  1. 同時作為第一個英文 `.speech-unit` 的 H2 英文字詞；
  2. `.meta`，內容固定為 `Section · part of speech · 繁中短譯`；
  3. `.vocab-details` 清單，依序為 `發音提示：` 與 `常見搭配：`；
  4. 其餘三個固定順序的 `.speech-unit`：繁中五欄、英文例句、繁中例句翻譯。
  H2 加入 TTS；meta、發音提示及搭配清單不加入 TTS。不得為朗讀英文字詞而在 H2 後
  複製第二份相同文字。
- `vocabulary.html` 原始碼必須展開供人工審查：`head`、`style`、`body`、控制列、
  `header`、`main`、每張 `article.vocabulary-item` 及其 H2、meta、details 與三個後續
  speech unit 都使用一致的兩空格巢狀縮排及合理換行。不同卡片或上述主要子區塊不得共用
  同一行；CSS 規則與 JavaScript 控制流程依「HTML 與 TTS 共通規格」分行。為維持
  `data-text === textContent.trim()` 而保留在同一行的單一 speech unit 內部文字不視為
  壓縮，但整份文件、整張卡片或多個 speech unit 不得因此合併成單行。
- 頁面內容與卡片 DOM 鎖定下列骨架。`body` 的第一個子元素必須逐字使用「TTS 與控制器
  鎖定契約」定義的 compact `.controls` DOM；下列註解不是可輸出的元素。只能替換
  方括號內容及重複單字卡，不得改 class、ID、元素順序或增加 wrapper：

```html
<body>
  <!-- 逐字插入本文件定義的 compact .controls DOM；不得輸出本註解。 -->
  <header>
    <h1>[課程標題] | 中文單字老師</h1>
    <p>[返回主要課程] · [返回課程索引]</p>
  </header>
  <main>
    <article class="vocabulary-item" data-vocab-id="[穩定唯一 ID]" data-section="[Section]" data-term="[term]">
      <h2 class="speech-unit" data-lang="en-US" data-text="[term]">[term]</h2>
      <p class="meta">[Section] · [part of speech] · [繁中短譯]</p>
      <ul class="vocab-details">
        <li><strong>發音提示：</strong>[可靠發音提示]</li>
        <li><strong>常見搭配：</strong>[collocation]</li>
      </ul>
      <div class="speech-unit" data-lang="zh-TW" data-text="[五欄可見文字的 textContent.trim()]"><div class="vocab-explanation"><p class="explanation-row term"><strong>單字：</strong><span>[term]。</span></p><p class="explanation-row part-of-speech"><strong>詞性：</strong><span>[part of speech]。</span></p><p class="explanation-row meaning"><strong>中文：</strong><span>[繁中短譯]。</span></p><p class="explanation-row lesson-meaning"><strong>本課意思：</strong><span>[本課專屬繁中說明]。</span></p><p class="explanation-row memory"><strong>記憶提示：</strong><span>[繁中記憶提示]。</span></p></div></div>
      <div class="speech-unit" data-lang="en-US" data-text="[English example]"><p lang="en">[English example]</p></div>
      <div class="speech-unit example-translation" data-lang="zh-TW" data-text="[例句繁中]"><p lang="zh-Hant">[例句繁中]</p></div>
    </article>
  </main>
</body>
```

- 第一個英文 `.speech-unit[data-lang="en-US"]` 必須是卡片直屬 H2；其可見內容與
  `data-text` 都只能是英文字詞，不建立內層 `<p>`。不得加入 `Word:`、詞性、中文或搭配，
  也不得在 H2 後再輸出第二個只含相同英文字詞的朗讀單元。
- 中文五欄講解中的 `本課意思` 必須使用自然的台灣繁體中文，直接說明該字詞在本課原句、
  事件或論述中的具體意思與功能。不得複製英文 `Plain English`、英文定義、英文例句或
  英文情境句，也不得只在英文文字後加中文句號；必要的人名、地名、作品名、專名、代號
  或來源領域慣用名稱可以保留英文。
- `本課意思` 必須是本課專屬說明，不得只重複 `中文意思` 的短譯，也不得全課共用
  「連結本課語境」等空泛模板。
- 五欄講解的標點須在可見內容與 `data-text` 同步修正；欄位值原已含句末標點時不得再
  機械補上中文句號，避免出現 `。。` 等重複標點。保留字詞、搭配及原有例句的必要標點。
- 第二個繁中 `.speech-unit[data-lang="zh-TW"]` 內含 `.vocab-explanation`，並固定依序使用
  五個 `.explanation-row`：`.term`／`單字：`、`.part-of-speech`／`詞性：`、
  `.meaning`／`中文：`、`.lesson-meaning`／`本課意思：`、`.memory`／`記憶提示：`。
  `data-text` 依相同順序串接五欄完整文字，且逐字等於該單元的 `textContent.trim()`。
  本單元的內部標記依骨架緊鄰序列化，不插入排版用的換行或縮排文字節點；欄位仍由
  grid 分成五列。不得把 formatter 產生的空白或 `&#10;` 補進 `data-text` 來通過相等檢查，
  也不得刪除教材文字本身必要的空格。格式化工具須保護此單元內部，其他主要子區塊仍各自換行。
- 第三個英文 `.speech-unit[data-lang="en-US"]` 只包含該詞英文例句，使用
  `<p lang="en">`；第四個 `.speech-unit.example-translation[data-lang="zh-TW"]` 只包含
  該例句的繁中翻譯，使用 `<p lang="zh-Hant">`。兩者 `data-text` 都逐字等於可見正文。
- 每張卡片恰有四個朗讀單元，語言順序固定為
  `en-US → zh-TW → en-US → zh-TW`；英文字詞、繁中五欄、英文例句與繁中例句各朗讀一次。
  第一個單元是 H2，其餘三個單元直接作為卡片子項；不建立 `.vocabulary-script-group`、
  `.vocabulary-term-unit` 或 `.vocabulary-example-unit`。
- 所有 `.speech-unit p` 使用 `margin:0`，讓單字與短例句保持自然高度；繁中五欄依內容
  自然增高，不得使用固定高度、等高 grid、裁切或 overflow 隱藏。
- 控制器直接依 DOM 順序巡覽所有 `.speech-unit[data-lang][data-text]`。播放會從目前段落
  自然接續至下一段；暫停／繼續共用 `#pause`，stop 取消語音並回到 `準備完成`。
  previous／next 與 ±10 取消目前語音後，只定位並高亮 clamp 後的目標段落，按 `play` 才開播。
  語音／語速變更依共用契約停止並維持目前段落的待播高亮，不自動續播或 resume 舊語音。
- active 單元加 `.active`，其所在 `.vocabulary-item` 同時加 `.active-group` 並捲動至
  viewport 中央。首段停用 previous、末段停用 next；狀態清楚顯示準備完成、播放中、
  待播位置、暫停位置、播放完成或錯誤。
- 在 320px、390px 與桌面寬度驗證四單元順序、自然高度、所有 `<p>` 零 margin、
  explanation row 的桌面雙欄／mobile 單欄切換，以及 sticky 控制列不遮住 active 單元；
  所有內容不得裁切、重疊或造成整頁水平溢出。

### Grammar Course

- `grammar.html` 與 Markdown `Tone, Nuance, and Grammar` 的文法項目及順序完全一致；
  主要 HTML 也須呈現相同的來源例句、規則、解析、教學修正版（適用時）及語氣／意義邊界。
- 每課依來源選取 3–10 個可遷移重點；來源不足時使用實際數量並清楚說明限制，不得硬湊。
- 不建立 header。main 依序包含：
  1. H1，固定格式 `中文文法老師 | [課程標題]`；
  2. `.meta`，自然說明實際文法點數及每點都有規則、來源例句、解析、快速檢核與答案理由；
  3. 一個 `zh-TW` 導讀 `.speech-unit`，概述本課將練習的所有文法點；
  4. 依順序排列的 `.grammar-point[data-grammar-id]`；
  5. 一個 `zh-TW` 總結 `.speech-unit`；
  6. `.index-link`，提供返回主要課程與課程索引。
- 每個 `.grammar-point` 的 H2 以 `N. 繁中名稱（English name）` 呈現中英文名稱及順序編號；
  H2 必須放在第一個 `zh-TW` `.speech-unit` 內，先朗讀完整標題（含編號及中英文名稱），
  再朗讀規則；標題只顯示及朗讀一次，不另建標題單元或在規則段落重複標題。
  每點固定依序建立恰好五個 `.speech-unit`：
  1. `zh-TW` 標題與規則：先讀 H2，再用自然繁中說明結構、用途及必要限制；
  2. `en-US` 來源例句：只朗讀一個忠實來源句，不加 `Original wording:` 等標籤；
  3. `zh-TW` 解析：直接說明來源句如何呈現規則、語氣及本課語境；
  4. `zh-TW` 快速檢核：固定以 `快速檢核：` 開頭，先提出明確問題，再完整列出 `A.` 與
     `B.` 兩個英文選項；不得在此單元提前揭露答案；
  5. `zh-TW` 答案理由：固定以 `答案是 A。` 或 `答案是 B。` 開頭，再說明判斷規則。
- 快速檢核必須由該文法點獨立作答，恰有一個最佳答案。錯誤選項應反映常見且單一的目標
  文法錯誤；不得改變來源人物、事實、責任、數字或承諾強度，也不得以語意荒謬、拼字錯誤
  或本點以外規則充當干擾。答案理由必須對應實際選項。
- 第一個規則朗讀單元使用 `<div class="speech-unit" data-lang="zh-TW"
  data-text="[完整雙語標題] [規則]"><h2>[完整雙語標題]</h2> <p lang="zh-Hant">[規則]</p></div>`，
  H2 與規則段落間保留一個文字空白，讓標題與解說不黏連。其餘朗讀單元使用
  `<div class="speech-unit" data-lang="..." data-text="..."><p lang="...">...</p></div>`；
  英文固定 `en-US`／`lang="en"`，繁中固定
  `zh-TW`／`lang="zh-Hant"`。每個 `data-text` 必須逐字等於該單元的
  `textContent.trim()`；原句不確定時保留 `Needs verification`。
- 正確來源句不得誤標為錯誤。若教學修正版是 style improvement，須在規則或解析中明示，
  但來源例句單元仍忠實顯示來源原句，不得以修正版冒充來源。
- 頁面內容 DOM 鎖定下列骨架。`body` 的第一個子元素必須逐字使用「TTS 與控制器鎖定契約」
  定義的 compact `.controls` DOM；下列註解不是可輸出的元素。只能替換方括號內容、文法
  點數及文字，不得改 class、ID、元素順序或增加 wrapper：

```html
<body>
  <!-- 逐字插入本文件定義的 compact .controls DOM；不得輸出本註解。 -->
  <main>
    <h1>中文文法老師 | [課程標題]</h1>
    <p class="meta">[N] 個文法點均包含規則、來源例句、解析、快速檢核及答案理由。</p>
    <div class="speech-unit" data-lang="zh-TW" data-text="[導讀]"><p lang="zh-Hant">[導讀]</p></div>
    <section class="grammar-point" data-grammar-id="grammar-1">
      <div class="speech-unit" data-lang="zh-TW" data-text="1. [繁中名稱]（[English name]） [規則]"><h2>1. [繁中名稱]（[English name]）</h2> <p lang="zh-Hant">[規則]</p></div>
      <div class="speech-unit" data-lang="en-US" data-text="[來源例句]"><p lang="en">[來源例句]</p></div>
      <div class="speech-unit" data-lang="zh-TW" data-text="[解析]"><p lang="zh-Hant">[解析]</p></div>
      <div class="speech-unit" data-lang="zh-TW" data-text="快速檢核：[問題] A. [選項 A] B. [選項 B]"><p lang="zh-Hant">快速檢核：[問題] A. [選項 A] B. [選項 B]</p></div>
      <div class="speech-unit" data-lang="zh-TW" data-text="答案是 [A/B]。[理由]"><p lang="zh-Hant">答案是 [A/B]。[理由]</p></div>
    </section>
    <div class="speech-unit" data-lang="zh-TW" data-text="[總結]"><p lang="zh-Hant">[總結]</p></div>
    <p class="index-link">[返回主要課程] · [返回課程索引]</p>
  </main>
</body>
```
- 控制器直接巡覽所有 `.speech-unit[data-lang][data-text]`。`previous`／`next` 與 ±10
  取消目前語音後，只定位並高亮 clamp 後的目標單元；導覽後按 play 從目標開頭新建播放，
  不得 resume 舊語音。語音／語速變更依共用契約停止並維持目前段落的待播高亮。
  play 只在既有播放確實已暫停且未經導覽或設定變更時 resume，否則從目前單元開頭播放；
  pause 在未播放或待播時不啟動朗讀，在既有播放時切換 pause／resume；stop 取消並回到
  `準備完成`。rate 的 input event 即時停止朗讀、取消佇列，並更新 `rateValue` 為一位小數
  `Nx`；不得只更新顯示而繼續舊朗讀。
- active 單元加 `.active`，其 `.grammar-point` 加 `.active-group`；首段停用 previous、
  末段停用 next。status 顯示準備、待播位置、播放進度、暫停位置或完成，真正語音錯誤顯示於
  `#error`；`canceled`／`interrupted` 不顯示為錯誤。
- 在 320px、390px、660px 與桌面驗證控制列換行、status 完整列、active scroll margin、
  每點五單元順序、快速檢核未洩漏答案、答案理由與選項一致、無裁切／重疊／水平溢出。

### Integrated Terminology Teaching

- 術語老師內容直接整合在 `terminology.html`，不另建 `terminology-teacher-script.html`。
- 與 Markdown `Key Terminology` 的項目及順序完全一致，可依 domain 篩選但不改變原始項目順序。
- 每個 `.term` 只輸出一份整合式術語卡，固定使用 `術語`、`定義`、`來源語境`、
  `流程關係`、`例句` 五個 `.term-field`。術語與例句是雙語欄位，其餘欄位只用
  自然繁中，不另建完整英文對照區塊。
- `術語` 欄依序顯示與朗讀英文術語及繁中名稱，中間以不朗讀的 `｜` 分隔；`例句` 欄依序
  顯示與朗讀英文例句及繁中翻譯。英文內容標示 `lang="en"`／`data-lang="en-US"`，
  繁中內容標示 `lang="zh-Hant"`／`data-lang="zh-TW"`。
- `定義`、`來源語境` 與 `流程關係` 各只有一個 `zh-TW` speech unit，內容必須
  是自然、完整且具實質資訊的繁體中文。必要的角色名、產品名、縮寫、未確認專名與技術
  符號可保留英文，但不得因此建立第二份英文說明。
- 分類、完整詞性與繁中證據分類，只在 H2 下方的一個 `.term-meta` 小字列顯示，
  不建立獨立證據欄、speech unit 或 TTS 資料屬性；三者仍可搜尋。
- 對 `待查證` 項目須明確指出未知部分，不猜測完整名稱。例句固定提供英文來源句或教學
  例句，並提供忠實的繁中翻譯；正文及 `data-text` 不加 `Teaching example:` 前綴，
  不逐卡附加教學示例聲明，也不得將教學例句標成來源逐字引文；統一說明依前述規則處理。
- 術語頁使用七鍵 compact player，固定依每卡
  `en-US → zh-TW → zh-TW → zh-TW → zh-TW → en-US → zh-TW`
  巡覽七個欄位級朗讀單元，排除分類、詞性與證據小字列；
  搜尋與 domain 篩選只改變卡片可見性，不得改寫、重排或排除播放器的朗讀單元。
- H1 後以 `.meta` 顯示實際術語總數與來源依據；搜尋、domain filters、結果數及無結果訊息
  集中在一張 `.tools` 卡片。搜尋輸入有可見 label；domain filters 使用具
  `type="button"`、`data-domain` 與 `aria-pressed` 的按鈕群組，第一個按鈕固定為啟用的
  `All`。每次篩選只切換卡片 `.hidden` 與按鈕 `aria-pressed`，不得重排或改寫卡片。

## 索引規格

- 索引固定為 `house\lesson\index.html`；存在時更新，不存在時建立，不另建頂層或多集索引。
- 課程頁返回索引的連結固定為 `../index.html`；索引連到 `[課程檔名]/[HTML 檔名]`。
  HTML 相對 URL 使用 `/`，本機檔案路徑使用 `\`。
- 依自然檔名順序列出實際存在的課程目錄。
- 每課只連結主要課程、Bilingual Reading、英文教學、本文導讀、中文單字老師、文法課程及整合式
  Terminology HTML。
- 七個連結依序固定指向該課目錄內的 `lesson.html`、`bilingual.html`、`teacher.html`、`content-summary.html`、
  `vocabulary.html`、`grammar.html` 與 `terminology.html`。
- 沒有 workspace 嚴格索引模板時，單層索引固定使用下列 DOM。只能替換方括號內容並重複
  `.lesson-card`；不得改變元素層級、class、七連結順序或可見文字：

```html
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>[主題名稱] English Lessons</title>
  <style>[使用本節鎖定索引 CSS]</style>
</head>
<body>
  <main class="page">
    <header class="index-header">
      <div>
        <p class="eyebrow">[英文主題標籤]</p>
        <h1 class="index-title">[主題名稱]<br>English Lessons</h1>
        <p class="intro">[一段精簡繁中介紹]</p>
      </div>
      <div class="subject-mark" aria-label="[主題名稱]">
        <strong>[兩字元縮寫]</strong><span>[主題短名]</span>
      </div>
    </header>
    <section class="lesson-list" aria-label="課程列表">
      <article class="lesson-card">
        <div class="lesson-number">Lesson [NN]</div>
        <div>
          <h2>[來源顯示標題]</h2>
          <p class="level">[來源類型／主題摘要] · CEFR [程度]（推定）</p>
        </div>
        <nav class="links" aria-label="[來源顯示標題] 課程連結">
          <a class="primary" href="[課程 URL]/lesson.html">主要課程</a>
          <a href="[課程 URL]/bilingual.html">中英對照朗讀</a>
          <a href="[課程 URL]/teacher.html">英文教學</a>
          <a href="[課程 URL]/content-summary.html">本文導讀</a>
          <a href="[課程 URL]/vocabulary.html">中文單字老師</a>
          <a href="[課程 URL]/grammar.html">文法與句構</a>
          <a href="[課程 URL]/terminology.html">整合術語</a>
        </nav>
      </article>
    </section>
  </main>
</body>
</html>
```

- 通用索引 CSS 固定使用：`*{box-sizing:border-box}`；body 為
  `margin:0;font:17px/1.6 "Noto Sans TC","Segoe UI",sans-serif;background:#f7f7f2;color:#20302d`；
  `.page` 為 `width:min(1180px,calc(100% - 24px));margin:24px auto 48px`；
  `.index-header` 為兩欄 grid、`gap:24px`、`align-items:center`、`margin-bottom:20px`；
  `.lesson-list` 為 `display:grid;gap:12px`；`.lesson-card` 為
  `display:grid;grid-template-columns:8rem minmax(0,1fr) minmax(18rem,auto);gap:16px;align-items:center`
  並使用白底、`1px solid #d6dfdc`、8px 圓角及 `12px 14px` padding；`.links` 使用
  `display:grid;grid-template-columns:repeat(3,minmax(9rem,1fr));gap:8px`。
- 索引 anchor 固定為 flex 置中、`min-height:44px`、`.45rem .7rem` padding、6px 圓角、
  `1px solid #91aaa4`、白底、深色文字及無底線；`.primary` 使用 `#2f6f64` 背景與白字。
  所有 anchor 都須有明確 `:hover` 與 `:focus-visible`，focus outline 固定為
  `3px solid #b96800` 及 `2px` offset。標題與連結使用 `overflow-wrap:anywhere`。
- 820px 以下 `.lesson-card` 改為
  `grid-template-columns:6rem minmax(0,1fr)`，`.links` 佔滿下一列；
  560px 以下 `.index-header` 與 `.lesson-card` 都改成單欄，`.links` 改為一欄，
  `.subject-mark` 靠左。任何 viewport 都不得水平溢出、裁切連結或改變七入口順序。
- 桌面版每課可使用「編號／標題資訊／連結」三欄，820px 以下重排，560px 以下改為單欄；
  連結須有至少 44px 可操作高度、清楚的 hover／focus 狀態與 `overflow-wrap:anywhere`。
- 索引視覺可沿用該 index 既有的 accent、標題與裝飾；新建時使用本文件的通用視覺基線。
  DOM、七連結責任、鍵盤操作、響應式重排及離線可用性必須一致。
- workspace 若定義「嚴格索引模板」，該模板明確覆寫本節允許彈性的視覺、class、可見標籤、
  DOM 層級及斷點，但不覆寫來源保真、只連結實際檔案、七入口責任、鍵盤操作及完成前驗證。
  更新時必須把模板視為固定 shell：只替換模板明示的課程重複區、標題、程度／主題摘要與
  href，不得從既有卡片複製歷史別名、任意新增 wrapper、保留多套標籤或逐課改寫 CSS。
- 只連結實際存在的檔案；不得為補齊索引而讀取其他 raw 正文。
- 保留既有索引風格、標題及有效連結，不覆寫人工導覽內容。
- 本 workspace 使用單層課程索引，不將來源拆成多個 Episode 或建立來源根目錄索引。

## 完成前驗證

1. 確認只建立或更新指定 raw 對應的一份 Markdown、`lesson.html`、`bilingual.html`、
   `teacher.html`、`content-summary.html`、`vocabulary.html`、`grammar.html`、`terminology.html` 及必要 index；
   不存在舊長檔名、第二份中文／文法／術語老師頁；raw、其他課程、staging 及 commit 未被
   修改。重新計算 raw 的 SHA-256，確認與開始時記錄或上游提供的值完全相同。
2. 若來源含 Email Content，確認 raw 的 headers、正文、空白、換行、順序、quoted history、簽名、URL 及格式都未被本次工作修改；所有衍生內容只存在於 lesson。
3. 逐節比對 raw，確認已依語意辨識所有 primary source content，而非依賴 `Content`、`Transcript` 或其他固定標題；確認作者／人物、順序、數字、版本、決策、行動、條件、例外、風險、依賴、不確定性及承諾強度完整。
   確認正文配對完整沿用 converted，中文原文的英文已標為譯文，來源資訊與轉換註記未混入
   Reader；輸出確實位於 `house\lesson\[課程檔名]`，且未讀取或修改 source。
4. 確認摘要、Reader、領域知識及一般背景沒有越過事實邊界，且未把 recap、中文補充或 learning aid 冒充原始正文。
5. 確認 Markdown 有一個 H1 與依序排列的 17 個 H2，主要 HTML 的內容及順序一致；主要
   HTML 的單一 main 依序包含頂部返回索引、H1、唯一 `.ai-content-notice` 及 17 個以 H2
   開頭的 section，且所有課程共用相同 CSS。若來源有值得學習的文法，兩者均包含
   source-grounded grammar clinic，且英文教學頁有對應的學習／應用 teaching point。
   另確認 `teacher.html` 只補充英文學習，`content-summary.html` 包含本文摘要、核心重點及
   理解方法；兩頁分別投影對應 canonical 資料，沒有複製整份講稿或互相混入職責。
   本文導讀逐點回溯來源，核對與主要課程摘要、內容筆記及 Reader 一致，未補寫事實。
   確認英文教學與本文導讀未顯示來源行號、Reader pair ID、問題 ID、來源註記清單或
   raw 路徑與雜湊；canonical 仍保留來源對應，既有原始來源連結可用，必要疑義與限制
   已自然融入雙語正文，沒有因移除追蹤資訊而遺漏。確認本文導讀的評論貼合上下文，
   沒有突兀說教或反覆免責聲明；角色宣言未被寫成普遍保證或已實現成果。
   僅供編輯參考的提醒須保留於 canonical 的 `editorial_notes` 與對應 HTML 註解，
   不在 `.speech-unit` 內，也不進入兩語 `data-text`；影響理解的疑義仍在雙語正文交代，
   Reader 的原有不確定標記保持不變。逐單元確認改寫後的可見文字與 `data-text` 一致。
   更新舊課程時，確認原有正確內容已依職責保留或移入導讀頁，沒有因拆分而遺漏。
6. 確認 Markdown、主要 HTML、單字老師稿的單字項目與順序一致；逐卡確認 H2、固定格式
   meta、發音／搭配清單及四個 speech unit 齊全。每卡語序必須恰為
   `en-US → zh-TW → en-US → zh-TW`，第二單元含五個指定 class 的 explanation row，
   第四單元具有 `.example-translation`；四個 `data-text` 均逐字等於可見正文。第一單元
   必須同時是卡片唯一的直屬 H2，且 H2 的文字、`data-term` 與 `data-text` 完全相同；
   H2 後不得存在另一個只含相同英文字詞的 `en-US` speech unit。以每張卡的 direct-child
   順序驗證 H2、meta、details、繁中五欄、英文例句與繁中翻譯，不得只統計單元總數。
   另直接檢查原始碼，確認每張 article 及上述主要子區塊各自換行、使用兩空格倍數縮排，
   且沒有兩張卡片或多個 speech unit 被壓在同一行。第二單元不得含排版用的空白文字節點，
   五欄完整文字串接後必須同時等於 `data-text` 與 `textContent.trim()`；不得只驗證含縮排
   換行的兩個字串恰好相等。
   逐項核對 `本課意思` 是符合來源語境的自然繁中，而非英文定義或只重複短譯；確認沒有
   `。。` 等機械拼接的重複標點。在 320、390、660 與 1280px 以 computed style 及元素
   座標驗證五欄由上到下各佔一列，桌面只有每列內的 label/value 為兩欄，660px 以下改為
   單欄；同時核對 header、H2、meta、details 的字級、背景與緊湊留白符合鎖定值。
   量測首次播放、跨卡及同卡四單元換段後的元素座標，確認目標符合共用可視區、不反覆
   置頂，並在必要時補捲動；停止、錯誤或播畢後重新播放仍能定位目前內容。
7. 確認 Markdown、主要 HTML 與 `grammar.html` 的文法項目、雙語標題、原文、教學修正版、
   規則、解析、語氣／意義邊界及順序一致；頁面恰有一個導讀與一個總結 speech unit，
   每個 `.grammar-point` 恰有五個 speech unit，語序為
   `zh-TW → en-US → zh-TW → zh-TW → zh-TW`。逐點確認 H2 位於首個單元內，該單元的
   `data-text` 以完整雙語標題及一個空白開頭，播放時依序讀出標題與規則且不重複；
   標題被高亮時仍保留 H2 語意。逐點確認快速檢核以該固定文字開頭、
   A/B 恰有一個最佳答案、答案未在檢核單元洩漏、答案理由與實際選項一致；所有
   `data-text` 精確等於可見正文。
8. 確認 Markdown、主要 HTML及整合式 Terminology 頁的術語項目、domain、詞性、一般定義、
   來源語境、Evidence 與順序一致。一般定義須說明術語本身，不把來源的價格、體驗、設備
   配置或評價當作通用意義；品牌、技術與來源指涉分開，新增背景及未確認資訊的分類正確。
   每張 `.term` 恰有五個固定順序的 `.term-field` 與七個固定順序的 `.speech-unit`，語序為
   `en-US → zh-TW → zh-TW → zh-TW → zh-TW → en-US → zh-TW`。逐欄確認術語與
   例句均有中英兩種內容，定義、來源語境、流程關係使用自然繁中，且頁面沒有
   第二份英文 Definition、Source Context、Workflow Relationship 或 Evidence。所有
   `data-text` 精確等於所屬 speech unit 的 `textContent.trim()`。每張卡的 domain、完整詞性
   與「證據：[繁中分類]」依序位於 H2 後唯一的 `.term-meta`，且該小字列及其所有後代均不含
   `.speech-unit`、`data-lang` 或 `data-text`，沒有 `hidden`／`aria-hidden`，仍能被搜尋。
   逐卡確認例句欄保留「例句：」標籤，英文例句正文及 `data-text` 均不含
   `Teaching example:` 前綴，繁中翻譯沒有重複標籤；小字列保留正確的 Evidence 分類，
   不逐卡附加教學示例聲明。依 `example_origin` 核對來源原句與教學新增例句，確認未將
   教學例句標成來源逐字引文；有教學新增例句時，表格或卡片清單前有統一說明，且不加入 TTS。
   英文專名周圍仍須是自然繁中句子，不得以「有一個中文字」作為通過條件。
   以 direct-child selector 驗證每張卡
   只有一個 H2 術語欄、其後一個 `.term-meta`，再依序有定義、來源語境、流程關係與
   例句欄；每欄恰有一個 `.field-label` 與一個 `.field-value`，不得接受瀏覽器自動修復後
   看似正確但原始碼具有多餘或錯置閉合標籤的結構。不得保留獨立證據欄或 `.evidence-value`。
   驗證頁面恰有一組 `.search-field`、
   `#termSearch`、`.filters`、`#resultCount` 與 `#noResults`，且不存在 `#domainFilter`。
   第一個 filter button 必須是 `data-domain="All"` 且
   `aria-pressed="true"`；其餘按鈕與卡片 domain 去重後的首次出現順序一致，初始
   `aria-pressed="false"`。輸入可唯一命中的英文、繁中、定義、例句、domain、詞性與證據分類，
   逐項確認
   只顯示符合搜尋及目前 domain 的卡片；確認結果數、無結果狀態、`.hidden`、按鈕狀態及
   清除搜尋後的完整恢復均正確。另逐字驗證術語頁 H1 符合
   `[課程標題] — [來源內容類型] Vocabulary and Expressions`（未明定來源類型時使用
   `[課程標題] — Vocabulary and Expressions`），並以 computed style 驗證其為
   `1.5rem/1.3`、自然換行且未裁切。
9. 確認未知縮寫未被猜測展開，`Needs verification`、`unclear` 及 source gaps 未消失。
10. 確認題號與 Answer Key 一致，答案未提前揭露，多解題列出可接受答案。
11. 確認 Bilingual Reading 的小節、配對與來源順序同步，中英單元一對一、語系正確，且沒有
    摘要取代正文、跨小節重複或自行補寫的內容。從同一 canonical pair list 逐項驗證 Markdown、
    主要 HTML 與 `bilingual.html` 的小節數、pair 數、編號、英文及繁中全文完全相等；
    Markdown 每個 H3 小節的 Pair、English 及繁中標籤與編號符合固定結構；
    `bilingual.html` 每個 `.speech-unit` 直接含純文字、沒有內層 `<p>`，且 `data-text`
    精確等於可見文字。
12. 靜態檢查唯一 ID、合法 HTML、語言屬性、選聲、錯誤處理、搜尋、篩選及 DOM 綁定；
    從六個 TTS 頁抽取並正規化播放器 DOM、播放器 CSS、voice scoring、voice 載入函式與
    共用控制流程，逐項與本次生成所用的鎖定 renderer 片段比較，三種 compact 頁的播放器
    JavaScript 必須相等。另依內容替換白名單掃描頁型差異，發現白名單外的控制文字、
    selector、property、狀態字串、事件或條件分支時立即失敗，不得只做頁內存在性檢查。
    另檢查可見正文沒有殘留 `**粗體**`、反引號、Markdown 連結或其他未解析的格式控制符，
    並確認 Source and Handling 的欄位標籤使用 `<strong>`、程式碼樣式值使用 `<code>`，
    Markdown 的 Video 欄位使用顯式 `[文字](URL)` 連結；主要 HTML 恰有對應的
    `a.source-video-link`，其解析後 `href` 與 raw 完整 URL 相同，且具備
    `target="_blank"` 與同時包含 `noopener`、`noreferrer` 的 `rel`。不得同時留下另一份
    不可點擊的純文字 Video URL。
13. 在瀏覽器實測六份基本 TTS HTML；workspace 另有 TTS 輸出時一併驗證。
    逐頁注入可控 boundary，涵蓋中英文、標點、emoji、巢狀 heading／行內元素、
    跨文字節點、`charLength` 缺少或為 0，以及完整播放器超過 140 字元的後續短 utterance。
    比對 mark 的實際文字與預期範圍，確認同時最多一個 `mark.spoken-word`，
    高亮前後可見文字、`data-text` 及語意結構不變；computed style 須為
    `rgb(255, 203, 96)` 背景與底線，且不造成版面位移。
    測試暫停／繼續、切段、停止、播畢、錯誤、朗讀語言／中英文語音／語速變更及篩選切換
    的保留或清除行為，
    並模擬延遲送達的舊 boundary，確認不會高亮新單元或恢復舊 mark。
    不發送 boundary 時仍須完整接續朗讀並保留段落高亮；另以實際 voice 驗證同步，
    不支援 boundary 的 voice 明列限制，不以模擬事件冒充實際語音測試通過。
    若有 `work-summary.html`，確認每節恰有一個標題 pair 與兩個標題單元，且 ID、
    語言、可見文字與 `data-text` 符合本節規格。實測三種模式的標題及正文播放順序、
    前後段與 ±10 跳轉、逐字高亮及群組捲動；有 S 個小節、N 組正文配對時，
    中英皆讀進度總數為 `2 × (S + N)`，單語模式為 `S + N`。不得漏讀、重讀標題、
    朗讀分隔符或 H1，也不得改動正文配對、來源對應與正文編號。
    Bilingual Reader、英文教學與本文導讀測試播放、暫停、停止、
    前後段、上／下 10 段、選聲、語速及三種 `readingMode`；逐頁確認單語模式不朗讀另一
    語言，並在第 1、5、11 與最後一個 eligible unit 測試 ±10 clamp。
    比較 `bilingual.html`、`teacher.html` 與 `content-summary.html`，確認三頁的整個 `<style>`、固定 `.controls`
    DOM 及 Web Speech JavaScript 逐字一致，且只存在本規格允許的內容差異。
    所有 TTS 頁（含 compact 頁與 workspace 額外輸出）皆須檢查 ±10 按鈕與事件綁定，
    並在第 1、5、11 與最後一個 eligible unit（存在時）實測索引減／加 10 與首尾 clamp；
    不足 10 個單元（含只有一個單元）時仍須正確跳至首尾，不得改變朗讀順序或朗讀排除
    內容。測試目標也須使用 clamp 後的索引，不得假設必定存在第 11 個單元；首尾停用
    按鈕的操作不得新增 utterance、呼叫 `cancel()` 或改變播放狀態。
    中文單字老師、文法頁與術語頁逐頁測試固定七按鈕、中文／英文語音、rateValue、
    status/error、首尾 disabled 及自然接續；單字頁另確認每卡四語序與 vocabulary-item
    active-group，文法頁確認導讀到總結及 grammar-point active-group。術語頁確認每卡七個
    欄位級單元的語言順序、term active-group、搜尋及 domain 篩選；確認播放及自然接續
    由流程關係直接銜接例句，下一段只定位至例句、上一段則反向定位，待按播放才朗讀，
    均排除分類、詞性與證據小字列。篩選及清除搜尋後仍不得朗讀小字列，
    進度總數固定為實際術語數乘以七；另量測首次播放、跨卡與同卡換欄的元素座標，
    確認目前欄位符合共用可視區、不反覆置頂，停止或播畢後重新播放仍能定位。
    三種 compact 頁面不得出現 `readingMode`，
    但必須提供 ±10 控制，且其播放器控制列 DOM 與
    CSS 的 selector／declaration 必須逐字一致；三者的播放器 JavaScript 也須逐字一致。
    逐一比對每個 transport button 的 `textContent.trim()` 與本文件鎖定值，確認固定圖示和
    固定文字都直接可見；再以 accessibility tree 或 accessible-name API 確認名稱包含
    `播放`、`暫停／繼續`、`停止`、`上一段`、`下一段` 或對應的 ±10 動作文字。只有圖示、
    只有 tooltip、文字被 visually hidden 或由 CSS pseudo-element 補上的按鈕一律失敗。
    於 320、390、660 與 1280px，在設定區域展開時確認各頁型規定的 label 與 select
    不會錯列、拆散或交換順序。
    逐一檢查控制列 label 的 computed `writing-mode` 為 `horizontal-tb`、只形成一行文字，
    且 `中文語音`、`朗讀語言` 與 `語速` 不會逐字堆疊；不得只以沒有整頁水平溢出判定通過。
    對三種 compact 頁面逐一確認 `.transport` 寬度等於控制列可用寬度、`下 10 段` 位於
    `.transport` 內且為最後一個按鈕、第一個 `.control-field` 的頂端不高於
    `.transport` 底端，並確認每個 label 與其 control 具有同一 `.control-field` 父元素。
    這些展開狀態檢查須在四種 viewport 全部通過，不得以某一寬度碰巧自然換行代替結構驗證。
    在同四種寬度逐頁驗證初始、播放、暫停、由兩種按鈕繼續、手動導覽或設定變更後待播、
    停止、自然播畢、錯誤及錯誤後重新播放的設定列狀態；適用頁面另測試暫停後切換
    `readingMode`。播放時全部
    `.control-field` 的 computed `display` 須為 `none`、高度為 0 且不佔 Tab 順序，
    `.transport` 與 `.status-line` 仍可見且相鄰，僅保留共享 `.row` 的正常 gap。
    控制列須比展開時矮，恢復展開後高度須與原先相同；所有狀態均不得出現水平溢出、
    文字裁切、控制項重疊或 sticky 控制列遮住目前卡片。確認收合前後 `readingMode`、
    兩個 `voiceURI`、`rate` 與 `rateValue` 均保留。模擬同步與延遲送達的舊 `onend`／
    `onerror`，驗證停止、前後段／±10 及設定變更後待播狀態維持展開，自然接續維持收合；
    舊回呼不得改變設定列、清除新單元引用、覆蓋待播高亮或錯改狀態。
    同時確認正文、搜尋與 domain 篩選不受收合影響。
    各 TTS 頁另在 320×600、390×600 與桌面測試四個導覽按鈕：以可控 stub 凍結目前語音的
    `onstart` 與 `onend`，在按鈕事件返回後立即量測目標，須符合共用可視區；元素邊界
    僅允許像素取整造成的 1 CSS px 容差。再於兩個實際畫面更新後及至少 700ms 後量測，
    `scrollY` 相對立即定位值的差異須不超過 1 CSS px，且目標仍可見；不得用假時鐘或
    只有索引、樣式與捲動呼叫次數正確作為通過證據。連按與按住按鍵須依最後索引定位，
    導覽事件返回時即有目標 `.active`、`aria-current`、群組 `.active-group` 與待播進度，
    無逐字 mark，設定列展開；不等待語音回呼。釋放舊回呼後仍維持同一待播目標且不朗讀。
    涵蓋首／中／末單元、同群組、跨群組、超長單元、暫停中換段及
    連續跨群組後換向；操作後跨多個畫面更新再次量測，確認沒有原生平滑動畫或殘留位移。
    靜態檢查播放器的捲動呼叫均明示 `behavior:"instant"`，並確認各頁共用相同定位函式。
    觸發舊 start／boundary／end／error 及舊定位回呼，確認不會拉回舊段、改錯進度、
    高亮或播放狀態；單元內短 utterance 接續不重複定位。
    四個導覽按鈕另須逐頁、逐語言模式從未播放、播放中、已暫停、完成與錯誤狀態測試：
    有效操作取消舊語音，不新增 utterance，也不呼叫 `speak()`／`resume()`；原生滑鼠與鍵盤
    操作結果一致。等待至少 700ms、兩個實際畫面更新及舊回呼送達後仍不得自動播放。
    待播時按 `pause` 不得改變狀態或啟動語音；按 `play` 才從目標完整單元的開頭朗讀，
    完整播放器不得沿用上一段的短 utterance 索引或 offset，且須保留 voiceURI、語速與模式。
    提交新語音但尚未送達有效 `onstart` 時不得顯示播放中；有效 `onstart` 後才進入播放中，
    其後 boundary、暫停／繼續與自然接續均維持原契約。按 `play` 收合設定列後須再次量測
    目標可見性；停止、真正錯誤或播畢仍清除待播／播放高亮與追蹤，導覽或設定變更則清除
    舊狀態後建立目標的待播高亮。
    所有 TTS 頁另從未播放、待播、播放中、已暫停、完成與錯誤狀態，分別變更 `zhVoice`、
    `enVoice`、`rate`，以及適用頁面的三種 `readingMode`；包含變更非目前朗讀語言的 voice。
    設定列可見時使用滑鼠及原生鍵盤操作 select／range，播放中設定列收合時則以事件注入
    驗證控制器，不修改交付 DOM；每次有效變更返回時，舊語音及佇列已取消，
    播放中／暫停旗標與短 utterance 進度已重設，沒有新增 utterance 或呼叫
    `speak()`／`resume()`。語音與語速變更保留 eligible index 與單元，模式變更則定位
    新模式的第一個單元，進度分母依該模式計算；不得混讀其他語言或改寫正文順序。
    驗證 `.active`、`aria-current="true"`、`.active-group`、待播進度及展開設定列立即生效，
    不含逐字 mark，目標在 320×600、390×600 與桌面均可見；兩個實際畫面更新及至少
    700ms 後仍無自動朗讀或延遲位移。連續拖動或以鍵盤調整 rate 須逐次更新 `rateValue`
    並維持待播，不能等到 `change` 才停止；滑桿放開不重複重設定位。
    模擬取消時同步及延遲送達的舊 start／boundary／end／error 與定位回呼，確認不覆蓋待播
    狀態、不恢復逐字 mark、不收合設定列、不拉回舊段或恢復朗讀。設定變更後按 `pause`
    仍為待播；按 `play` 才使用最新模式、voiceURI 與 rate，從目標完整單元開頭新建播放，
    不 resume 舊語音或沿用短 utterance offset；有效 `onstart` 後才顯示播放中並正常接續。
    未經導覽或設定變更的一般暫停，仍能由 `play` 與 `pause` 繼續原語音。
    另測新模式沒有 eligible units 時的提示、清除高亮與停用控制，恢復有單元模式後可重新
    定位待播；初始化與 `voiceschanged` 自動重載不得誤觸設定變更流程或新增語音。
    另以未注入 stub 的 Microsoft Edge 驗證播放中或暫停後變更設定會停止舊朗讀，待播時
    不自動續播，按 `play` 才依新設定從目標開頭朗讀；無法實測的語音或事件須明列未驗證。
    在主要 HTML 以滑鼠點擊並以鍵盤聚焦後按 Enter 測試 `.source-video-link`；兩種操作都須
    建立新分頁，且新頁面的初始或最終 URL 與 raw 的影片 URL 相同。若測試環境無網路，
    仍須攔截新分頁並核對導覽目標，不得只以 anchor 存在判定通過。
    另以 Microsoft Edge 開啟 Bilingual Reader，確認 `.transport` 單獨成列、四個
    `.control-field` 順序正確、status 可見，按鈕與欄位不重疊且整頁沒有水平溢出。
    對 Bilingual Reader、英文教學與本文導讀中含多句、`>>`、中英標點及超過 140 字元的單元檢查
    句級切分、完整接續及取消行為；
    確認 raw 中的開頭字幕標記仍存在，衍生課程的畫面與 `data-text` 已移除不具教學功能的
    標記，每個 utterance 不超過 140 字元，且最後一句完成後才前往下一顯示單元。另確認
    英文教學與本文導讀每個主題恰有兩個朗讀單元，各含對應語言的可見主題標題與一次說明，
    `data-text` 與各單元的 `textContent.trim()` 完全相同。實測中英皆讀依序播放中文標題、
    中文說明、英文標題、英文說明；單語模式不漏讀標題、不混讀另一語言，前後段與進度
    仍以兩個單元計算，且不朗讀頁面 H1 或導覽。另測試 Terminology 搜尋及 domain 篩選。
    另以 stubbed `speechSynthesis.getVoices()` 在六個 TTS 頁注入完全相同且刻意打亂順序的
    voice 集合，至少包含低分 voice、同分 voice、Microsoft Xiaoxiao Natural／Online、
    英文 Natural voice 及其他語言 voice；逐頁驗證中英文候選清單、option 順序、預設
    `voiceURI` 與評分結果完全相同。再觸發 `voiceschanged`，分別驗證舊選值存在時保留、
    舊選值消失時改選最高分、初始空清單後延遲出現 voices 時正確選取。任何頁面選到不同
    預設 voice、混入錯誤語言或直接選取第一個低分 voice，都視為失敗。
    stubbed Web Speech API 只用於隔離測試分頁；測試後關閉該分頁，另以未注入 stub 的乾淨
    分頁重新載入實際輸出，確認原生語音清單與互動正常。不得將模擬 voice、全域測試物件或
    初始化腳本留在交付頁面，也不得將模擬測試通過宣稱為實際音訊驗證通過。
14. 確認七份 HTML 的 `<title>`、H1、頁型名稱、meta／header 說明及返回導覽符合頁面骨架；
    返回文字不在朗讀單元內，所有相對連結都可離線開啟。
15. 在 320px、390px 及桌面寬度檢查全部 HTML 與 index，確認無整頁水平溢出、遮擋或無法操作的控制項；
    以 Tab、Enter／Space 及各控制項的瀏覽器標準按鍵操作播放器、搜尋、篩選與連結，
    不將純朗讀單元加入 Tab 順序。另在待播、播放中與暫停時，固定語音回呼，
    點擊首／中／末朗讀單元及標題，並以程式暫時聚焦單元後按 Enter／Space，
    確認不會因此呼叫 `speak()`／`cancel()`、切換段落或改變播放狀態；
    連結及其他互動元件仍執行原生操作，測試後不得留下暫加的焦點屬性。對
    `bilingual.html` 統計 pair 卡片高度及英文單元句數分布，確認高度隨內容自然變化、沒有
    預設段落 margin 造成的整批額外留白；對話型 transcript 另須符合 Bilingual Reading
    規定的句數分布、長度警報及人工回查要求。逐頁抽取 H1、H2、H3、p、清單、第一／中間／
    最後一張卡片及 speech unit 的 computed `margin`、`padding`、`line-height`、`height`
    與 `min-height`，比對本文件鎖定值；相同 selector 不一致、非控制項固定高度、非必要
    min-height、空元素撐高或超過 1rem 的正文上下 padding 都視為失敗。
    另以同一短句建立單行與含縮排換行的兩份記憶體 DOM，分別覆蓋直接純文字、
    內層 `<p>` 及 `.term-meta` 三種結構；在相同可用寬度下，兩份的可見行數與
    高度必須相同，正文與資料內容不因格式化改變；朗讀單元的
    `textContent.trim()`／`data-text` 逐字相等，小字列仍不含 TTS 資料屬性。
    一般 H2 的 computed 行高須為字級的 `1.35` 倍；頁型專用標題依其專用值驗證。
    頁首 `.meta` 須無框線與 padding，不得套用 AI 提醒框；不得僅因 margin 數值正確
    就忽略 `white-space` 或文字節點所產生的空白行。
    單字頁另逐項驗證 header 的下 margin 為 0、H1 在所有寬度均為 `1.5rem`、非 active
    單字 H2 的上／左／右 padding 均為 0、meta 無提示框，以及五個 explanation row
    的 margin、border 與背景一致，`term` 列不得多出卡片框線。第二個繁中單元及五欄的
    computed `white-space` 必須為 `normal`，不得在五欄前後形成縮排造成的空白行。
    比較不同課程時，另以相同的中性短標題、單字及五欄文字建立僅供測試的記憶體 DOM，
    核對頁面與卡片的排版數值，避免把教材長度造成的自然高度差誤判為 UI 差異；
    測試後重載原頁，不得將測試文字寫回 HTML。另於 660、661、999 與 1000px 確認所有 TTS
    transport 的 computed display 為 grid，欄數依序為二、四、四與七，按鈕文字不溢出，
    不以 CSS 宣告存在代替驗證。
    術語頁另逐一比對
    `.term`、術語 H2、`.term-meta`、`.term-field`、`.field-label`、`.bilingual-value`、
    `.example-pair` 與 `.tools` 的 computed display、grid template、gap、margin、padding、
    border、font size 及 line height；確認 660px 上下的欄位切換符合規格，第一／中間／
    最後一張卡的相同 selector 數值完全一致。`.term-meta` 須以同列小字呈現分類、詞性與
    證據，字級為 `.78rem`、行高 `1.35`、margin `.15rem 0`，無 padding、框線與提示背景；
    320、390、660 與 1280px 均可自然換行、無裁切，不建立獨立證據欄。
    另逐一量測四個非術語 `.term-field`：
    computed 上下 margin 必須各為 `.12rem`，其內 `.speech-unit` margin 必須為 `0`、
    上下 padding 必須各為 `.18rem`；相鄰非術語欄位的可見間距不得超過 `.12rem`，不得以
    空白文字節點、空元素或通用 speech-unit margin 撐高。
16. 驗證 index 的 `main.page`、`header.index-header`、`section.lesson-list`、
    `article.lesson-card`、`div.lesson-number` 及 `nav.links` 層級與順序。沒有 workspace
    覆寫時，每張卡必須恰有七個固定順序連結且只有主要課程連結具有 `.primary`；有嚴格模板
    時，逐項依該模板驗證，不得混用通用與覆寫 DOM。於 320px、390px、
    560px、820px 及桌面檢查 grid 欄數、44px 最小連結高度、hover／focus-visible、
    subject mark 位置及無水平溢出。再從 index 開啟所有連結，確認檔名、相對路徑與目標存在，
    且不存在尚未生成頁面的失效入口。
17. 以 UTF-8 重新讀取七份 HTML 與 Markdown，掃描連續三個以上 `?`、Unicode replacement
    character `U+FFFD` 及 private-use 字元 `U+E000–U+F8FF`；除來源確實包含且已核對的文字外，
    任一命中都視為編碼失敗。再由瀏覽器核對 `<title>`、H1、控制列標籤、說明及返回連結，
    不得只確認檔案可解析。產生或格式化含非 ASCII 內容時必須明示 UTF-8，不得經過會以系統
    ANSI code page 解碼再覆寫的 shell pipeline。
18. 直接檢查七份 HTML 的原始碼，而非只檢查瀏覽器解析後的 DOM：確認文件與主要容器不是
    單行，巢狀標記使用一致縮排，內嵌 CSS 與 JavaScript 沒有壓縮片段，且結構可供人工審查
    及局部 patch。比較格式化前後的 DOM、可見文字與互動測試結果，確認空白敏感內容、
    CSS cascade、TTS、可存取性及頁面行為未改變；不得以 formatter 成功退出代替此驗證。
    使用 standards-compliant HTML parser 檢查原始碼，任何 unmatched、unexpected、
    misnested 或 duplicate-attribute 錯誤都須修正，不得依賴瀏覽器容錯。另掃描 tab、
    非兩空格倍數的結構縮排、同一行的相鄰 `.term-field`／卡片、單行 CSS 規則及單行控制
    流程；排除文字內容、URL、字串與空白敏感區後，任何命中都視為格式驗證失敗。
19. 執行可用的檔案診斷與 `git diff --check`；檢查限定 diff 沒有與目標無關的全檔重排或
    純格式變更，並刪除一次性腳本及暫存檔。
20. 最後檢查 Git 狀態，清楚區分本次修改與原先已存在的變更，不得回復或納入無關變更。
