# Loop 8 改善ログ
日時: 2026-03-30
バージョン: v1.0.2 → v1.0.3
テーマ: クラウドストレージ 個人+仕事兼用 予算月2,000円以内

## GenSparkの検索プロセス（確認済み）

### 並列読み取りURL（第1ラウンド）
- https://one.google.com/about/plans?hl=ja
- https://www.microsoft.com/ja-jp/microsoft-365/onedrive/onedrive-plans-and-pricing
- https://www.dropbox.com/ja/plans
- https://www.apple.com/jp/icloud/
- https://www.sync.com/pricing-individual/
- https://www.pcloud.com/ja/cloud-storage-pricing-plans.html

### 並列読み取り（第2ラウンド：ファミリー共有・機能詳細）
- Google One 公式・ファミリー共有
- Microsoft OneDrive Personal Vault
- Dropbox 同期機能詳細
- iCloud ファミリー共有
- Sync.com セキュリティ詳細

### GenSparkが使ったプロセスの特徴
1. 全サービスの公式料金ページを直接読んで価格を正確に確認
   → Google One 2TB: ¥1,450（私は¥1,300と誤記）
   → iCloud 2TB: ¥1,500（私は¥1,300と誤記）
2. BLUF：冒頭で「仕事→Microsoft / 2TB→Google or Apple / 同期→Dropbox」と明示
3. 各サービスの弱点を正直に明示
   例: Microsoft「2TBではない」、iCloud「Windows中心の仕事だと弱い」、Dropbox「円安で体感コストがぶれやすい」
4. Sync.comも調べたが最終推薦には含めず（品質フィルタリング）
5. 「迷ったらこの選び方でOK」という具体的な使用パターン別判断ツリー
6. ライセンス構造：最大5台同時利用、ファミリー共有を確認して明記

## v1.0.2の改善状況

### 改善できた点（v1.0.2が機能した）
- ✅ ライセンス構造確認（ファミリー共有・マルチデバイス台数）
- ✅ 為替レート確認（1 USD ≈ 160円）
- ✅ pCloudのライフタイムプランを含めた（GenSparkは含めなかったが有用情報）
- ✅ BLUF構造

### 残った差分（v1.0.3で対処）
1. **公式料金を直接確認せずに価格ミスが発生**（¥1,300→正しくは¥1,450/¥1,500）
   → 第2ラウンドで公式URLを直接フェッチして確認することを必須にする
2. 「迷い解消ツリー」（使用パターン別の最終判断ガイド）がなかった
3. 各サービスの弱点の具体性が足りなかった

## スコア推定
- v1.0.1（プロジェクト管理ツール/ビデオ会議平均）: 60点
- v1.0.2（クラウドストレージ）: 73点 vs GenSpark 88点
→ v1.0.3は78〜82点を目指す（特に価格精度の改善）
