# Antigravity GCP 範本

在 GitHub Actions 執行 Antigravity CLI（agy），透過 GCP 免費試用額度串接 AI 模型。

## 取得 AGY_OAUTH_TOKEN

### 前置條件

- Google Cloud 帳號（已啟用免費試用 $300 額度）
- 本機已安裝 Antigravity CLI（`agy`）

### 步驟

1. 在終端機執行：

```bash
agy --print "hello"
```

2. 瀏覽器會跳出 Google 帳號授權頁面。

> ⚠️ **重要：請選擇「Use a Google Cloud Project」**，不要登入個人的 Google AI Pro 帳號。選擇你建立好的 GCP 專案來授權。

3. 授權完成後，AGY 會將憑證寫入以下路徑：

```
~/.gemini/antigravity-cli/antigravity-oauth-token
```

4. 取得 token 內容：

```bash
cat ~/.gemini/antigravity-cli/antigravity-oauth-token
```

輸出會是一段 JSON，類似：

```json
{
  "auth_method": "oauth",
  "token": {
    "access_token": "ya29.a0...",
    "refresh_token": "1//0e...",
    "token_type": "Bearer",
    "expiry": "2025-..."
  }
}
```

5. 複製**整份 JSON 內容**，這就是 `AGY_OAUTH_TOKEN` 的值。

> ⚠️ 請複製完整的 JSON，不是只有 refresh_token 欄位。

## 取得 AGY_GCP_PROJECT

1. 前往 [Google Cloud Console](https://console.cloud.google.com/)
2. 選擇你在登入 AGY 時授權的專案
3. 在專案資訊卡片（Dashboard）或網址列中找到 **Project ID**

> ⚠️ 要填的是 **Project ID**（例如 `my-project-123456`），不是專案顯示名稱。

Project ID 也可以在 Cloud Console 左上角專案選擇器中看到，每個專案名稱下方灰色小字就是 ID。
