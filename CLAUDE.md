# 專案背景：美甲店 LINE 機器人（n8n 作品集）

使用者在自學 n8n 串 LINE 機器人，目標是做成接案作品集。模擬客戶是一間美甲店。
語言：一律用繁體中文，回報簡短（做了什麼、測試結果、下一步需要使用者做什麼）。

## 規則

- 一律使用 LINE Messaging API，不使用已停用的 LINE Notify。
- 憑證只選用 n8n 裡已存在的 Header Auth 憑證（Name: Authorization）。
  不要把 token、channel secret、n8n API key 寫進工作流、檔案、commit 或回報。
- 匯出工作流 JSON 進 repo 前，先檢查裡面沒有 token。
- 需要使用者本人操作的步驟（Google 授權視窗、手機確認有沒有收到 LINE）請停下來告訴他。
- 遇到歧義或不可逆的更動（例如設為 Active、刪除節點）先問，不要自己決定。
- 每完成一段就實際執行測試，貼出 n8n 的執行結果（成功或錯誤訊息）。

## 已完成（不需重做）

- LINE 官方帳號與 Messaging API 已啟用。
- n8n 已存好 Header Auth 憑證，直接選用。
- n8n 工作流「美甲預約通知」已存在，內有 HTTP Request 節點 `Send LINE Push`：
  `POST https://api.line.me/v2/bot/message/push`，已手動測試成功。
- Google 表單「美甲預約」已建立並連結試算表，分頁「表單回應 1」。
  欄位：時間戳記、姓名、聯絡電話、服務項目、預約日期、時段、備註。
- 工作流時區 Asia/Taipei。

## 待辦（依序）

### ② 預約收單＋通知老闆（下一步）

- 手動觸發換成 Google Sheets Trigger（On row added，Poll 每分鐘）。
- Document 選「美甲預約（回覆）」類似名稱的試算表，Sheet 選「表單回應 1」。
- 接到 `Send LINE Push`，訊息：`新預約｜姓名｜服務項目｜預約日期 時段｜電話 聯絡電話`。
- 先不放「備註」（可能含引號或換行讓 JSON 壞掉）。之後要加，用 Expression 的 `JSON.stringify`。
- 收件人 `to`：使用者的 LINE user ID（U 開頭，尚未提供；測試階段用自己，之後換成老闆）。
- 測試通過後設為 Active（設之前先問使用者）。
- 完成標準：用公開表單填一筆新資料，約 1 分鐘內手機收到格式正確的預約通知。

### ① 常見問題自動回覆

- 先確認 LINE 後台已開 Webhook、關閉「自動回應訊息」。
- n8n Webhook 節點接收 → 查 FAQ 試算表（關鍵字 → 回答）→ reply API 回覆。
- 找不到答案：回覆預設文字，並通知老闆。
- Webhook 收到的事件含客人的 LINE user ID，可順便記錄（見 ③）。
- 提醒：LINE 後台要填 Production URL，工作流必須是 Active 才收得到。

### ③ 預約前一天自動提醒

- Schedule Trigger（每天固定時間，Asia/Taipei），讀預約試算表，篩出預約日期 = 明天的列。
- 推播給客人需要客人的 LINE user ID，表單拿不到。

**尚未決定，請先和使用者討論，不要自行假設。** 候選方案：

1. 客人加好友後傳一次電話號碼，由 ① 的 Webhook 記錄「電話 → userId」到對照表，③ 再用表單電話對照（目前推薦，因為複用 ①）。
2. 表單改用 LIFF 網頁直接取得 userId（工程量大）。
3. 不推播客人，改成通知老闆「明天有這些預約」。

## 環境說明

- 工作流在 n8n Cloud 上，不在這個 repo 裡。這個 repo 用來存匯出的工作流 JSON、截圖與 README。
- 雲端 Claude Code session 連不到 n8n Cloud / Google / LINE。
  在使用者本機終端機執行時，可用 n8n REST API（API key 放環境變數，例如 `N8N_API_KEY`、`N8N_BASE_URL`），
  或搭配 Chrome 擴充功能操作使用者已登入的瀏覽器。
