## 建立來源轉換與課程生成提示詞

請在 `workspace-prompts\spec\chats` 下建立一份 Markdown 檔案，提供可直接使用的提示詞，分別用於來源轉換、英文課程生成，以及依序完成兩個階段。

- 輸入：
  - 來源轉換規格：`workspace-prompts\spec\source-conversion.md`
  - 課程生成規格：`workspace-prompts\spec\generate-english-lesson-prompts.md`
- 輸出：
  - `workspace-prompts\spec\chats` 下的新 Markdown 檔案。

每個提示詞須有清楚的標題，並明確列出適用規格、輸入與輸出路徑；使用可替換的檔名占位符，且不得偏離規格要求。