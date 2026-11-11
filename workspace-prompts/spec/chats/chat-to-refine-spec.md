# 改造來源轉換與英文課程規格

以下兩份規格從其他 workspace 複製而來，請直接依本 workspace 的需求調整，使流程明確、內容精簡且易於使用。

## 1. 來源轉換

調整 `workspace-prompts\spec\source-conversion.md`：

- 輸入：使用者指定的 `house\source` 下單一 Markdown 檔案，例如 `house\source\日本少子化房.md`。
- 轉換：無論原文為中文或英文，均整理為易讀的中英雙語內容，保留原意。
- 排版：正文逐句對照，每句中文後緊接對應的英文，方便英文學習。
- 輸出：在 `house\converted` 下建立新的 Markdown 檔案，不修改來源檔案。

## 2. 英文課程生成

調整 `workspace-prompts\spec\generate-english-lesson-prompts.md`：

- 輸入：使用者指定的 `house\converted` 下單一已整理 Markdown 檔案。
- 生成：依該檔案內容製作英文學習課程。
- 輸出：在 `house\lesson` 下建立該課程的專屬子目錄，存放課程檔案。

## 調整要求

- 移除舊 workspace 專用的路徑與不適用的要求。
- 明確定義輸入、處理步驟、輸出格式及命名規則。
- 更新受改名影響的引用，確保「來源 → 雙語整理 → 英文課程」流程一致。

## 調整雙語對照順序

請修改 `workspace-prompts/spec/generate-english-lesson-prompts.md`，將 `bilingual.html` 的雙語對照順序由「先英文、後中文」改為「先中文、後英文」，每句中文後緊接對應的英文，並同步更新規格中的相關說明與範例。
