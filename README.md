# n8n 工作流程：Exa＿健康研究摘要器 (健康顧問)

這個 n8n 工作流程提供了一個簡單的網頁表單介面（「健康顧問」），讓使用者可以提出健康相關的問題。它會使用 Exa API 在 PubMed 上搜尋相關的近期研究論文，然後利用 OpenAI 語言模型將研究結果摘要成繁體中文。

## 工作流程概觀

1.  **表單提交觸發 (`On form submission`)**: 當使用者透過指定的 n8n 表單提交問題時，啟動此工作流程。
2.  **Exa 搜尋請求 (`HTTP EXA Request`)**: 接收使用者的問題，並向 Exa API 發出查詢。
    *   指定搜尋「研究論文」(research paper) 類別。
    *   擷取前 3 筆結果。
    *   篩選 2022 年 1 月 1 日之後發表的研究。
    *   將搜尋網域限制在 `pubmed.ncbi.nlm.nih.gov`。
    *   要求取得結果的文字內容 (text content)。
3.  **AI 摘要 (`AI Agent` + `OpenAI Chat Model`)**:
    *   處理從 Exa 搜尋取得的文字結果。
    *   透過連接的 `OpenAI Chat Model` 節點使用 `gpt-4o-mini` 模型。
    *   遵循系統提示：「請把所有文章濃縮成100字重點整理翻譯成繁體中文。」
    *   以繁體中文輸出一份簡潔的摘要。

**注意：** 此工作流程包含一個已設定好的 **ElevenLabs 文字轉語音節點 (`HTTP Request1`)**，目的是將摘要文字轉換成語音。然而，在提供的 JSON 檔案 (`Exa_爬文.json`) 中，此節點**並未連接**到 `AI Agent` 節點的輸出。因此，依照目前的定義，工作流程會在產生文字摘要後停止。

## 前置準備

*   一個運作中的 n8n 實例 (雲端版或自行架設)。
*   API 金鑰 (API Keys)：
    *   Exa AI API 金鑰
    *   OpenAI API 金鑰
    *   (選用，若要連接 TTS 節點) ElevenLabs API 金鑰
*   n8n 憑證 (Credentials)：
    *   在您的 n8n 實例中已設定好的 OpenAI 憑證。

## 設定步驟

1.  **匯入工作流程**: 將 `Exa_爬文.json` 檔案匯入到您的 n8n 實例中。
2.  **設定憑證**:
    *   確保您已在 n8n 中設定好 OpenAI 憑證。`OpenAI Chat Model` 節點需要憑證 (範例中使用 ID 如 `u6qqJEKkbai5BXYQ`，您需要選擇自己設定好的憑證)。
3.  **新增 API 金鑰**:
    *   **Exa**: 編輯 `HTTP EXA Request` 節點。前往「Headers」區段（可能在「Authentication」/「Header Auth」下，依 n8n 版本/介面可能不同），將 `Authorization` 標頭值 (`Bearer <Your_API_key>`) 中的 `<Your_API_key>` 取代為您實際的 Exa API 金鑰。
    *   **(選用) ElevenLabs**: 如果您打算連接並使用文字轉語音節點 (`HTTP Request1`)，請編輯該節點。前往「Headers」區段，將 `xi-api-key` 標頭值中的 `<Your_API_key>` 取代為您實際的 ElevenLabs API 金鑰。您同時也需要將 `AI Agent` 節點的輸出連接到此 `HTTP Request1` 節點的輸入。
4.  **啟用工作流程**: 在 n8n 介面中儲存並啟用此工作流程。

## 使用方式

1.  工作流程啟用後，找到 `On form submission` 觸發節點的測試 Webhook 網址 (通常顯示在節點的設定面板或工作流程總覽中)。
2.  在您的網頁瀏覽器中開啟此網址。您將會看到「健康顧問」表單。
3.  在「What is your question?」文字區域中輸入您的健康相關問題。
4.  提交表單。
5.  工作流程將會執行。您可以在 n8n 的執行紀錄 (execution log) 中檢視執行細節以及 `AI Agent` 節點最終輸出的摘要文字。

## 限制

*   文字轉語音功能在此 JSON 檔案中已設定，但預設並未連接。
*   搜尋僅限於 PubMed 上 2022 年 1 月 1 日後發表的前 3 筆結果。
*   摘要內容簡潔 (約 100 字)，且限定為繁體中文。
*   工作流程的執行依賴外部 API (Exa, OpenAI, ElevenLabs) 的可用性與速率限制。
*   摘要的品質取決於搜尋結果的相關性以及 AI 模型的能力。
*   **免責聲明：** 此工具提供研究論文的自動化摘要，**不應**被視為專業醫療建議的替代品。若有任何健康方面的疑慮，請務必諮詢合格的醫療保健提供者。
