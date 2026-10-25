---
name: usausa-verbatim-ppt-designer
description: "「世界一エンジニア兼PPT作成クリエイター」として、要件書・README・仕様書・原稿の文字を一字一句変えずに（改変・要約・追加の禁止）、目的とペルソナ別に認知科学・人間中心設計・色彩学（査読済み論文・規格）に基づいてキレイな法人研修レベルのPPTに仕上げるSkill。文字を保ったままのデザイン、台帳照合による改変ゼロ検証、インデント・行と列の整列・アイコン（サイズ/余白/縦位置）の微調整と統一までを一貫して行う。「要件書からPPT」「READMEをスライドに」「仕様書をPPTに」「文字は変えないで」「改変禁止」「デザインだけ整えて」「ペルソナ別」「認知科学で」「色彩学で」「人間中心設計で」「インデントを揃えて」「行列を揃えて」「アイコンを統一して」「微調整して」「法人研修レベル」といった依頼があれば、明示的にSkillと言われなくても必ず使う。既存PPTの体裁だけを直す場合（文字は触らない）にも使う。"
---

# 文字改変禁止PPTデザインSkill（世界一エンジニア兼PPT作成クリエイター）

**核心：文字は台帳から一字一句コピーされる“構造”にして、デザインだけを自由に最適化する。**
「気をつけて変えない」ではなく、「変えられない作り」にする。検証スクリプトが改変を機械的に検出する。

## 3つの鉄則
1. **文字は変えない**：要約・言い換え・誤字訂正・句読点の統一・全角半角の変更・語の追加をしない。
   誤りに気づいたら直さず、報告に「原本の疑義」として書く。
2. **デザインは根拠で決める**：ペルソナ×目的→トークン→全スライドがトークンから生成される。
   スライドごとに色・位置・サイズを手で決めない。根拠は `references/evidence.md`（引用前に必ず出典照合）。
3. **仕上げは微調整の機械化**：行・列・インデント・アイコンは `normalize_layout.py` で統一し、報告する。

## 全体フロー
1. **要件把握**（原本・目的・ペルソナ・使う場面・ブランド色/フォントを確認。足りない分だけ1回でまとめて質問）
2. **台帳を作る**：`make_ledger.py`（原本→ledger.json）
3. **トークンを作る**：`make_tokens.py` → `palette_check.py`（配色の検査）→ 要点をユーザーに確認
4. **スライド計画を作る**：`slide-plan.yaml`（ブロックIDで参照。文章は書かない）。構成案を先にユーザーへ見せて合意
5. **ビルド**：`build_deck.py`
6. **整列の統一**：`normalize_layout.py`（ドライラン→報告→`--apply`）
7. **検査**：`verify_verbatim.py` → 画像化して全枚目視 → `references/qa-checklist.md`
8. **報告**

スクリプトは `scripts/`（このSkillのフォルダ）にある。`cd` して実行するか絶対パスで呼ぶ。依存：python-pptx, PyYAML, python-docx（docx原本のとき）。

## ステップ1：要件把握
聞く項目（既に分かっているものは聞かない）：
- 原本ファイル（.md/.txt/.docx。PDFは先に `pdf-reading` Skillで本文を取り出す）
- **目的**：講義（投影）／自習・配布／報告・意思決定／手順書 → `lecture|self_study|decision|manual`
- **ペルソナ**：非IT初心者／新卒／中堅DX／経営層 → `novice|new_grad|mid_engineer|executive`
- ブランド主色(hex)・日本語フォント・フッター文言（無ければ既定を使い、仮置きと報告）
- 企業テンプレ（.pptx）がある場合は、そのレイアウトを使う旨を伝え `usausa-corporate-rule-ppt` と併用する

## ステップ2：台帳
```bash
python3 scripts/make_ledger.py 原本.md -o ledger.json
```
- 見出し/段落/箇条書き/番号/コード/表の行をブロックID（B001…）にする。
- Markdown記号（#, -, *）は構造情報に移し本文は変えない。`**` と `` ` `` は外れるので、
  出力に「記号を外したブロック」が出たら**必ずユーザーに報告**する。
- 台帳は読み取り専用の“唯一の正”。以後、文字を直接タイプしない。

## ステップ3：トークン
```bash
python3 scripts/make_tokens.py --persona novice --purpose lecture --primary 1F5A99 --font-jp "Meiryo UI" --footer "社名" --author "作成者名" -o tokens.yaml
python3 scripts/palette_check.py tokens.yaml
```
- 数値の意味は `references/persona-matrix.md`。主色が白と4.5:1未満なら自動で暗くする（報告する）。
- NGが出たら配色を直す（基準を緩めない）。色だけで意味を区別しない。
- 作成者（`--author`、未指定ならfooter）は文書プロパティに入る。空だとビルド時に警告 → ユーザーに確認する。
- 日本語フォントは**必ず明示**（未指定だと代替フォントで崩れる）。配布先PCに入っているものを選ぶ。

## ステップ4：スライド計画
`assets/slide-plan.sample.yaml` を参考に。layout は `cover/section/text/two_col/cards/steps/table/code`。
- 選び方は `references/persona-matrix.md` の「原本に合わせたレイアウトの選び方」。
- 1スライド1メッセージ。タイトルは原本の見出しをそのまま使う（結論型に書き換えない）。
- **入りきらない時は文字を削らず、サイズを最小まで下げ、それでも溢れるならブロック境界で分割**。
- 1ブロックを複数スライドに重複して載せない。原本のブロックは全て配置する（未配置は報告）。
- 発表者メモ(`notes`)にも台帳の文字だけを使える。
- アイコンは `icon: 名前`。`icons/名前.png` があればそれを使い、無ければ幾何学グリフ（文字を使わない）。
  法人納品では企業指定のアイコンPNGを `--icons DIR` で渡すのが望ましい。アイコンは必ず見出しの文字と併置する。

## ステップ5〜6：ビルドと整列
```bash
python3 scripts/build_deck.py tokens.yaml slide-plan.yaml ledger.json out.pptx [--icons icons/]
python3 scripts/normalize_layout.py out.pptx tokens.yaml                    # ドライラン（報告のみ）
python3 scripts/normalize_layout.py out.pptx tokens.yaml --apply -o out.fixed.pptx --report norm.md
```
`normalize_layout.py` がやること（文字は不変、実行前後の全段落を照合し1文字でも違えば保存しない）：
- **スナップ**：位置・サイズをグリッド（既定1/16インチ）へ
- **行・列の整列**：近い左端/上端を揃え、同じ行・列で3個以上並ぶ要素は幅・高さを統一して等間隔に
- **入れ子の追従**：カードの中身はカードの変形に追従
- **インデント**：階層ごとのマージンと箇条書きのぶら下げ、行間・段落間を統一
- **アイコン**：スライド内で同一サイズ（規定サイズに丸める）、文字との間隔を一定にし、見出し1行目の縦中心に合わせる
他人が作った既存PPTにも使える（トークンで基準を渡す）。補正件数が多い場合は、先にドライラン結果をユーザーに見せる。

## ステップ7：検査（提示前に必須）
```bash
python3 scripts/verify_verbatim.py out.pptx ledger.json --strict-coverage
python3 /mnt/skills/public/pptx/scripts/office/soffice.py --headless --convert-to pdf out.pptx
pdftoppm -jpeg -r 60 out.pdf slide      # 全枚を画像で目視
```
- `verify_verbatim.py`：台帳に無い段落が1つでもあれば NG。フッター・ページ番号（名前が `chrome:`）のみ許可。
- LibreOfficeは日本語フォントが代替になるため、はみ出しは厳しめに出る。余白に約10%の余裕を見る。
- `references/qa-checklist.md` を全項目確認。
- **納品前は `usausa-ppt-spartan-review`（観点表58項目・合否判定）で第三者テストにかける。** 作成者の自己チェックだけで終わらせない。
  このSkillの出力は自動検査（A〜H）を通る設計だが、目視項目（余白のリズム・アイコンの意味・語の途中の改行）は別途FAILしうる。

## ステップ8：報告（短く）
- 文字：照合結果（OK／NG件数）、台帳作成時に外した記号、原本の疑義（誤字らしき箇所を直さず指摘）
- デザイン：ペルソナ×目的、主色・フォント（仮置きは明記）、palette_checkの結果
- 微調整：補正件数（図形／アイコン／インデント）
- 警告：溢れ・分割推奨・未配置ブロック
- 引用した論文は**照合できたものだけ**。未確認は書かない

## 既知の限界（スパルタ・テストで確認済み）
- 内蔵アイコンは**幾何学図形で意味を持たない**（星・円・三角など）。意味のあるアイコンは企業指定PNGを `--icons` で渡す。
- 日本語の折り返しは語の途中（例：「公式」→「公／式」）で切れることがある。孤立行（最終行1〜2文字）は避けるが、語の途中改行は避けきれない。文字は変えられないため、カード幅・分割で対処する。
- コード枠・カードは内容量に合わせた高さだが、スライドは上詰めで下に余白が残る（全体の縦中央寄せはしない）。
- 整列ツールは「同じ行/列に3個以上の兄弟」「積み重ね（帯＋カード）」「入れ子（カード＞中身）」を想定。想定外の凝ったレイアウトは、ドライラン結果を見て適用範囲を決める。

## 注意
- 企業ブランドルールの検査は `usausa-ppt-review-and-fix`、企業テンプレ準拠の新規作成は `usausa-corporate-rule-ppt` と併用できる。
- 表紙・章扉など“構造上必要な見出し”も台帳の文字を使う。新しい文言（例：「続き」）が必要なら、追加してよいかをユーザーに確認し、
  承認時のみ `verify_verbatim.py --allow "続き"` を使う。
- 編集方針は [[editorial-philosophy]]（うさうさの原則）を踏襲：断定しすぎない／事実と所感を分ける／出典を捏造しない。

## ファイル構成
```
usausa-verbatim-ppt-designer/
├── SKILL.md
├── scripts/  make_ledger.py / make_tokens.py / palette_check.py / build_deck.py /
│             normalize_layout.py / verify_verbatim.py / common.py
├── assets/   slide-plan.sample.yaml / design-tokens.sample.yaml / sample-source.md
└── references/ evidence.md / persona-matrix.md / qa-checklist.md
```
