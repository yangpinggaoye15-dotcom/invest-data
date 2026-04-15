# Fix Report 2026-04-14 19:52 JST

## 実施した修正
| # | ファイル | 問題 | 修正内容 | 発見元 |
|---|---------|------|---------|--------|
| 1 | simulation_log.json | 5803フジクラ: current_price/pct/days_elapsed が4/13値のまま | current_price 5698→5935, current_pct 18.71→23.65%, days_elapsed 5→6 | Team8 |
| 2 | simulation_log.json | 5010日本精蝋: current_price/pct/days_elapsed が4/9エントリー値のまま | current_price 241→243, current_pct 0→0.83%, days_elapsed 3→4 | Team8 |
| 3 | simulation_log.json | 7014名村造船: current_price/pct/days_elapsed が4/9エントリー値のまま | current_price 4340→4175, current_pct 0→-3.80%, days_elapsed 3→4 | Team8 |
| 4 | simulation_log.json | 3076Aiホールディングス: current_price/pct/days_elapsed が4/11エントリー値のまま | current_price 2872→2793, current_pct 0→-2.75%, days_elapsed 2→3 | Team8 |
| 5 | simulation_log.json | 3003ヒューリック: current_price/pct/days_elapsed が4/11エントリー値のまま | current_price 1930→1894.5, current_pct 0→-1.84%, days_elapsed 2→3 | Team8 |
| 6 | simulation_log.json | 3003ヒューリック: 4/13 daily_logが1930.0（エントリー値）のまま。実際は1,880.5円でSMA50トリガー発動済み | price 1930→1880.5, daily_pct 0→-2.57%, cumulative_pct 0→-2.57%, bear確率引上げ | Team8 |
| 7 | simulation_log.json | last_updated が 2026-04-13 のまま | 2026-04-14 に更新 | 自動監査 |
| 8 | knowledge/info_patterns.md | 250行（200行超・CLAUDE.mdルール#8違反）| 最古エントリー「2026-04-11 初回」ブロック（55行）を削除 → 195行 | 自動監査 |

---

## knowledge ファイルサイズ監査
| ファイル | 行数 | 状態 |
|---------|------|------|
| info_patterns.md | 195 | ✅（修正後・250→195） |
| risk_patterns.md | 175 | ⚠️ |
| audit_patterns.md | 174 | ⚠️ |
| hr_patterns.md | 165 | ✅ |
| security_patterns.md | 164 | ✅ |
| analysis_patterns.md | 141 | ✅ |
| report_patterns.md | 133 | ✅ |
| verification_patterns.md | 112 | ✅ |
| strategy_patterns.md | 103 | ✅ |
| fix_patterns.md | 39 | ✅ |
| event_patterns.md | 31 | ✅ |

---

## invest-data push 確認
- 最新コミット: 2026-04-14 daily report 2026-04-14 (scheduled)
- 状態: ✅ 本日実施済み

---

## スキップした問題と理由
| 問題 | スキップ理由 |
|------|------------|
| CVE-2026-34041 未対応（Team6指摘） | 修正NG対象ファイルの可能性。オーナー判断が必要 |
| GitHub Actions SHA固定化未実施（Team6） | .github/workflows/ は修正NG対象 |
| earnings_grade全欠損（Team5指摘） | スクリーニングロジック変更が必要。スコープ外 |
| Team2シミュレーション銘柄不完全記載（Team5） | analysis.md は Team2 生成。修正権限なし |
| 5803 daily_log 日付重複（4/8・4/9 各2件） | 前回より継続スキップ。意図的記録か判断不可 |
| 3003 4/15寄り付き損切り実行（Team8推奨） | 実際の損切りはオーナー判断事項。4/15 Fix時にシミュレーションログへ反映 |
