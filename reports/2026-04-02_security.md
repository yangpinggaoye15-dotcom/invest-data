# 情報セキュリティチーム レポート
日付: 2026-04-02

---

## 総合評価: YELLOW

> 内部監査では重大な問題は検出されず。ただし、本日時点で外部サプライチェーン攻撃・Next.js/Vercel脅威・AI APIソースコード流出など、本システムに直接関連する高リスク外部脅威が複数確認されており、予防的対応を推奨する。

---

## 内部監査結果

| 項目 | 状態 | 詳細 |
|------|------|------|
| コミット履歴 | ✅ | 直近20件（代表: `418e830`）を確認。`key` / `secret` / `password` / `token` のいずれのキーワードも含まれていない。内容は429リトライ処理追加・`PARALLEL_WORKERS=1`削減という機能修正であり、機密情報の混入リスクなし。 |
| CDNスクリプト | ✅ | 今回のコミットは `_fetch_daily` 関数修正のみ。`index.html` への外部CDNスクリプト追加は確認されず。プロジェクトルール違反なし。 |
| APIキー露出 | ✅ | `sk-`・`AIza`・`Bearer` パターンのハードコードはコミット差分内に検出されず。既知の安全設計（Vercel環境変数管理）と整合的。 |
| Vercelプロキシ | ✅ | `api/claude.js` / `api/gemini.js` の変更は今回コミットに含まれない。APIキーはサーバーサイド環境変数経由で管理、GeminiキーはHTTPヘッダー非送信設計を維持。現時点で実装上の問題は確認されていない。⚠️ **ただし後述の外部脅威（CVE-2025-55182, Axios侵害）に対する影響評価を要する。** |
| GitHub Actions | ✅ | 今回のコミットにワークフロー変更なし。シークレット定義（`ANTHROPIC_API_KEY` / `GEMINI_API`）はGitHub Secrets管理で設計上問題なし。⚠️ **ただし後述のGitHub Actionsサプライチェーン攻撃傾向を踏まえ、既存ワークフローの権限設定を確認推奨。** |

---

## 外部脅威情報（Geminiより）

本システム（Python バックエンド / Vercel + Next.js フロントエンド / GitHub Actions CI-CD / Anthropic・Google AI API 使用）に関連する脅威を以下に絞り込む。

### 🔴 高リスク（即時評価推奨）

| # | 脅威 | 関連コンポーネント | 概要 |
|---|------|--------------------|------|
| 1 | **CVE-2025-55182 Next.js RCE（UAT-10608キャンペーン）** | Vercel / Next.js | 2026-04-02現在、少なくとも766ホストが侵害済。認証情報・SSHキー・クラウドトークン・環境シークレットが大規模流出中。本システムのVercelデプロイが対象になりうる。 |
| 2 | **Axios npmパッケージ侵害（RAT混入）** | Next.js / フロントエンド依存関係 | `axios@1.14.1` / `axios@0.30.4` にRAT（`plain-crypto-js`）が混入。本システムの`package.json`でAxiosを使用している場合、APIキー・DB認証情報が流出する可能性。2026-04-01確認。 |
| 3 | **Claude Code CLIソースコード流出** | Anthropic API 利用部分 | 2026-03-31にAnthropicのClaude Code CLIソースコード（51.2万行超）が流出。既存CVE（CVE-2025-59536, CVE-2026-21852）の悪用が容易化し、MCP経由のRCEやAPIキー窃取リスクが上昇。 |
| 4 | **LiteLLM PyPI侵害** | Python バックエンド依存関係 | `litellm==1.82.7 / 1.82.8` にインフォスティーラーが混入。バックエンドの`requirements.txt`でLiteLLMを使用している場合、クラウド認証情報・APIキー・シェル履歴が流出済の可能性。2026-03-31確認。 |

### 🟡 中リスク（今週中に確認推奨）

| # | 脅威 | 関連コンポーネント | 概要 |
|---|------|--------------------|------|
| 5 | **CVE-2026-27978 Next.js CSRF** | Vercel / Server Actions | `origin: null` がCSRF検証をバイパス。Server Actionsを使用している場合、サンドボックス環境（PDF埋め込み等）から攻撃可能。 |
| 6 | **CVE-2026-27977 Next.js 認証バイパス（開発モード）** | 開発環境 / CI環境 | `next dev` 使用時に`Origin: null`でHMR WebSocket接続が可能。GitHub ActionsのCIにdev mode起動が含まれる場合は注意。 |
| 7 | **GitHub Actions サプライチェーン攻撃傾向** | CI/CD | Trivy事例など、タグの書き換えによる悪意コミット注入が継続。`uses: actions/xxx@v3`等のタグ参照を使用している場合にリスク。 |
| 8 | **CVE-2026-4519 Python `webbrowser.open()` コマンドインジェクション** | Python バックエンド | URL引数にハイフンが含まれる場合に任意コード実行の可能性。`_fetch_daily`等でURLを動的生成している箇所を確認推奨。 |
| 9 | **CVE-2026-25645 `requests` 一時ファイル予測可能名** | Python バックエンド | `requests`ライブラリ使用箇所でのローカル攻撃リスク。共有ホスト環境では影響度上昇。 |
| 10 | **インフォスティーラー48時間以内売却** | 開発者端末 / CI環境 | 開発者PCからの認証情報が48時間以内にダークウェブで販売される事例。GitHub Secretsのローテーション判断材料として考慮。 |

### 🟢 低リスク（情報収集・監視継続）

- Langflow CVE-2026-33017（本システムで未使用と推定されるが、AI開発ツール関連として監視）
- Microsoft 365への標的型攻撃（開発チームのコラボレーションツールとして間接リスクあり）
- SNS型投資詐欺・ビジネスチャットなりすまし（本システムのエンドユーザー保護観点）

---

## 要対応事項

### 🔴 緊急（本日〜2026-04-03 中に実施）

**1. Axiosバージョンの緊急確認とロールバック**
```
対象: package.json / package-lock.json
確認コマンド: npm list axios
危険バージョン: axios@1.14.1 / axios@0.30.4
対応: 安全な直近バージョン（例: axios@1.13.x）へ固定し npm audit を実施
副次対応: Vercel環境変数（APIキー・DB認証情報）の即時ローテーション
```

**2. CVE-2025-55182 Next.js パッチ適用状況の確認**
```
対象: package.json の next バージョン
確認: Next.js の最新パッチ済バージョンへの更新可否を確認
暫定措置: Vercelダッシュボードでアクセスログを確認し、
          異常なリクエスト（/.env, /api/../等のパス）がないか精査
```

**3. LiteLLM使用有無の確認**
```
対象: requirements.txt / pyproject.toml / pip freeze
確認コマンド: pip show litellm
危険バージョン: 1.82.7 / 1.82.8
対応: 使用確認された場合 → クラウド認証情報・ANTHROPIC_API_KEY・
      GEMINI_API キーを即時ローテーション
```

### 🟡 今週中（2026-04-07 まで）

**4. GitHub Actions ワークフローのセキュリティ強化**
```yaml
# 現状確認ポイント
- permissions: をワークフロー冒頭で明示的に最小化されているか
  例: permissions: contents: read のみ、など
- actions/xxx@tag 参照をSHA固定に変更
  例: actions/checkout@v4 → actions/checkout@<commit-sha>
- ANTHROPIC_API_KEY / GEMINI_API のシークレットがログ出力されていないか
  （echo $SECRET や env: 経由の露出がないか）
```

**5. Python依存ライブラリの脆弱性スキャン**
```
対象CVE: CVE-2026-4519 / CVE-2026-25645 / CVE-2026-3479
実施: pip-audit または safety check を実行
     _fetch_daily 関数でURL生成にハイフンが含まれる外部入力がないか確認
```

**6. Anthropic APIキーのローテーション（予防的措置）**
```
理由: Claude Code CLIソースコード流出により既存CVEの悪用が容易化
対応: Anthropicコンソールで ANTHROPIC_API_KEY を再発行
     → GitHub Secrets および Vercel Environment Variables を更新
     → 旧キーを即時無効化
     → APIキー使用ログを確認し不審なトークン使用がないか精査
```

---

## 推奨事項

### 短期（4月中）

1. **`pip-audit` / `npm audit` の CI 組み込み**
   - GitHub Actions ワークフローに依存関係スキャンステップを追加し、今回のLiteLLM・Axios事例のような侵害パッケージの自動検出を実現する。

2. **Vercel環境変数のローテーション周期の設定**
   - 今回のサプライチェーン攻撃・インフォスティーラー事例を踏まえ、APIキー類を90日ごとに定期ローテーションするポリシーを策定する。

3. **Next.jsのServer Action利用箇所における `origin: null` バイパス対策（CVE-2026-27978）**
   - Server Actionsのリクエスト元を明示的に検証するミドルウェアを追加し、`null` オリジンを明示的に拒否する実装を検討する。

### 中期（2〜3ヶ月）

4. **SBOMの生成と管理**
   - Python / Node.js 双方の依存ライブラリについてSBOM（Software Bill of Materials）を生成・管理し、サプライチェーン攻撃発生時の影響範囲を即時特定できる体制を整備する。

5. **GitHub Actions ランナーのセルフホスト化 または OpenID Connect（OIDC）への移行**
   - VercelへのデプロイトークンをGitHub Secretsに長期保存するのではなく、OIDCによる短命トークン発行に移行し、トークン窃取時の被害を最小化する。

6. **AIエージェント・MCP利用時のサンドボックス強化**
   - Claude Code CLIソースコード流出を踏まえ、MCPサーバーや環境変数経由でのAPIキーアクセスをネットワークレベルで制限し、ゼロトラスト原則を適用する。

### 監視継続事項

| 監視対象 | 理由 |
|---------|------|
| Vercel デプロイログ（異常パスアクセス） | CVE-2025-55182 悪用キャンペーン継続中 |
| PyPI / npm セキュリティアドバイザリ | LiteLLM・Axios事例のような侵害が継続する傾向 |
| Anthropic セキュリティアドバイザリ | Claude Code CLIソースコード流出後の追加CVE公開リスク |
| GitHub Security Advisories | GitHub Actions サプライチェーン攻撃の継続 |

---

> **監査担当注記**: 今回の総合評価を `YELLOW`（警戒）とした主因は、内部コードの問題ではなく、**本システムの技術スタック（Next.js/Vercel, Python, npm/PyPI依存, Anthropic API）と高い整合性を持つ外部脅威が2026-04-02時点で複数同時進行中**であることによる。Axiosバージョン確認とLiteLLM使用確認の結果次第では、評価を`RED`に引き上げ、全認証情報の即時ローテーションを発動する。