# 17. Proof of Useful Work — zkML で「嘘をつけない」経済を作る

> ⚠️ **実装ステータス (2026-04-19 現在)**:
> 本章で論じる **zkML 検証は現状 production にロードされていません**。
> `tirami-zkml-bench` crate には `MockBackend` (shape テスト用、暗号的に
> 無効)しかなく、ezkl / risc0 の本番バックエンド統合は **Phase 20+ の
> 予定** で未実装です。`tirami-ledger::zk::policy_allows_trade()` も
> 現行 production コードパスから呼ばれていません。
>
> したがって本章は**将来の設計意図 (design intent)** を述べるもので
> あり、「現在動作している機能の説明」ではありません。Tirami の現行
> lazy-provider 対策は署名済み trade (dual signature) + audit
> challenge + collusion detector の 3 層のみで (§17.8 に整理)、zkML
> はその 4 層目として Phase 20+ で追加する計画です。

> Bitcoin の PoW は「無駄な計算」を燃やして信頼を買った。
> Tirami の PoUW は「役に立つ計算」から信頼を生もうとする。
> しかしそれを可能にするには、**計算結果が正しいことを暗号的に
> 証明できなければ**ならない。その暗号プリミティブが zkML である。

§16 で議論した stake-required mining は「違反すると損をする」
メカニズムだった。しかし「そもそも違反を検出できる」必要がある。
Tirami には Phase 17 以降で 3 つの違反検出機構がある:

1. **Collusion detector** — 統計的に怪しい取引パターンを検出。
2. **Audit challenge** — ランダム再実行で結果の再現性を確認。
3. **Proof of inference** (Phase 18.3 以降) — 計算結果が本物で
   あることを暗号的に証明する zkML。

本章は 3 つ目、**最も重要**な zkML について議論する。

---

## 17.1 Lazy Provider 攻撃

具体的な攻撃シナリオから入る。Tirami のプロバイダーは以下の
契約を結んでいる:

- Consumer が `chat completion` を要求 (プロンプト + max_tokens)。
- Provider は large model (e.g. Llama 70B) を実行し、トークンを
  返す。
- Provider は 70B モデル相当の compute を claim し、その分の
  TRM を受け取る。

ここで **Lazy Provider** が登場する:

```
悪質 Provider X:
  - 表向きは Llama 70B を serve していると advertise。
  - 実際には Llama 7B (10 倍軽い) で応答を生成。
  - ユーザーには区別が困難 (7B でも文法は正しい)。
  - 70B 相当の TRM を不当に受け取る。
```

Collusion detector と audit challenge はこの攻撃に対して:
- 完璧ではない (統計的検出は false negative を持つ)。
- 遅い (audit は時間的に遅れる)。
- コストがかかる (再実行は provider 側の compute を消費)。

**完全な解**は「計算結果が claim されたモデルで生成された」ことを
暗号的に証明できること。これが zkML の仕事。

---

## 17.2 zkML の基本発想

zkML (zero-knowledge machine learning) とは:

1. Provider がモデル weights (Llama 70B の特定コミット) を持つ。
2. Prompt P に対し output O を生成する際、「**モデル M で P を
   入力して O を出力した**」という文を zero-knowledge proof π に
   コンパイル。
3. Consumer は π を検証する。M の weights は知らなくてよい。
4. π が通れば「実際に M で計算された」ことが暗号的に保証。

これにより lazy provider は不可能になる:
- 7B で応答しても、「70B が使われた」という π は作れない。
- 70B 相当の π を作るには、70B を実際に実行する必要がある。
- 計算量は偽造できない。

---

## 17.3 現実の制約 — zkML は重い

しかし zkML には致命的な課題がある。**証明生成は推論の 10 × 〜
1000 × 遅い**。

| バックエンド | 1 M パラメータ モデルの prove 時間 | verify 時間 | 証明サイズ |
|---|---|---|---|
| ezkl (2025 時点) | 10–60 秒 | 100 ms | 数 KB |
| risc0 | 60 秒〜数分 | 数十 ms | 〜1 KB |
| halo2 カスタム | モデル固有のチューニング必須 | 数 ms | 〜500 B |

つまり、**すべてのトレードに zkML proof を要求するのは
非現実的**。70B モデルで毎回 1 分の proof 生成を待たせたら
ユーザー体験が壊滅する。

ここで Tirami は現実的な妥協として**確率的証明戦略**を採る。
Filecoin の PoST (Proof of Spacetime) と同じ発想:

- 全トレードではなく、ランダムに選ばれた一部 (e.g. 1%) に対して
  zkML proof を要求する。
- 「いつ要求されるかわからない」ので、provider は常に正直に
  計算せざるを得ない (違反を 1 度でも proof 失敗で検出されると
  slashed になる)。
- 全体のレイテンシ増加は平均 1% 分に抑えられる。

Phase 18.3 の `ProofPolicy` はこの戦略のための状態機械。

---

## 17.4 ProofPolicy の 4 状態

```rust
// crates/tirami-ledger/src/zk.rs
pub enum ProofPolicy {
    Disabled,      // 証明を一切参照しない (Phase 17 まで)
    Optional,      // 証明付きなら reputation ボーナス (Phase 19)
    Recommended,   // 証明無しは audit tier ペナルティ
    Required,      // 証明無しは ledger で拒否 (Constitutional)
}
```

各状態の経済的意味:

### Disabled (Phase 17 以前)
- 証明を作っても参照されない。
- lazy provider は collusion detector / audit のみで抑止。
- 「暗号的に 100% 正しい」という保証はない。

### Optional (Phase 19 以降のデフォルト)
- 証明付きのトレードは reputation bonus が入る。
- 証明なしでも trade は valid。
- Provider は任意で proof を添付。
- 初期段階: **zkML のインフラが整うまでの移行期間**。

### Recommended (Phase 20+ 目標)
- 証明なしトレードは audit tier を `Unverified` に固定。
- Unverified は 100% audit されるので、実質的に compute 2 倍の
  コスト (自分で計算 + 監査で他ノードが計算)。
- Provider は「proof 付きで tier を上げる」経済的合理性を持つ。

### Required (Phase 21+ 目標)
- 証明なしトレードは `execute_signed_trade` で拒否される。
- zkML proof が**必須**。
- ここに到達すると、lazy provider は暗号的に不可能。

---

## 17.5 単調ラチェット — 後退不可

`ProofPolicy` の遷移は Constitutional に制約されている:

```rust
pub fn try_ratchet_proof_policy(
    current: ProofPolicy,
    proposed: ProofPolicy,
) -> Result<ProofPolicy, ProofPolicyRatchetError> {
    if (proposed as u8) >= (current as u8) {
        Ok(proposed)
    } else {
        Err(ProofPolicyRatchetError::Downgrade)
    }
}
```

つまり governance で `Disabled → Optional → Recommended → Required`
へは進めるが、逆方向には進めない。

なぜこの制約が重要か:

- 「Required で 1 年運用したが不便なので Optional に戻そう」
  という提案が通ってしまうと、過去の「Required 期間に安心して
  取引していたユーザー」の信頼が裏切られる。
- Bitcoin で言えば「ある時期から `MAX_MONEY = 21 M` を 42 M に
  戻せる」と同じレベルの破壊。
- **一度 Required に到達したら永久に Required** — これが
  ユーザーに対する最も強い約束。

---

## 17.6 経済学的分析 — Information Asymmetry の解消

Michael Spence のシグナリング理論 (1973 Nobel) は、市場における
情報の非対称性を議論した:

- 雇用主は応募者の能力を事前に知らない。
- 応募者は学歴 (高コスト signal) を見せて能力を推定させる。
- このコストが能力と相関するなら、市場は効率化する。

Tirami の lazy provider 問題も本質は情報非対称性:

- Consumer はプロバイダの内部計算を知らない。
- Provider は自分が何をしたか知っている。
- 非対称のコストは consumer 側が払う (低品質応答を受け取る)。

シグナリング理論の教訓:
- 信号 (proof) のコストが偽造できない必要がある。
- 偽造できない信号は、発信者の type を確実に revealing する。

zkML proof は暗号的に偽造不可能 (そうであるよう設計されている)
ので、完璧な signal となる。

---

## 17.7 他の PoUW プロジェクトとの比較

Tirami の zkML 戦略は、業界でも珍しいアプローチ。比較:

| プロジェクト | 正しさ検証 | lazy 抑止 | zkML 使用 |
|---|---|---|---|
| **Bittensor** | なし (validator weight 消費攻撃に脆弱) | 弱 (reputation のみ) | なし |
| **io.net** | なし (オフチェーン trust) | 弱 | なし |
| **Gensyn** | Optimistic proof-of-learning | 中 (challenge-response) | 一部 |
| **Worldcoin** | zk-SNARK-of-humanity | ー | 独自 |
| **Filecoin** | PoSt + 確率的 challenge | 強 | なし (ストレージ特化) |
| **Tirami (目標)** | zkML 確率的 + collusion + audit | **強** | **計画中** |

Tirami の差別化:
- Bittensor / io.net の「trust だけ」モデルを、暗号的証明で
  アップグレード。
- Filecoin の確率的 challenge モデルを、**LLM inference** ドメインに
  転用。
- Gensyn の optimistic approach より強い保証 (fraud proof 不要)。

---

## 17.8 現実的な 2026 年時点

2026 年 4 月現在、zkML は以下の状況:

- **ezkl** (ONNX-native) は LLM inference で実用レベルに近づきつつ
  あるが、70B モデルはまだ重い。
- **risc0** は汎用 zkVM で、Rust コードを実行しながら proof を
  生成できる。が、ML モデルには最適化されていない。
- **halo2** はカスタム回路で最も効率的だが、モデル固有の設計が
  必要。

Tirami の `tirami-zkml-bench` crate は以下の状態:
- `MockBackend` (常時利用可、crypto 的には無効) — ハーネスを
  shape-test する用。
- `EzklBackend`, `Risc0Backend`, `Halo2Backend` — feature-gated
  scaffolds。実際のバックエンド統合は Phase 20+ で順次。

つまり、ProofPolicy = Optional はすでに機能するが、実際に proof を
添付できるのは MockBackend のみ (本番には使えない)。真の zkML
証明は Phase 20 の ezkl 統合を待つ。

---

## 17.9 マクロ経済的含意

ProofPolicy が Required に到達したときの経済効果を考察する:

1. **Provider の参入障壁が上がる**
   - zkML proof 生成には追加 GPU / 時間が要る。
   - 小規模 provider (MacBook Air) は Optional までで止まる可能性。
   - Required 相当を出せる provider は大規模インフラ持ちに偏る。

2. **価格分化**
   - Provider は「Required」「Optional」「No proof」のティア分化。
   - Recommended / Required 期間では「proof 付き」トレードが
     プレミアム価格を付ける (audit tier 上昇による reputation gain)。

3. **Sybil 攻撃の困難化**
   - 1,000 個の fake provider で安い proof-less serve を提供しても
     `Required` では ledger で rejected。
   - Sybil が proof を作るには実計算を要する — 結果として計算能力
     に応じた公正な市場に近づく。

4. **Credible Neutrality の強化**
   - Required 状態では「どの provider が何を serve したか」を
     第三者が暗号的に検証可能。
   - 開発者ですら「proof のない trade を rescue する裏口」を
     作れない (Constitutional ratchet)。

これが Tirami が長期的に目指す姿。2026 年の Optional 状態は、
その目標に向かう第一歩。

---

## 次章へ

§18 では、本章の逆説的な話をする。**TRM は金融商品ではない**、
と公式にマニフェスト化する立場について。「でも第三者が勝手に
市場を作ったら?」「secondary market のリスクは誰が負うのか?」と
いう問いに対する Tirami の立場を議論する。

参照:
- 実装: `crates/tirami-ledger/src/zk.rs`
- ベンチ crate: `crates/tirami-zkml-bench`
- 設計: `tirami/docs/zkml-strategy.md`
- パラメータ: `spec/parameters.md § 24`
