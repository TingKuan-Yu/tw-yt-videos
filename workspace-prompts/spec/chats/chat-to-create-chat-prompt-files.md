## 建立來源轉換與課程生成提示詞

請在 `workspace-prompts\spec\chats` 下建立一份 Markdown 檔案，提供可直接使用的提示詞，分別用於來源轉換、英文課程生成，以及依序完成兩個階段。

- 輸入：
  - 來源轉換規格：`workspace-prompts\spec\source-conversion.md`
  - 課程生成規格：`workspace-prompts\spec\generate-english-lesson-prompts.md`
- 輸出：
  - `workspace-prompts\spec\chats` 下的新 Markdown 檔案。

每個提示詞須有清楚的標題，並明確列出適用規格、輸入與輸出路徑；使用可替換的檔名占位符，且不得偏離規格要求。

## 建立規格更新後的課程更新提示詞

請在 `workspace-prompts/spec/chats` 下新增一份 Markdown 檔案，提供可直接使用的提示詞，供規格更新後依最新要求更新既有英文課程。

- 適用規格：`workspace-prompts/spec/source-conversion.md` 與 `workspace-prompts/spec/generate-english-lesson-prompts.md`。
- 提示詞須區分「僅更新課程」與「重新轉換來源後更新課程」，明確列出各階段適用規格、輸入、允許修改的範圍及輸出路徑，不混用兩階段的責任。
- 使用 `[課程檔名]` 作為占位符，對應 `house/source/[課程檔名].md`、`house/converted/[課程檔名].md` 與 `house/lesson/[課程檔名]/`。
- 每個提示詞須有清楚的標題，遵守規格中的更新、內容保留及驗證要求，不修改原始來源或無關課程。
