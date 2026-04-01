# Actions 說明文件

此目錄收錄 GitHubClawToolkit Issue 處理流水線所使用的 Composite GitHub Actions。每個 Action 皆可獨立被其他 Workflow 引用。

## Actions 一覽

| Action | 用途 |
|---|---|
| [`commit-push-issue-branch-action`](#commit-push-issue-branch-action) | 將 Issue Workflow 產生的變更提交並 force-push 至對應的 `issue-<N>` branch |
| [`error-handler-action`](#error-handler-action) | 解析 Copilot CLI JSONL 日誌、辨識錯誤類型，並將繁體中文摘要寫入結果檔 |
| [`update-comment-action`](#update-comment-action) | 將結果檔內容更新（PATCH）至指定的 GitHub Issue 留言 |

---

## commit-push-issue-branch-action

將 Issue Workflow 執行期間產生的所有變更，以指定的 commit message 提交，並 force-push 至對應的 `issue-<N>` branch。若 staging area 無任何變更，則自動跳過 commit，避免產生空提交。

### Inputs

| Input | 必填 | 預設值 | 說明 |
|---|---|---|---|
| `issue_number` | ✅ | — | Issue 編號；用於 commit message 和 branch name |
| `push_token` | ❌ | `github.token` | 用來執行 `git push` 的 GitHub Token |
| `github_repository` | ❌ | `github.repository` | `owner/repo` 格式的儲存庫名稱 |
| `git_user_name` | ❌ | `github-actions[bot]` | Git commit 作者名稱 |
| `git_user_email` | ❌ | `41898282+github-actions[bot]@users.noreply.github.com` | Git commit 作者 Email |
| `commit_message` | ❌ | `chore: update results for issue #${ISSUE_NUMBER}` | Commit message 範本；`${ISSUE_NUMBER}` 會自動展開為實際 Issue 編號 |

### Outputs

無

### 使用範例

```yaml
- name: 提交並推送 Issue 分支
  uses: Duotify/GitHubClawToolkit/actions/commit-push-issue-branch-action@v1
  with:
    issue_number: ${{ github.event.issue.number }}
    push_token: ${{ secrets.GITHUB_TOKEN }}
    github_repository: ${{ github.repository }}
```

---

## error-handler-action

當 Copilot CLI 任務失敗時，解析 JSONL 輸出日誌，依優先順序辨識錯誤類型（認證失敗、配額超限、WebSocket 錯誤、`session.error` 等），並將對應的繁體中文錯誤摘要寫入結果檔，供後續步驟貼至 GitHub Issue 留言。

### 錯誤分類優先順序（由高至低）

1. **OpenAI WebSocket HTTP 500** — 同時出現 `responses_websocket`、`failed to connect to websocket`、`500`
2. **Google API 429 配額超限** — 出現 `429`
3. **認證失敗** — 出現 `Error: No authentication information found.`
4. **JSONL `session.error`** — 逐行解析，抽取最後一筆 `type === "session.error"` 的 `data.message`
5. **Fallback** — 以上皆無時使用 `missing_session_error_message`

### Inputs

| Input | 必填 | 預設值 | 說明 |
|---|---|---|---|
| `output_file` | ✅ | — | Copilot CLI JSONL 輸出日誌的檔案路徑 |
| `result_file` | ✅ | — | 錯誤摘要輸出檔的路徑 |
| `missing_session_error_message` | ❌ | 繁中 fallback 訊息 | 找不到 `session.error` 時使用的訊息 |
| `message_prefix` | ❌ | 繁中說明文字 | 擷取到 `session.error` 時，寫在訊息上方的前綴 |
| `message_suffix` | ❌ | 繁中補充文字 | 擷取到 `session.error` 時，寫在訊息下方的後綴 |
| `no_auth_message` | ❌ | 繁中認證錯誤訊息 | 偵測到認證失敗時使用的訊息 |
| `google_quota_exceeded_message` | ❌ | 繁中配額超限訊息 | 偵測到 HTTP 429 時使用的訊息 |
| `openai_permission_message` | ❌ | 繁中 OpenAI 錯誤訊息 | 偵測到 OpenAI WebSocket HTTP 500 時使用的訊息 |

### Outputs

無（錯誤摘要直接寫入 `result_file`）

### 使用範例

```yaml
- name: 處理執行錯誤
  if: failure()
  uses: Duotify/GitHubClawToolkit/actions/error-handler-action@v1
  with:
    output_file: workspaces/issue-${{ github.event.issue.number }}/copilot-exec-log.json
    result_file: workspaces/issue-${{ github.event.issue.number }}/result.txt
```

---

## update-comment-action

讀取結果檔內容（若檔案不存在則使用預設失敗訊息），在末尾附加 `brain_meta` bot 識別標記，並透過 `gh api PATCH` 更新至指定的 GitHub Issue 留言。API 呼叫失敗時最多重試 5 次（每次間隔 6 秒）。

### Inputs

| Input | 必填 | 預設值 | 說明 |
|---|---|---|---|
| `issue_number` | ✅ | — | 目標 Issue 編號 |
| `comment_id` | ❌ | `''` | 要更新的留言 ID |
| `message_file` | ❌ | `''` | 留言內容的檔案路徑；不存在則使用預設失敗訊息 |
| `brain_meta` | ❌ | `<!-- githubclaw-brain-result: {"source":"githubclaw-worker-brain"} -->` | 附加於留言末尾的 bot 識別標記（HTML 註解格式） |

### Outputs

| Output | 說明 |
|---|---|
| `comment_id` | 被更新的留言 ID（從 API 回應中抽取） |

### 使用範例

```yaml
- name: 更新 Issue 留言
  if: always()
  uses: Duotify/GitHubClawToolkit/actions/update-comment-action@v1
  with:
    issue_number: ${{ github.event.issue.number }}
    comment_id: ${{ steps.progress-comment.outputs.comment_id }}
    message_file: workspaces/issue-${{ github.event.issue.number }}/result.txt
```

---

## 版本發布流程

### 1. 更新 `action.yml`

修改對應 Action 目錄下的 `action.yml`，完成功能變更或修正。

### 2. 提交並推送

```bash
git add actions/
git commit -m "chore: update actions for vX.Y.Z"
git push origin main
```

### 3. 打上版本標籤

```bash
git tag vX.Y.Z
git push origin vX.Y.Z
```

將 `vX.Y.Z` 替換為實際版本號（例如 `v1.2.0`）。打上 tag 後，其他 Workflow 即可透過 `@vX.Y.Z` 固定引用該版本。
