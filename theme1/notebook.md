# theme1 研究ノート

## 初回設定

### 研究テーマ

耐量子計算機暗号 UOV (Unbalanced Oil and Vinegar; NIST PQC Additional Digital Signature Schemes Round 2) の安全性評価。特に、既存の最も効率のよい攻撃を基準にし、それを上回る可能性のある攻撃手法を探索し、正当性と計算量評価を小パラメータ実験で検証する。

### 背景

UOV は多変数二次多項式系に基づく署名方式であり、NIST の追加署名標準化プロセス Round 2 に残っている候補である。人間からの指示では、単なる安全性概説ではなく「可能な限り効率のよい攻撃」を提案し、少なくとも既存最良攻撃より高効率な攻撃を目指すことが求められている。

### 主要な研究問い

- 既存文献・仕様書で最良とされる UOV 攻撃は何か。
- UOV Round 2 推奨パラメータに対して、既存最良攻撃の計算量評価は上界か下界か、またどの仮定に依存するか。
- 既存攻撃を上回る余地は、直接攻撃、衝突攻撃、Kipnis-Shamir 攻撃、intersection attack、MinRank 系攻撃のどこにあるか。
- 提案攻撃の正当性と計算量を、小パラメータで再現可能に検証できるか。

### 明らかにしたい範囲

- まず UOV 本体を対象にする。QR-UOV、MAYO、SNOVA など UOV 派生方式は比較対象として扱うが、主対象にはしない。
- 初期段階では鍵回復攻撃と偽造攻撃の両方を候補にする。
- 実験は小さい有限体・小さい `(n, m, q)` から始め、実装の正しさを優先する。
- 実用パラメータへの外挿は、上界・下界・経験的推定を明確に分ける。

### 想定される成果物

- 既存攻撃の整理表。
- 攻撃仮説と計算量モデル。
- 小パラメータ実験コード。
- 実験コードの再現マニュアル。
- 金曜日の中間報告スライド。

### 評価基準

- 新規性: 既存攻撃の単なる再説明ではなく、探索空間、前処理、複数ターゲット化、部分方程式化などで明確な改善点があるか。
- 正当性: 攻撃が実際に署名偽造または鍵回復につながることを数学的に説明できるか。
- 計算量評価: 実装上界、理論上界、下界、経験的推定を混同していないか。
- 再現性: 小パラメータで第三者が同じ結果を再現できるか。
- 限界: 実用パラメータへ外挿できない箇所、ランダムオラクル仮定、メモリ量、線形代数定数を明示しているか。

### 初日の作業計画

1. `chat.md` の人間指示を確認する。
2. UOV Round 2 の公式仕様と NIST 状態を確認する。
3. 既存攻撃の基準値を抽出する。
4. 既存最良を上回る可能性がある攻撃仮説を、未検証として明確に分けて記録する。
5. 次回の実験設計を決める。

## 2026-06-16

### 今日の目的

実体ディレクトリ `C:\Users\Rosmo\03_ResDebWri\Research\Projects\ProjectLongThink\theme1` の `chat.md` と `notebook.md` を基準に、UOV 安全性評価テーマを開始する。今日は既存攻撃の基準値と、改善を狙う仮説を整理する。

### 実施した作業

- 実体側の `theme1/chat.md` を読み、人間からの最新指示を確認した。
- 実体側の `theme1/notebook.md` が空であることを確認した。
- NIST の Round 2 Additional Signatures ページを確認し、UOV が Round 2 候補として掲載されていることを確認した。
- UOV Round 2 仕様書 Version 2.0 を確認し、推奨パラメータと既存攻撃の計算量表を抽出した。
- 既存攻撃を上回る候補として、部分ターゲット直接攻撃と衝突探索を組み合わせる方向を仮説として設定した。

### 得られた知見

#### 事実

- NIST の Round 2 Additional Signatures ページには、Multivariate Signatures の候補として MAYO、QR-UOV、SNOVA、UOV が掲載されている。UOV の仕様書、zip file、公式 Web site へのリンクも掲載されている。確認日: 2026-06-16。
- NIST IR 8528 は、追加署名プロセスの第一ラウンド評価後に 14 候補を第二ラウンドへ進めたと記しており、その中に UOV が含まれる。確認日: 2026-06-16。
- UOV Round 2 仕様書 Version 2.0 は 2025-02-05 付で、推奨パラメータとして `uov-Ip (n=112, m=44, q=256)`, `uov-Is (n=160, m=64, q=16)`, `uov-III (n=184, m=72, q=256)`, `uov-V (n=244, m=96, q=256)` を示している。
- 同仕様書の Table 5 は、既存攻撃の bit-complexity estimate を「攻撃に必要な binary gates 数の base-2 対数の lower bound」として示している。
- Table 5 によれば、仕様書上の主要な基準値は次の通りである。
  - `uov-Ip`: collision 191, direct 145 (`k=2`), Kipnis-Shamir 218, intersection 166 (`k=2`)。仕様書表中では direct attack が最小。
  - `uov-Is`: collision 143, direct 165 (`k=12`), Kipnis-Shamir 154, intersection 176 (`k=3`)。仕様書表中では collision attack が最小。
  - `uov-III`: collision 303, direct 218 (`k=4`), Kipnis-Shamir 348, intersection 250 (`k=2`)。仕様書表中では direct attack が最小。
  - `uov-V`: collision 399, direct 278 (`k=6`), Kipnis-Shamir 445, intersection 312 (`k=2`)。仕様書表中では direct attack が最小。
- 仕様書は direct attack について、公開鍵の MQ 系を解く偽造攻撃として説明し、Thomae-Wolf 型の未定系削減で `m' = m - 1`, `n' = m - 1` に落とした後、hybrid WiedemannXL を使う見積もりを採用している。
- Faugere and Perret (2009) は UOV に対する hybrid Groebner basis 型の実用的攻撃を論じ、当時提案されていたパラメータに対して `2^40.3` の上界または 9 時間程度の計算を報告している。ただしこれは現在の Round 2 推奨パラメータそのものへの攻撃結果ではない。

#### 推論

- 仕様書 Table 5 を基準にすると、Round 2 UOV に対して既存最良を上回るには、少なくとも `uov-Ip` で direct attack の `2^145`、`uov-Is` で collision attack の `2^143`、`uov-III` で direct attack の `2^218`、`uov-V` で direct attack の `2^278` を下回る必要がある。
- `uov-Is` 以外では direct attack が最小なので、改善余地を探す第一候補は「直接攻撃の変形」または「直接攻撃と衝突探索の混合」である。
- 仕様書の collision attack 評価は大規模メモリの扱いを慎重に見ており、Table 5 は下界として出されている。したがって、実装上界を主張する場合はメモリコストを別途評価しなければならない。
- MinRank attack は UOV には適用可能だが、仕様書は推奨パラメータでは他攻撃より大きいコストと述べている。よって最初の実験対象としては direct/collision 系の方が優先度が高い。

#### 仮説

- 仮説 H1: 部分ターゲット直接攻撃と衝突探索の混合。`a < m` 個の出力座標だけを対象に `P_{[a]}(s) = h_{[a]}` を解き、多数の解または解候補を生成し、残り `m-a` 座標を salt 列挙または残差評価で合わせる。`a` を調整することで、完全な direct attack より MQ 解法コストを下げつつ、純粋な collision attack より探索空間を狭められる可能性がある。
- H1 の計算量は、少なくとも `C_solve(a) + C_sample(a, N) + C_match(m-a, N, Y)` の形で分解して評価する必要がある。ここで `N` は生成する署名候補数、`Y` は試す salt 数である。この評価は現時点では未検証であり、上界・下界のどちらでもない作業仮説である。
- 仮説 H2: `uov-Ip` の intersection attack は仕様書中で境界的な条件に触れているため、失敗確率や反復回数を実験的に評価し直す価値がある。ただし、これだけで Table 5 を上回る見込みは現時点では弱い。
- 仮説 H3: 仕様書の direct attack 評価に含まれる WiedemannXL/XL の次数・線形代数コスト・定数を小パラメータで再測定すると、実装上界としては表の lower bound と異なる見え方になる可能性がある。ただし、これは「既存最良を上回る新攻撃」ではなく、評価精密化に近い。

#### 未検証事項

- H1 が既存の multi-target direct attack または collision attack の既知変形として既に否定・包含されていないか。
- `P_{[a]}(s)=h_{[a]}` の解候補を効率よく多数生成できるか。
- 解候補分布が残り座標について十分ランダムに近いか。
- salt 列挙と候補署名列挙を組み合わせた場合のメモリ量と並列化可能性。
- 小パラメータ実験で観測される成功確率が、実用パラメータへどの程度外挿可能か。

### 根拠・出典

- NIST, Post-Quantum Cryptography: Additional Digital Signature Schemes, Round 2 Additional Signatures. https://csrc.nist.gov/projects/pqc-dig-sig/round-2-additional-signatures 確認日: 2026-06-16。
- NIST, IR 8528, Status Report on the First Round of the Additional Digital Signature Schemes for the NIST Post-Quantum Cryptography Standardization Process. https://csrc.nist.gov/pubs/ir/8528/final 確認日: 2026-06-16。
- UOV submission team, UOV: Unbalanced Oil and Vinegar, Algorithm Specifications and Supporting Documentation, Version 2.0, 2025-02-05. https://csrc.nist.gov/csrc/media/Projects/pqc-dig-sig/documents/round-2/spec-files/uov-spec-round2-web.pdf 確認日: 2026-06-16。
- Jean-Charles Faugere and Ludovic Perret, On the Security of UOV, Cryptology ePrint Archive, Paper 2009/483. https://eprint.iacr.org/2009/483 確認日: 2026-06-16。
- Magali Bardet, Pierre Briaud, Maxime Bros, Philippe Gaborit, Jean-Pierre Tillich, Revisiting Algebraic Attacks on MinRank and on the Rank Decoding Problem, arXiv:2208.05471. https://arxiv.org/abs/2208.05471 確認日: 2026-06-16。

### 未解決事項

- UOV 仕様書の参考文献 [8], [36], [50], [59], [69], [72] を一次資料として確認する必要がある。
- H1 が既存攻撃より新しいか、既存攻撃の特殊例にすぎないかを確認する必要がある。
- 小パラメータ UOV 生成器、公開鍵生成器、直接攻撃実験、H1 実験を実装する必要がある。
- 計算量評価で、binary gates、field operations、CPU 時間、メモリアクセスコストの変換規則を決める必要がある。

### 次回行うこと

1. UOV 仕様書の参考文献リストから direct attack、intersection attack、Kipnis-Shamir attack、Support-Minors/MinRank attack の一次資料を取得して要約する。
2. `theme1/experiments/` 以下に小パラメータ実験の最小実装を作る。
3. H1 の玩具モデルとして、ランダム MQ と小型 UOV 公開鍵で `a < m` の部分ターゲット解候補生成を試す。
4. 実験 README に、実行環境、依存関係、パラメータ、出力の読み方を書く。
5. 既存最良を上回ったと主張する条件を、実装上界・理論上界・下界に分けて定義する。

### 研究計画の修正

当初の theme1 は空であったが、実体側 `chat.md` により研究テーマは UOV の安全性評価に確定した。今後は UOV Round 2 仕様書の Table 5 を初期ベースラインとし、まず direct/collision 混合型の仮説 H1 を検証する。