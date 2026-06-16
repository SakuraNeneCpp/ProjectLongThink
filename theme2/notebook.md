# theme2 研究ノート

## 初期設定

### 研究テーマ

一般化 UOV トラップドアの有用性を探索し、署名に限定せず、ハッシュ、鍵交換、ゼロ知識証明、匿名クレデンシャル等への応用可能性を評価し、実装可能な暗号方式候補に落とし込む。

### 背景

- 事実: Human は `idea.pptx` にある一般化 UOV トラップドアについて、有用性が現状低いと認識しており、本研究ではその有用性を探り、効率・安全性の両面から評価する実用的暗号を考案・実装するよう指示している。
- 未検証事項: この実行環境では `idea.pptx` のテキスト抽出・スライド解析に失敗したため、今日の整理は `chat.md` の指示と公開資料に基づく。`idea.pptx` 内の具体的な「一般化」の定義は次回以降に確認する必要がある。

### 主要な研究問い

1. 一般化 UOV トラップドアは、通常の UOV/MAYO 型 hash-and-sign 以外に、どの暗号プリミティブで本質的に役立つか。
2. トラップドアが提供する「MQ 写像の preimage sampling」を、chameleon hash、sanitizable signature、ZK proof、匿名クレデンシャル、鍵交換などへ安全に接続できるか。
3. 公開鍵サイズ、署名・証明サイズ、計算量、既知攻撃に対する余裕を考えると、どの応用が実装対象として最も現実的か。

### 明らかにしたい範囲

- UOV 系公開写像の構造的性質と、既知攻撃面の整理。
- 署名以外の応用候補の失敗理由を含む比較。
- 最小実装のための有限体、パラメータ、API、テスト項目の設計。
- 研究成果として主張できる新規性・正当性・有用性の範囲。

### 想定される成果物

- 一般化 UOV トラップドアの仕様メモ。
- toy 実装または研究用 prototype。
- 候補応用の安全性・効率評価表。
- 金曜日の中間報告スライド。

### 評価基準

- 新規性: 単なる UOV 署名の再実装ではなく、トラップドアの別用途または機能拡張に実質的な差分があるか。
- 正当性: MQ 解法、MinRank 型攻撃、微分・双線形化による衝突、ランダムオラクル依存などを明示しているか。
- 有用性: 既存 PQC と比べて鍵サイズ・証明サイズ・計算量・機能性のいずれかに明確な利点があるか。
- 再現性: 実装、パラメータ、測定手順が追跡可能か。
- 限界: 未証明仮定、未検証の PPTX 内容、toy 実装と実用実装の差を隠さないか。

### 初日の作業計画

- 最新 Human 指示を確認する。
- UOV/MAYO と NIST 追加署名プロセスの位置づけを確認する。
- 署名以外の応用候補を列挙し、最初の有望候補と危険候補を分ける。
- 次回実装に向けた最小候補を定める。

## 2026-06-16

### 今日の目的

一般化 UOV トラップドアの研究を開始し、既知の UOV 系事実と、署名以外の応用候補を整理する。特に、chameleon hash / sanitizable signature、鍵交換、ゼロ知識証明・匿名クレデンシャルへの接続可能性を、事実・仮説・推論・未検証事項に分けて評価する。

### 実施した作業

- `theme2/chat.md` を確認し、Human の最新指示を研究目標として採用した。
- `theme2/notebook.md` が空であることを確認し、初期設定を作成した。
- `idea.pptx` の内容確認を試みたが、現在の実行環境では PPTX 内テキストの抽出に必要なローカル実行手段が使えず、具体式は未確認として扱った。
- NIST PQC 追加署名プロセス、UOV、MAYO、MQOM、OV 系鍵交換提案、post-quantum chameleon hash / sanitizable signature の公開情報を確認した。
- 応用候補を比較し、素朴な chameleon hash 案の問題点と、ZK/匿名クレデンシャル方向の有望性を整理した。

### 得られた知見

#### 事実

- UOV は、トラップドア付き多変数二次写像に基づく hash-and-sign 型のデジタル署名である。UOV 公式サイトは、UOV が 1999 年の Kipnis-Patarin-Goubin に由来し、NIST の追加 PQC デジタル署名プロセスに提出されたと説明している。
- NIST は 2026-05-14 に追加デジタル署名の Round 3 候補を発表し、9 候補に FAEST, HAWK, MAYO, MQOM, QR-UOV, SDitH, SNOVA, SQIsign, UOV を含めている。多変量系では MAYO, QR-UOV, SNOVA, UOV が残っている。
- NIST の主要 PQC 標準は 2024 年に FIPS 203/204/205 として公開された ML-KEM, ML-DSA, SLH-DSA であり、追加署名は ML-DSA/SLH-DSA のバックアップや独自ユースケースを狙う長期評価である。
- MAYO Round 2 specification は、Oil and Vinegar の公開鍵を、秘密部分空間 `O` 上で 0 になる MQ 写像 `P: F_q^n -> F_q^m` として説明している。通常の署名ではランダムな `v` を選び、`o in O` について `P(v+o)=t` を線形方程式として解く。
- MAYO は `dim(O)=o<m` の小さい oil space と `k`-fold の whipped map `P*` を使い、`ko>m` にして署名生成時の線形解法を回復する。仕様書の例では NIST level 1 の一組のパラメータ `(n,m,o,k,q)=(81,64,17,4,16)` で public key 4912 bytes, signature 186 bytes と説明されている。
- UOV 公式サイトは、classic UOV level 1 の public key が 272 KB と大きい一方、圧縮変種では 43 KB まで下げる設計を示している。ただしこの数値は同サイトの Round 2 説明に基づく。
- MQOM は unstructured MQ 問題に基づく MPC-in-the-head 型署名であり、公式サイトは小さい public key/secret key と、2.8 KB から 4.1 KB 程度の level 1 signature range を主張している。これは UOV 系 preimage relation を ZK/MPC-in-the-head で扱う際の比較対象になる。
- 2024 年の arXiv 論文 OliVier は、OV 多項式と fully quadratic equations を混合した OV 系 public key exchange cryptosystem を提案している。ただし NIST 標準候補ではなく、実用安全性は本研究内で未検証である。
- 2026 年の arXiv 論文は、McEliece trapdoor を chameleon hash に使う post-quantum sanitizable signature を提案している。これは UOV ではないが、「trapdoor による controlled collision finding」が post-quantum な sanitizable signature の実用動機になることを示す関連事例である。

#### 仮説

- 一般化 UOV トラップドアの最も自然な機能は「公開 MQ 写像の preimage sampler」であり、KEM/鍵交換よりも、署名、credential issuance、ZK witness generation、controlled rewriting の方が接続しやすい。
- 単純な KEM には向きにくい。理由は、UOV 型写像が一般に多対一であり、復号者がトラップドアで得る preimage が送信者の選んだ preimage と一致するとは限らないため、`K=H(preimage)` 型の shared secret を安定に復元できないからである。
- 素朴な chameleon hash `CH_pk(M,r)=P(r)+G(M)` は一見有望だが、randomizer domain を unrestricted にすると、公開二次写像の polar form/differential を使った衝突探索が成立する可能性が高い。したがって、そのままでは collision-resistant chameleon hash と主張できない。
- むしろ、第一の実装候補は「UOV/GUOV 署名を匿名 token / credential とみなし、保持者が `P(s)=H(attributes)` の witness `s` を知ることを ZK で示す方式」がよい可能性がある。この方向では、トラップドアは issuer の credential 発行に使われ、ZK は holder privacy と selective disclosure に使われる。

#### 推論

- 任意の二次写像 `P` について、差分 `P(x+d)-P(x)` は `x` に関して線形になる。`n>m` の UOV 型写像では、攻撃者が差分 `d` を選び、線形方程式を解いて `P(x+d)-P(x)=Delta` を満たす `x` を探す余地が大きい。このため、`P` をそのまま chameleon hash の randomizer map に使う設計は、少なくとも強い衝突耐性の候補として危険である。
- chameleon hash 方向を続けるなら、randomizer を小さい公開ドメイン・固定重み集合・検証可能な制約付き集合に制限する、または random oracle preprocessing と追加 commitment を組み合わせる必要がある。ただし、制約付き randomizer に対してトラップドアが collision を生成できるかは未解決である。
- ZK/匿名クレデンシャル方向では、既存の UOV/MAYO 署名の「短い witness」と MQOM 系の「MQ relation を証明する技術」が接続できる。新規性は、一般化 UOV トラップドアで発行した witness を、署名検証ではなく privacy-preserving proof の対象にする点に置ける可能性がある。
- 公開鍵サイズは大きな制約である。UOV classic のままでは credential verifier に重いが、MAYO 型の seed-expanded public key や小さい oil space の発想を取り込めば、prototype の現実性は上がる。

#### 未検証事項

- `idea.pptx` の「一般化 UOV トラップドア」が、上記の hidden zero subspace / preimage sampler 型理解と一致するか。
- 素朴 chameleon hash への differential collision 推論が、対象の一般化トラップドアにもそのまま適用できるか。
- ZK/匿名クレデンシャル案で、MQ relation の証明サイズが実用範囲に入るか。
- MAYO 型 whipped map を credential/ZK relation に使う場合、証明 relation が増大しすぎないか。
- 既知の MinRank、rectangular MinRank、Support Minors Modeling、Groebner basis 系攻撃に対する安全余裕。

### 根拠・出典

- NIST, "Post-Quantum Cryptography", CSRC, 確認日 2026-06-16. https://csrc.nist.gov/Projects/post-quantum-cryptography
  - 2024 年の FIPS 203/204/205 公開、追加標準化プロセスの位置づけを確認。
- NIST, "Post-Quantum Cryptography: Additional Digital Signature Schemes - Round 3 Additional Signatures", CSRC, 確認日 2026-06-16. https://csrc.nist.gov/Projects/pqc-dig-sig/round-3-additional-signatures
  - 2026-05-14 発表の Round 3 候補 9 件と、多変量系候補を確認。
- NIST IR 8610, "Status Report on the Second Round of the Additional Digital Signature Schemes for the NIST Post-Quantum Cryptography Standardization Process", 2026-05, 確認日 2026-06-16. https://csrc.nist.gov/pubs/ir/8610/final
  - Round 3 選定理由と選定候補を確認。
- UOV team, "UOV - Unbalanced Oil and Vinegar", 確認日 2026-06-16. https://www.uovsig.org/
  - UOV の概要、NIST 追加署名提出、Round 2 時点の key/signature size 説明を確認。
- MAYO team, "MAYO Round 2 Specification", 確認日 2026-06-16. https://pqmayo.org/assets/specs/mayo-round2.pdf
  - Oil and Vinegar の trapdoored MQ map、oil space、whipped map、例示パラメータを確認。
- MQOM team, "MQ on my Mind (MQOM)", 確認日 2026-06-16. https://mqom.org/
  - unstructured MQ + MPC-in-the-head 型署名、key/signature size の比較材料を確認。
- Antonio Corbo Esposito, Rosa Fera, Francesco Romeo, "OliVier: an Oil and Vinegar based cryptosystem", arXiv:2405.08375, 2024-05-14, 確認日 2026-06-16. https://arxiv.org/abs/2405.08375
  - OV 系鍵交換提案が存在すること、ただし標準候補ではなく未検証であることを確認。
- Shahzad Ahmad, Stefan Rass, Zahra Seyedi, "Post-Quantum Sanitizable Signatures from McEliece-Based Chameleon Hashing", arXiv:2602.20657, 2026-02-24, 確認日 2026-06-16. https://arxiv.org/abs/2602.20657
  - post-quantum chameleon hash / sanitizable signature の応用動機と、trapdoor collision finding の関連事例を確認。

### 未解決事項

- `idea.pptx` の具体的トラップドア定義を読めていない。
- chameleon hash に使える制約付き randomizer domain を設計できるか不明。
- ZK/匿名 credential 方向の証明サイズと verifier cost が未測定。
- 既存 UOV/MAYO/SNOVA/QR-UOV と比べた新規性の置き所が未確定。
- 実装対象を GF(16) にするか GF(2) toy から始めるか未決定。

### 次回行うこと

1. `idea.pptx` の内容を抽出し、一般化 UOV トラップドアの定義・記号・想定パラメータを研究ノートに写す。
2. 最小 prototype の仕様を決める。候補は GF(16) または GF(2) 上の UOV/GUOV preimage sampler、public evaluation、sign/preimage-sample のテスト。
3. ZK/匿名 credential 案を `IssueCredential`, `ProveKnowledge`, `VerifyProof` の形で抽象 API 化し、MQOM/MPC-in-the-head 型 proof と通常 UOV verification のコストを比較する。
4. chameleon hash 案について、unrestricted randomizer への differential collision を小さい toy instance で実験し、危険性を再現する。
5. 金曜日までに、候補応用の比較表と、実装優先順位を中間報告に入れられる形へ整理する。

### 研究計画の修正

- 初期方針として、署名そのものの改良よりも、UOV/GUOV preimage を witness として使う ZK/匿名 credential 方向を第一候補にする。
- chameleon hash / sanitizable signature 方向は有用性が高いが、素朴構成に衝突耐性上の重大懸念があるため、制約付き randomizer domain を設計できるまで第二候補とする。
- 鍵交換/KEM 方向は、OliVier のような別設計は存在するものの、一般化 UOV トラップドア単体からは shared secret 復元の問題が大きいため、現時点では低優先度とする。
