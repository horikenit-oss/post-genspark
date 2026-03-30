# post-genspark 自律改善ループ
## autoresearch完全準拠版 — エージェント実行指示書

---

## このファイルの役割

karpathy/autoresearch の `program.md` に完全準拠した自律ループ指示書。
AIエージェントはこのファイルの指示に従い、**人間の介入なしに無限ループを回す（NEVER STOP）**。

### autoresearchとの対応関係

| autoresearch | post-genspark |
|---|---|
| `train.py` — 実験対象 | `skills/*/SKILL.md` — 実験対象 |
| `prepare.py` — 固定評価器 | `evaluate_rubric.md` — 変更禁止 |
| `program.md` — エージェント指示書 | このファイル |
| `results.tsv` — **gitで管理しない**浮動ファイル | `results.tsv` — **gitで管理しない** |
| `val_bpb` — 最適化指標（低いほど良い） | `gap_score = genspark_score - skill_score`（低いほど良い） |
| git commit → KEEP / git reset → DISCARD | 同じ |

---

## セットアップ（セッション開始時に1回だけ）

```bash
# 1. ブランチ確認
git branch  # autoresearch/mar30 等の実験ブランチにいることを確認

# 2. ファイル確認
# 読むもの: program.md（このファイル）, evaluate_rubric.md, skills/*/SKILL.md
# 絶対に変更しないもの: evaluate_rubric.md
# 実験で変更するもの: skills/*/SKILL.md

# 3. results.tsv の初期化（存在しなければ作成）
# results.tsv は .gitignore に入っており、git管理されない
# ヘッダー行のみで始める:
echo "commit\tgap_score\tskill_score\tgenspark_score\tstatus\tskill\ttopic\tdescription" > results.tsv

# 4. 確認したらループ開始
```

---

## ループの全体フロー（autoresearch準拠）

```
[ループ開始]
  ↓
Step 1: git 状態の確認
  ↓
Step 2: 仮説を立て、SKILL.md を修正する
  ↓
Step 3: git commit（実行前にコミット — autoresearch と同じ）
  ↓
Step 4: スキルを実行してアウトプットを生成
  ↓
Step 5: 同じクエリを GenSpark に送信してアウトプット取得
  ↓
Step 6: evaluate_rubric.md の5次元で両方を採点
  ↓
Step 7: gap_score を前回と比較 → KEEP or DISCARD
  ↓
Step 8: results.tsv に記録（git commit しない）
  ↓
Step 1 に戻る（NEVER STOP）
```

---

## Step 1: git 状態の確認

```bash
git log --oneline -5        # 直近のコミット履歴
git branch                  # 現在のブランチ（autoresearch/<tag>にいること）
git status                  # 未コミットの変更がないことを確認
```

現在のベストgap_scoreは `results.tsv` の直近KEEPレコードから確認。

---

## Step 2: 仮説生成と SKILL.md 修正

**トピック選択:**
- `topics.json` から未使用のトピックを1つ選ぶ
- ラウンドロビン: product-comparison → service-selection → travel-planning → 繰り返し

**仮説生成ルール:**
- 前回のDISCARD/gapの原因から「どの思考パターンが機能していなかったか」を特定
- **禁止**: トピック固有のルール追加（「スキー旅行ではレンタル確認」等）
- **必須**: どのカテゴリにも適用できる汎用的な動き方として記述

仮説の良い例:
> 「カテゴリランキングを参照して未知候補を発見する（Pattern F）」
> → コーヒーメーカーでも、SaaSでも、観光地でも同じ動きで機能する

仮説の悪い例:
> 「スキー旅行のレンタル料金確認ステップを追加する」
> → トピック固有の暗記。新カテゴリに使えない

**SKILL.md 修正:**
```bash
# バージョン番号を上げて修正
# 例: version: 2.0.1 → 2.0.2
```

---

## Step 3: git commit（実行前）

autoresearch と同様に、**実行前にコミットする**。
これにより、DISCARDのときに `git reset` で確実に元に戻せる。

```bash
git add skills/<skill>/SKILL.md
git commit -m "Loop<N> [<skill>] hypothesis: <仮説の1行サマリー>"
```

例:
```bash
git commit -m "Loop11 [product-comparison] hypothesis: add ranking-based discovery (Pattern F)"
```

---

## Step 4: スキルを実行

`skills/<skill>/SKILL.md` の **核心思考パターン A〜F** に従って出力を生成する。

### 実行前チェックリスト
- [ ] パターンA: 制約→失敗モード変換 を実行したか
- [ ] パターンB: 全候補の公式ページを並列フェッチしたか
- [ ] パターンC: 総コストを変える変数を洗い出したか
- [ ] パターンD: 予算超過を数値で証明し代替を提示したか
- [ ] パターンE: 次の1〜2点の具体的な次アクションを提案したか
- [ ] パターンF: カテゴリランキングで未知候補を発見したか

---

## Step 5: GenSpark を実行

Chrome MCP で同じクエリを GenSpark に送信する。

```javascript
// javascript_tool で実行
const textarea = document.querySelector('textarea');
textarea.focus();
textarea.value = 'クエリ文字列';
textarea.dispatchEvent(new Event('input', { bubbles: true }));
textarea.dispatchEvent(new KeyboardEvent('keydown', {
  key: 'Enter', code: 'Enter', keyCode: 13, bubbles: true
}));
```

待機: 最低60秒、完了まで60秒ずつ延長。

---

## Step 6: 採点（evaluate_rubric.md 準拠）

```
skill_score     = 次元1 + 次元2 + 次元3 + 次元4 + 次元5  (max 100)
genspark_score  = 次元1 + 次元2 + 次元3 + 次元4 + 次元5  (max 100)
gap_score       = genspark_score - skill_score
```

採点はスキル出力とGenSpark出力を**独立に**採点してから差を計算する。

---

## Step 7: KEEP / DISCARD 判定

```python
if gap_score_now < gap_score_prev:
    status = "keep"
    # コミットをそのまま維持
else:
    status = "discard"
    git reset --hard HEAD~1   # 直前のコミットを取り消す
    # SKILL.md が前の状態に戻る
```

**簡潔さボーナス:**
- gap_scoreが同じでもSKILL.mdが短くなった（不要なルールを削除した）場合は "keep"
- 微小な改善でも複雑な記述が増えた場合はDISCARDを検討

---

## Step 8: results.tsv に記録（git commit しない）

```bash
# コミットハッシュを取得
COMMIT=$(git rev-parse --short HEAD)

# results.tsv に追記（git管理しない）
echo "${COMMIT}\t${GAP}\t${SKILL}\t${GENSPARK}\t${STATUS}\t${SKILL_NAME}\t${TOPIC}\t${DESC}" >> results.tsv
```

フォーマット:
```
commit	gap_score	skill_score	genspark_score	status	skill	topic	description
```

例:
```
3bb7c9e	26	68	94	discard	product-comparison	コーヒーメーカー	hypothesis: pattern F ranking check
faab82d	19	73	92	keep	product-comparison	ワイヤレスイヤホン	pattern F confirmed: toffy/anker discovered
```

---

## KEEP/DISCARD の具体例

```
# KEEP の場合（gap改善）
→ コミットはそのまま残る
→ results.tsv: status=keep

# DISCARD の場合（gap悪化）
→ git reset --hard HEAD~1  で直前のコミットを削除
→ SKILL.md が前の状態に戻る
→ results.tsv: status=discard（失敗も記録する — 学習のため）
→ program.md の「過去のDISCARD仮説ログ」に追記
```

---

## NEVER STOP

ユーザーに指定された時間か手動停止まで、**確認なしに継続する**。

3ループごとに進捗レポートを出力:
```
=== 進捗レポート ===
実施済みループ: N件
best gap_score: X (Loop M, <スキル>)
KEEP/DISCARD: K件 / D件
現在のブランチ: autoresearch/mar30
最新コミット: <hash> <message>
次: <スキル> × <トピック>
=================
```

### 早期停止すべき例外
- GenSpark が3連続タイムアウト → 報告して停止
- git reset がエラー → 報告して停止（それ以外は継続）

---

## 過去のDISCARD仮説ログ

形式: `[日付] [スキル] [仮説] → なぜ効果がなかったか`
（ここに追記していく）
