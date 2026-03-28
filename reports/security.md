# 情報セキュリティチーム レポート
日付: 2026-03-28

## 総合評価: YELLOW

## チェック結果
| 項目 | 状態 | 詳細 |
|------|------|------|
| APIキー漏洩 | ✅ | ハードコードされたAPIキーなし。`sk-ant-...` はplaceholderのみ、`AIza...` 実キーなし |
| Gitコミット | ✅ | 最近20件に機密情報なし。`7222361` "revert API key storage" はストレージ方式変更の正規コミット |
| 公開データ | ✅ | `portfolio.json` は空（`{}`）、`watchlist.json` に個人情報・APIキーなし |
| index.html | ✅ | ハードコードキーなし、外部CDNスクリプトなし、APIキーはlocalStorage（`gk`/`ck`）管理 |
| 設定ファイル | ⚠️ | `.gitignore` に `.env` / `*.key` パターンなし（現在は.envファイル不在だが保護なし） |

## 要対応事項

### ⚠️ `.gitignore` に `.env` / `*.key` が未登録
- **状況**: 現在リポジトリに `.env` ファイルは存在しないが、`.gitignore` に保護パターンがない
- **リスク**: 将来 `.env` ファイルを作成した際に誤ってコミットされる可能性がある
- **対応**: `.gitignore` に以下を追加する
  ```
  .env
  .env.*
  *.key
  *.pem
  ```

## 推奨事項

### stock_mcp_server.py の export_fins_data で NaN 対策を追加
- **状況**: `export_fins_data()` の `json.dumps(fins_data, ...)` が `_sanitize_nans()` を経由していない
- `_fetch_fins_history()` 内の `_num()` 関数が `float("NaN")` を返す可能性がある（J-Quants API が "NaN" 文字列を返した場合）
- `run_screen_full.py` には `_sanitize_nans()` が存在するが `stock_mcp_server.py` には未実装
- **影響**: `fins_data.json` にNaN値が含まれるとブラウザの `JSON.parse()` が失敗する
- **推奨**: `_num()` 関数を修正して `math.isnan()` チェックを追加、またはエクスポート時にNaN→Nullの変換処理を挟む

### .claude/worktrees/ のクリーンアップ
- `cranky-nash` ワークツリーが `.claude/worktrees/` に残存している
- セキュリティリスクではないが、古いコードが蓄積する
- マージ済みであれば `git worktree remove` で削除を推奨
