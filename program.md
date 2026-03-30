# post-genspark 自律改善ループ
## autoresearchインスパイア版 — エージェント実行指示書

---

## このファイルの役割

autoresearchの`program.md`に相当する。
AIエージェントはこのファイルの指示に従って、人間の介入なしに自律的にループを回す。

**変更対象 (train.py相当):** `skills/*/SKILL.md`
**固定評価器 (prepare.py相当):** `evaluate_rubric.md` — 変更禁止
**実験ログ:** `results.tsv`
**最適化指標:** `gap_score = genspark_score - skill_score`（低いほど良い）
**改善判定:** `gap_score が前回より下がったら KEEP、変わらないか上がったら DISCARD`

---

## ループの全体フロー

```
[ループ開始]
  ↓
Step 1: 次のトピックを選択（topics.json から未使用のもの）
  ↓
Step 2: 選んだスキルで出力を生成（WebSearch + WebFetch で情報収集）
  ↓
Step 3: 同じクエリを GenSpark に送信してアウトプット取得
  ↓
Step 4: evaluate_rubric.md の5次元で両方を採点 → skill_score / genspark_score / gap_score
  ↓
Step 5: 差分分析 — GenSpark が勝っている点を特定して「仮説」を立てる
  ↓
Step 6: SKILL.md を仮説に基づいて更新
  ↓
Step 7: git commit (KEEP候補) または 何も変えない (DISCARD候補)
  ↓
Step 8: results.tsv に記録
  ↓
Step 9: ループ先頭に戻る
```

---

## Step 1: トピック選択ルール

```python
# 疑似コード
used_topics = [row["topic"] for row in results.tsv]
available = [t for t in topics.json if t["id"] not in used_topics]
# 3スキルをラウンドロビンで回す
next_skill = round_robin(["product-comparison", "service-selection", "travel-planning"])
next_topic = random.choice([t for t in available if t["skill"] == next_skill])
```

**ラウンドロビン順:** product-comparison → service-selection → travel-planning → 繰り返し

---

## Step 2: スキル実行

`SKILL.md` の **核心思考パターン A〜E** に従って出力を生成する。

### 必須チェックリスト（実行前）
- [ ] パターンA: 制約→失敗モード変換 を実行したか
- [ ] パターンB: 全候補の公式ページを並列フェッチしたか（比較記事価格を使っていないか）
- [ ] パターンC: 総コストを変える変数を洗い出したか（シナリオ分岐を作ったか）
- [ ] パターンD: 予算超過を数値で証明し代替を提示したか
- [ ] パターンE: 次の1〜2点の具体的な次アクションを提案したか

---

## Step 3: GenSpark 実行

Chrome MCP を使って GenSpark に同じクエリを送信する。

```
1. tabs_context_mcp で tabId を取得
2. navigate して https://www.genspark.ai に移動
3. javascript_tool で textarea にクエリを入力して Enter を送信
4. 60〜120秒待機
5. get_page_text でアウトプットを取得
6. まだ処理中なら さらに60秒待機して再取得
```

### GenSpark Enter送信のコード（実績あり）
```javascript
const textarea = document.querySelector('textarea');
textarea.focus();
textarea.value = 'クエリ文字列';
textarea.dispatchEvent(new Event('input', { bubbles: true }));
textarea.dispatchEvent(new KeyboardEvent('keydown', {
  key: 'Enter', code: 'Enter', keyCode: 13, bubbles: true
}));
```

---

## Step 4: 採点

`evaluate_rubric.md` の5次元で **スキル出力** と **GenSpark出力** を採点する。

```
skill_score     = 次元1 + 次元2 + 次元3 + 次元4 + 次元5  (max 100)
genspark_score  = 次元1 + 次元2 + 次元3 + 次元4 + 次元5  (max 100)
gap_score       = genspark_score - skill_score
```

**重要:** 採点は両システムを独立に採点した後に差を計算する。
GenSparkの点を先に見てからスキルを採点するバイアスを避ける。

---

## Step 5: 差分分析と仮説生成

採点後、**GenSparkが上回った次元** を特定して仮説を立てる。

### 仮説生成のルール
**禁止:** トピック固有のルールを追加する（「スキー旅行ではレンタル確認」等）
**必須:** 思考パターンの動き方として一般化する

仮説の良い例:
> 「次元1（情報精度）が低い原因: 検索スニペットの価格を使った。
> 仮説: パターンBの並列フェッチを Step 2 ではなく Step 4 で実行するよう順序を変える」

仮説の悪い例:
> 「スキー旅行のレンタル料金を確認するステップを追加する」
> → これはトピック固有の暗記。新しいカテゴリに使えない。

---

## Step 6: SKILL.md の更新

仮説に基づいて `skills/[スキル名]/SKILL.md` を更新する。

### 更新ルール
1. 既存パターン（A〜E）の**動き方を精緻化**する（追加より改善）
2. 新パターンを追加する場合は、**最低2つの異なるカテゴリに適用できる汎用性**があること
3. 変更は最小限に。1ループ1仮説が原則
4. 変更前のバージョンを `versions/` に保存してから更新

```bash
cp skills/[skill]/SKILL.md skills/[skill]/versions/v[new_version].md
# SKILL.md を編集
```

---

## Step 7: git commit / discard の判定

```
if gap_score_now < gap_score_prev:
    status = "KEEP"
    git add skills/[skill]/SKILL.md
    git commit -m "[ループ番号] [スキル名] [トピック]: gap -[改善幅]pt [変更サマリー]"
else:
    status = "DISCARD"
    git checkout -- skills/[skill]/SKILL.md  # 変更を破棄
    # results.tsv には記録する（失敗も学習）
```

---

## Step 8: results.tsv への記録

```tsv
[loop_id]\t[date]\t[skill]\t[topic]\t[skill_score]\t[genspark_score]\t[gap_score]\t[status]\t[key_change]\t[commit_hash]
```

- `key_change`: KEEP なら変更内容の1行サマリー、DISCARD なら「仮説X は効果なし」
- `commit_hash`: `git rev-parse --short HEAD` で取得

---

## 収束条件・停止基準

ユーザーに指定された時間になったら停止。それ以外は自律継続。

### 早期停止すべき例外状況
- GenSpark が3連続でエラー/タイムアウト → ユーザーに報告して停止
- gap_score が3ループ連続で変化なし（DISCARD続き）→ 仮説生成の視点を変えて継続
- SKILL.md が著しく複雑になってきた（パターン数が8を超えた）→ 整理して統合

---

## 進捗レポート（ループN件ごとに出力）

3ループごとに以下を標準出力に表示する:

```
=== 進捗レポート ===
実施済みループ: N件
gap_score 推移: [前回] → [今回]（差分: ±X）
KEEP/DISCARD: K件 / D件
最も改善した変更: [key_change]
現在のgap_score: [値]
次のループ予定: [スキル] × [トピック]
================
```

---

## 思考パターン（参照用）

SKILL.mdに記述されている核心思考パターンの一覧。
ループ中にこれらが機能しているかを常に確認する。

### パターンA: 制約→失敗モード変換
ユーザーの制約条件を「このカテゴリでハマる典型的な失敗モード」に変換してから比較軸を設計する。

### パターンB: 価格は一次情報から並列取得
候補が決まったら全員の公式ページを並列フェッチ。比較記事の価格は使わない。

### パターンC: コストを変える変数を先に洗い出す
総コストを変える隠れた条件を特定してシナリオ分岐を作る。

### パターンD: 予算超過を数値で証明して代替を提示
感想ではなく計算結果で証明する。同軸で比較できる代替を出す。

### パターンE: 次の判断に必要な情報だけを次アクションとして提案
情報の羅列ではなく、次の意思決定ギャップを埋める1〜2点。

---

## 過去のDISCARD仮説ログ（ここに追記していく）

形式: `[日付] [スキル] [仮説] → なぜ効果がなかったか`
(例) `2026-03-30 product-comparison 「製品数を8に増やす」→ 情報過多になりgap_score悪化`
