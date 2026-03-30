# Loop 2 改善ログ
日時: 2026-03-30
バージョン: v1.0.0 → v1.0.1
テーマ: プロジェクト管理ツール エンジニア5人+非エンジニア3人 月額1万円以内

## GenSparkの検索プロセス（リバースエンジニアリング）

### 並列読み取りURL（観察済み）
- atlassian.com/software/jira/pricing（Jira公式）
- nulab.com/pricing/backlog/（Backlog公式）
- linear.app/pricing（Linear公式）
- github.com/pricing（GitHub公式）
- notion.com/integrations/github

### 追加検索クエリ（観察済み）
- `USD JPY exchange rate official today`
- `official USD JPY exchange rate Bank of Japan today`
- `current USD JPY Reuters`
- `ClickUp pricing $7 $10 official unlimited plan`
- `Atlassian Jira standard pricing $8.60 official`

### GenSparkの構造化順序
1. カテゴリ主要ツールの公式料金ページを並列取得
2. USD/JPY為替レートを複数ソースで確認（160円を使用）
3. 全ツールの8人分月額を統一レートで計算
4. 結論をBLUFで冒頭に提示
5. 比較表（予算内5ツール）
6. 予算超過ツールも別記（Asana/Notion/Linear）
7. 「私はこう勧めます」で締め

## 自分の負けポイント

### 負け1: Trelloを見落とした（重大）
- 非エンジニアの定着という観点で非常に有力
- カンバンUIの使いやすさでGenSparkが2位に挙げた
- 私は「Backlog, Notion, Asana, Linear, ClickUp」で検索して視野が狭かった

### 負け2: GitHub Teamを見落とした
- エンジニア5人がいる条件で¥5,100/月と安い
- エンジニア主導チームには有力な選択肢

### 負け3: Jira Freeを見落とした
- 10ユーザーまで無料
- 「まず無料で始める」選択肢を提示できなかった

### 負け4: 為替レートを取得しなかった
- 私は固定の150円想定で計算
- GenSparkは当日160円を確認してすべてに適用（Asana: ¥14,100、Notion: ¥12,800）
- 私のAsana見積もり（¥9,600）は古い円建て価格で、実態と乖離

### 負け5: 予算超過ツールの明示
- GenSparkは「予算オーバーになりやすい候補」セクションを設けた
- ユーザーが後から「予算を少し上げてもいい」となった際に有用な情報

## v1.0.1での改善点
- [x] Step 2: 為替レートを先に検索
- [x] Step 3: プロジェクト管理の典型候補リスト（Trello/GitHub/Jira含む）
- [x] Step 4: 公式料金ページを直接読む
- [x] Step 5: 予算超過ツールも別記
- [x] BLUF構造

## 次のループで確認すること
- 旅行プランニングスキルの作成・実行・GenSpark比較
- または商品比較v1.0.1を別製品で再実行して改善度を検証
