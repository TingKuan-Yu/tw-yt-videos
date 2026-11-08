# 來源轉換與英文課程生成提示詞

將 `[檔名]` 替換為實際檔名（不含 `.md`），再複製所需提示詞執行。例如：`日本少子化房`。
每次只處理一個指定檔案；下列提示詞均以適用規格為準，不取代其完整要求。

## 1. 僅轉換來源

```markdown
請依下列規格，將指定來源轉換為易讀的繁體中文與英文對照 Markdown。

- 適用規格：
  - `workspace-prompts\spec\source-conversion.md`
- 輸入：
  - `house\source\[檔名].md`
- 輸出：
  - `house\converted\[檔名].md`

要求：
- 保留完整正文與原意，每句中文後緊接對應英文，依規格處理 Markdown 硬換行。
- 來源檔案唯讀；若同名輸出已存在，先取得覆寫授權。
- 僅執行來源轉換，不生成課程或更新課程索引。
- 完成規格要求的驗證，回報輸入、輸出、原文語言、轉換摘要及待確認問題。
- 驗證失敗或未完成時明確回報，不宣稱成功；不得 stage、commit 或 push。
```

## 2. 僅生成英文課程

```markdown
請依下列規格，從指定的雙語整理檔生成英文學習課程。

- 適用規格：
  - `workspace-prompts\spec\generate-english-lesson-prompts.md`
- 輸入：
  - `house\converted\[檔名].md`
- 輸出：
  - `house\lesson\[檔名]\` 內規格定義的一份課程 Markdown 與七份 HTML。
  - `house\lesson\index.html` 的必要建立或更新。

要求：
- 只讀取指定 converted，不讀取 source 或其他教材正文，也不回寫 converted。
- 沿用既有中英配對與順序；課程 Reader 按規格顯示英文在前、中文在後，不重新翻譯。
- 輸出目錄不存在時使用 `create`；已存在時先取得更新授權，保留正確人工內容。
- 輸入配對缺漏或無法可靠辨識時明確回報，不在本階段補做來源轉換。
- 完整遵守八檔輸出、章節、HTML、TTS、索引及驗證契約；索引只加入已生成並驗證通過的頁面。
- 回報輸入、輸出檔案、索引變更、驗證結果及未驗證項目；不得 stage、commit 或 push。
```

## 3. 依序轉換來源並生成課程

```markdown
請依下列兩份規格，先轉換指定來源，通過驗證後再生成英文學習課程。

- 適用規格：
  - 來源轉換：`workspace-prompts\spec\source-conversion.md`
  - 課程生成：`workspace-prompts\spec\generate-english-lesson-prompts.md`
- 輸入：
  - `house\source\[檔名].md`
- 輸出：
  - `house\converted\[檔名].md`
  - `house\lesson\[檔名]\` 內規格定義的一份課程 Markdown 與七份 HTML。
  - `house\lesson\index.html` 的必要建立或更新。

執行順序：
1. 依來源轉換規格建立完整雙語 Markdown，正文每句繁體中文後緊接對應英文；source 一律唯讀。
2. 完成來源保真、雙語配對、UTF-8、Markdown 格式及 source SHA-256 未變的驗證。
3. 只有轉換驗證通過後，才以 `house\converted\[檔名].md` 作為課程階段的唯一內容來源。
4. 依課程規格產生八檔課程及必要索引；課程階段不重新讀取 source、不回寫 converted，也不重新翻譯正文配對。
5. 執行課程規格要求的靜態、跨檔、瀏覽器、TTS、連結及 converted SHA-256 未變的驗證。

限制與回報：
- 同名 converted 已存在時先取得覆寫授權；課程目錄已存在時先取得更新授權，保留正確人工內容。
- 任一階段失敗時明確回報，停止依賴該結果的後續步驟，不交錯執行兩個階段。
- 分別回報兩個階段的輸入、輸出、驗證結果、待確認問題與未驗證項目，不將部分完成稱為全部通過。
- 不處理其他來源或課程，不得 stage、commit 或 push。
```
