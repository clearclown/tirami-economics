# 16. Stake-Required Mining — 経済的皮膚を入れる

> ⚠️ **実装ステータス (2026-04-19 現在)**:
> `ComputeLedger::can_provide_inference()` のチェック関数は
> 実装済みですが、**HTTP `/v1/chat/completions` および P2P
> pipeline の trade 実行パスから呼ばれていません** (tests でのみ
> 呼ばれる状態)。つまり本章が述べる「stake がないと TRM を稼げない」
> ルールは **現行 production 運用では enforcement されていません**。
> 実 gate 配線は Phase 20 で予定しています。
>
> 現行プロトコルでは以下が有効です:
> - Slashing loop (Phase 17 Wave 1.3 — `spawn_slashing_loop`) は
>   動作。
> - `StakingPool` に stake することで yield bonus を得られる (§13)。
> - Welcome loan sunset エポック 2 (Phase 18.2、Constitutional) は
>   `can_issue_welcome_loan` で enforcement 済。
>
> したがって本章の本文は **将来の設計意図** を述べるものです。

> 「何も失うものがない人」を経済ルールで縛るのは不可能だ。
> Tirami の以前のバージョンは、この失敗を犯していた。Phase 18.2
> で修正された。

§15 で触れた憲法的不変量には `MIN_PROVIDER_STAKE_CONSTITUTIONAL_FLOOR = 10 TRM`
が入っている。これは Phase 18.2 で導入された「stake がないと TRM を
稼げない」というルールの床である。本章では、**なぜ stake を要求
するのか**、**なぜ完全に free だった Phase 17 以前はダメだったのか**、
そして**従来の経済学の「skin in the game」概念との対応**を議論する。

---

## 16.1 Phase 17 以前の穴

Phase 17 で Tirami にはすでに以下が揃っていた:
- Slashing 機構 (`apply_slash`) — 違反時に stake を焼く。
- 協調 detector — tight cluster、volume spike、round-robin 検出。
- 監査システム — 疑わしい trade をランダム再実行。

しかし一つ**致命的な穴**があった。これらのペナルティの対象が
全員 `StakingPool` に stake を積んでいることを暗黙に仮定していた
のに、実際には「TRM を稼ぐのに stake は必要ない」構成だった。

つまり:

```
プロバイダA: stake 0 TRM で compute を提供し、TRM を稼ぎ続ける。
          監査で違反が見つかっても、焼ける stake がないので
          apply_slash は 0 TRM を焼く。実質ノーペナルティ。
```

違反抑止メカニズムは**施行可能でなければ意味がない**。Bitcoin
miner が 51% 攻撃したら、彼らの投資したハードウェア + 電力が
ネットワーク分裂によって**回収不能**になるから攻撃コストが高い。
このコストが抑止を生む。

Tirami Phase 17 以前は、この「コスト」が 0 だった。

---

## 16.2 従来経済学の Skin in the Game

Nassim Taleb の『Skin in the Game』 (2018) は、意思決定者が自身の
選択のリスクを引き受けることの重要性を説いた:

- 外科医は自分の患者を治療する技術を身につけるのに、自分が
  切られるリスクは引き受けない — それでも良い外科医になれるのは、
  評判 (reputation) という長期的 skin がかかっているから。
- 投資アドバイザーは他人の金を運用するので、損失の直接的 skin は
  持たない — だから監督規制 (SEC / 金融庁等) が必要。
- 政治家は選挙で落ちるリスクはあるが、政策の長期的コストは引き受け
  ない — だから憲法 + 司法による抑止が必要。

Tirami のプロバイダも同じ構造だ。「違反すると何が失われるか」を
明確にしないと、rational actor は**違反する方が得**になる。

---

## 16.3 Phase 18.2 の解法

Phase 18.2 は stake を必須にした。ただし、完全な「ゼロから stake」
を要求するとネットワークの初期参加障壁が高すぎる。Filecoin の
bootstrap grant と同じ発想で、**段階的スロープ**を採用する:

```
新規参加: stake 0 TRM で参加可。ただし累計 earn が
          STAKELESS_EARN_CAP_TRM = 10 TRM に達するまで。

10 TRM 稼いだ後: MIN_PROVIDER_STAKE_TRM = 100 TRM を stake しないと
               追加の TRM を稼げない。
```

これを `can_provide_inference` で runtime チェック:

```rust
pub fn can_provide_inference(&self, node_id: &NodeId, pool: &StakingPool, now_ms: u64) -> bool {
    // プライマリパス: active stake が閾値以上
    if pool.active_stake(node_id, now_ms) >= MIN_PROVIDER_STAKE_TRM {
        return true;
    }
    // フォールバック: bootstrap faucet 内
    let earned = self.balances.get(node_id).map(|b| b.contributed).unwrap_or(0);
    let never_slashed = self.slash_events.iter().all(|e| e.node_id != *node_id);
    earned < STAKELESS_EARN_CAP_TRM && never_slashed
}
```

観察すべき重要な点:

1. **slash された provider は faucet 経路を失う**。つまり「10 TRM
   までは stakeless で、11 TRM 目から stake 必要」— これを抜ける
   ために違反して slashed すると、**stakeless にも戻れない**。
   Rehabilitation には stake を積むしかない。

2. **stake 閾値 (100 TRM) は可変だが、憲法床 (10 TRM) がある**。
   governance で 50 TRM には下げられるが、5 TRM にはできない。

3. **faucet 上限 (10 TRM) にも憲法天井 (100 TRM) がある**。
   「faucet を 1,000,000 TRM に拡大して実質ゼロコスト化する」
   攻撃を防ぐ。

---

## 16.4 経済学的分析 — なぜこの閾値か

Phase 18.2 の数値 (`100 TRM` stake / `10 TRM` faucet) はどこから
来たか。これは議論の余地がある設計判断で、以下の考慮がある:

**100 TRM stake の根拠**:
- 1 TRM = 10⁹ FLOP の定義から、100 TRM = 10¹¹ FLOP ≈ 1 分の
  Apple M2 GPU 稼働 = 実運用の 1 日の仕事量レベル。
- 違反時のペナルティ上限 (slash rate major = 30%) で最大 30 TRM
  を焼くことができ、これは welcome loan (1,000 TRM) の 3% に
  相当する。違反者が再参加を試みるときに感じる摩擦として十分。
- ただし、投機目的の大量 stake を阻む「小規模プロバイダの参加
  障壁」にしないため、意図的に低めに設定。

**10 TRM faucet の根拠**:
- 新規ノードが「とりあえず試す」ためのゼロコスト枠。
- 10 TRM = 10¹⁰ FLOP ≈ 数十秒の compute 相当。
- 悪用を制限するために憲法天井 100 TRM を設定。

**Sybil 耐性の計算**:
- 1,000 個の Sybil ノードを立てて faucet を食い潰そうとすると、
  1,000 × 10 = 10,000 TRM の stakeless earn が可能。
- これは Phase 17 Wave 4 の per-ASN rate limit + Phase 18.2 の
  slash_events グローバルチェックで相殺される:
  - Sybil が違反を起こせば slashed、以降 stakeless 不可。
  - 違反を起こさないなら... 合法的な compute を提供しているので、
    ネットワーク全体には**純益**。

この組み合わせが「低い参加障壁 + 強い違反抑止」のスイートスポット
を作る。

---

## 16.5 従来のマイニングとの違い

Bitcoin mining との違いを対比する:

| | Bitcoin | Tirami Phase 18.2 |
|---|---|---|
| 初期投資 | ASIC ハードウェア (数千ドル〜) | TRM stake (100 TRM、または faucet 10 TRM) |
| 初期投資の回収性 | ハードは売却可能 | stake は unlock 時に戻る (burn 除く) |
| 違反コスト | ネットワーク分裂 / ハード無価値化 | slashing (stake を焼く) |
| 参加障壁 | 高 (ハードウェア購入必須) | 低 (10 TRM faucet) |
| 違反検出の速さ | ブロック検証 (10 分) | 監査 + collusion detector (リアルタイム) |

Tirami の方が参加障壁は低い (faucet のおかげ)、違反検出は速い
(audit + collusion)、違反コストは比率的に重い (100 TRM stake は
プロバイダの「資本」の相当部分)。

---

## 16.6 Welcome Loan のサンセット

Phase 18.2 の設計には伏せられた配慮がある。既存の
`welcome_loan_amount = 1,000 TRM` は「新規ノードにブートストラップの
資本を与える」機構で、これは stakeless faucet と機能が重なる。

そこで `WELCOME_LOAN_SUNSET_EPOCH = 2` (憲法的) を導入した。
エポック 2 (≥ 87.5% 供給発行時点) に達すると welcome loan は永久停止。

エポック 2 時点のネットワークは:
- すでに十分成熟 (87.5% 供給済み)。
- 新規参加者は stake を積むか、10 TRM faucet で試すかの二択。
- welcome loan 抜きでも参加経路は残る。

この設計は Filecoin の「block reward が時間経過で減る」と同じ
思想: 初期の撒餌は**卒業**する。

---

## 16.7 実装の検証

第三者監査人にとって、stake-required mining が正しく施行されて
いるかを検証する手段:

```bash
# 1. 憲法床と閾値の確認
grep -nE "MIN_PROVIDER_STAKE_|STAKELESS_EARN_CAP_|WELCOME_LOAN_SUNSET_" \
  crates/tirami-ledger/src/lending.rs

# 2. can_provide_inference のユニットテスト (+6 件)
cargo test -p tirami-ledger --lib can_provide_inference

# 3. can_provide_inference 実行パス
grep -rnE "can_provide_inference\(" crates/tirami-node/src/
```

重要なのは、このチェックが**HTTP handler と P2P pipeline の両方**で
呼ばれていること。片方だけだと「低レイテンシな HTTP 経路から
stake を回避」という抜け穴が作れる。

---

## 16.8 参加可能性とのバランス

stake を要求すると、当然「計算機はあるが TRM がない貧しいユーザー」
の参加が難しくなる。Tirami はこれを 3 つの経路で緩和:

1. **Faucet 10 TRM** — 完全無料の試走枠。
2. **Welcome Loan 1,000 TRM** — 0% 金利で 72 時間借りる (sunset 前のみ)。
3. **Referral bonus** — 既存ユーザーが紹介すると両者に bonus。

つまり「計算機を持っていれば、お金がなくても TRM は稼げる」
経路が保存されている。これは Tirami の根本思想「compute is money」
を維持するための設計。

---

## 16.9 経済モデルへの含意

マクロで見ると、stake-required mining は TRM 供給動学に以下の
影響を与える:

- **流通速度の低下** — 一部の TRM は常に stake に「拘束」される。
  流通可能な TRM は `total_supply × (1 - stake_ratio)` に減る。
- **需給の弾力性上昇** — stake された TRM を unlock する gate が
  あるため、急激な売り圧力は抑制される。
- **物価安定** — 結果として TRM ↔ compute の交換レートは安定化
  (stake が流通量バッファとして機能)。

これは現代金融で言う「Monetary base vs M2」の関係に似ている。
Tirami の「monetary base」は total supply、流通している TRM (M2
相当) はそのうち stake されていない部分。

---

## 次章へ

§17 では、stake と並ぶもう一つの違反抑止軸「Proof of Useful
Work」——つまり **zkML (zero-knowledge machine-learning proof)**
を扱う。「計算結果が正しいことを暗号的に証明する」ためのゲート
`ProofPolicy` を、Optional → Recommended → Required の単調ラチェット
でどう運用するかを議論する。

参照:
- 実装: `crates/tirami-ledger/src/lending.rs::{MIN_PROVIDER_STAKE_TRM, STAKELESS_EARN_CAP_TRM}`
- 設計: `tirami/docs/constitution.md § Article XI`
- パラメータ: `spec/parameters.md § 22, § 23`
