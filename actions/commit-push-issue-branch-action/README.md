# commit-push-issue-branch-action

將 issue workflow 產生的結果提交並推送至對應的 `issue-<N>` branch。

此 action 為通用版本，適用於任何需要在 issue branch 上提交並推送結果的工作流程，不限於特定類型的產出（圖片、文字、程式碼等）。

## Inputs

| 名稱 | 必填 | 預設值 | 說明 |
|------|:----:|--------|------|
| `issue_number` | ✅ | — | Issue 編號，用於 commit message 和 branch name |
| `push_token` | ✅ | — | 用來 `git push` 的 GitHub token |
| `github_repository` | ✅ | — | 倉庫名稱，格式為 `owner/repo` |
| `git_user_name` | ❌ | `github-actions[bot]` | git commit 的 `user.name` |
| `git_user_email` | ❌ | `41898282+github-actions[bot]@users.noreply.github.com` | git commit 的 `user.email` |
| `commit_message` | ❌ | `chore: update results for issue #${ISSUE_NUMBER}` | 自訂 commit message，支援 `${ISSUE_NUMBER}` 展開 |

## 使用範例

### 最簡版（僅傳必填 inputs）

```yaml
- uses: Duotify/GitHubClawToolkit/actions/commit-push-issue-branch-action@main
  with:
    issue_number: ${{ github.event.issue.number }}
    push_token: ${{ secrets.GITHUB_TOKEN }}
    github_repository: ${{ github.repository }}
```

### 完整版（傳所有 inputs）

```yaml
- uses: Duotify/GitHubClawToolkit/actions/commit-push-issue-branch-action@main
  with:
    issue_number: ${{ github.event.issue.number }}
    push_token: ${{ secrets.PUSH_TOKEN }}
    github_repository: ${{ github.repository }}
    git_user_name: "my-bot[bot]"
    git_user_email: "my-bot@users.noreply.github.com"
    commit_message: "chore: 更新 issue #${ISSUE_NUMBER} 的處理結果"
```
