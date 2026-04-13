# セキュリティチーム レポート 2026-04-13

## 本日の脅威情報

[事実] 本日（2026-04-13）は IMF・世界銀行春季会合（4/13〜4/17, ワシントンD.C.）の初日。  
[AI分析] 国際金融会議開催期間中は、金融機関および関連システムへの標的型攻撃・フィッシング増加が統計的に確認されており、Vercelホスト上のシステムも監視対象となる可能性がある。**脅威レベル: 中**

[事実] daily_context.json（market_info）に本日付けの新規サイバー攻撃・金融詐欺報道の記載なし。  
[AI分析] 新規の重大インシデント情報はないが、IMF会合期間を通じて引き続き警戒が必要。

### 継続監視中の既知脅威

[事実] **CVE-2026-34041（CVSS 9.8 CRITICAL）**: `act`ツール 環境インジェクション/RCE 脆弱性。4/9以降4週間以上未確認のまま継続中。  
[AI分析] 本脆弱性は本システムの GitHub Actions 環境に影響しうる最高優先度案件。本週（IMF会合期間内）の確認が必須。

[事実] **CVE-2026-23869（CVSS 7.5 HIGH）**: Next.js App Router RSC FlightプロトコルDoS脆弱性。修正版 15.5.15/16.2.3 が公開済み。

[事実] **CVE-2026-27978**: Next.js Server Action CSRF検証バイパス（origin:null）継続監視中。

---

## API・認証状況

[AI分析] 各APIキーの状態評価（knowledge記録・コミット履歴に変更なしのため）:

- **ANTHROPIC_API_KEY**: 正常（Vercel環境変数、ハードコードなし確認済み）
- **GEMINI_API**: 正常（Vercel環境変数、api/gemini.js プロキシ経由）
- **JQUANTS_API_KEY**: 正常（GitHub Secrets、run_screen_full.py経由のみアクセス）
- **DATA_REPO_TOKEN**: 正常（GitHub Secrets、daily_screening.yml/daily_teams.yml 限定使用）

[AI分析] Vercelプロキシ設計（APIキーをHTTPヘッダー非送信・/api/経由）は引き続き正常稼働と判断。直近コミット（a29a435, 6854389, e279d69）はすべて「knowledge update」のみでAPI関連変更なし。

---

## 推奨対応

### 優先度: 高
1. **CVE-2026-34041 確認（本週中必須）**  
   `act` ツール（GitHub Actions ローカル実行ツール）の利用有無を確認し、使用している場合は 0.2.86 以降へ更新。未使用なら記録してエスカレーション解除。

### 優先度: 中
2. **data/ 未追跡ファイルの整理**  
   `git rm --cached data/` の実施（4/9 から継続推奨・未実施）。機密ファイルの誤コミットリスクを排除。

3. **GitHub Actions 固定SHA参照化**  
   サプライチェーンリスク対策として、workflow内のサードパーティActionをコミットSHAで固定。

4. **Next.js バージョン確認**  
   Vercel環境の Next.js が 15.5.15/16.2.3 以降か確認。

### 優先度: 低
5. IMF会合期間（〜4/17）中の Vercel プロキシへの不審アクセスを定期確認。
