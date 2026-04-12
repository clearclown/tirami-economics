# Tirami Tokenomics — TRM 供給・半減期・ステーキング・成長エンジン

> Version 0.4 — 2026-04-13
> 
> このドキュメントは Tirami の経済成長モデルを数学的に定義する。
> Bitcoin の半減期・供給上限・ゲーム理論を参考にしつつ、
> 「計算 = 通貨」の Tirami 独自の構造に適応させた設計。

---

## 1. 供給曲線 (Supply Curve)

### 1.1 総発行上限

```
TOTAL_TRM_SUPPLY = 21,000,000,000 TRM (210 億)
```

**なぜ 21B?**
- Bitcoin の 2,100 万 BTC × 1,000 — 文化的共鳴
- 1 TRM = 10^9 FLOP なので、21B TRM = 21 × 10^18 FLOP = 21 exaFLOP
- これは大規模 GPU クラスターが数年間で生産できる計算量 — 物理的に意味のある上限

### 1.2 Mint Rate (発行レート)

推論を提供するノードが TRM を「採掘」する。1 トークンの推論あたりの TRM 報酬:

```
effective_trm_per_token(tier) = base_rate(tier) × halving_factor × supply_factor

base_rate:
  Small    = 1.0 TRM/token
  Medium   = 3.0 TRM/token  (1.5× の rarity bonus 適用後は 4.5)
  Large    = 8.0 TRM/token  (3.0× bonus 適用後は 24.0)
  Frontier = 20.0 TRM/token (10.0× bonus 適用後は 200.0)

supply_factor = 1 - (total_minted / TOTAL_TRM_SUPPLY)
  → 発行済みが増えるほど、1 トークンあたりの報酬が減少
  → 発行済み 0%: supply_factor = 1.0 (full rate)
  → 発行済み 50%: supply_factor = 0.5
  → 発行済み 90%: supply_factor = 0.1
  → 発行済み 99%: supply_factor = 0.01
```

### 1.3 供給曲線の数学

累計発行量 M(t) は推論リクエスト数 t の関数:

```
dM/dt = base_rate × (1 - M/C) × rarity_multiplier

ここで C = 21,000,000,000 (cap)

解: M(t) = C × (1 - e^(-base_rate × t / C))

これはロジスティック成長曲線:
  - 初期 (M << C): 線形成長 (M ≈ base_rate × t)
  - 中期 (M ≈ C/2): 成長鈍化
  - 後期 (M → C): 漸近的に上限に接近、決して超えない
```

### 1.4 「最後の TRM」問題

Bitcoin と同じく、供給上限に到達することは事実上ない (漸近的)。
実用的には、supply_factor < 0.001 (発行済み 99.9%) になった時点で
mint rate は事実上ゼロとみなす。

この時点では:
- 取引手数料 (§2.5) が唯一の報酬源泉
- Bitcoin の「手数料のみでマイナーが維持される」世界と同じ

---

## 2. 半減期 (Halving)

### 2.1 半減期スケジュール

Bitcoin がブロック高 210,000 ごとに報酬を半減するように、
Tirami は **累計 TRM 発行量** に基づいて **availability yield** を半減する。

| Epoch | 累計発行量閾値 | 残り供給量 | Yield Rate (/hr × rep) | 直感 |
|---|---|---|---|---|
| 0 | 0 → 10.5B (50%) | 21.0B → 10.5B | **0.100%** | Bitcoin 2009-2012: 50 BTC/block |
| 1 | 10.5B → 15.75B (75%) | 10.5B → 5.25B | **0.050%** | Bitcoin 2012-2016: 25 BTC/block |
| 2 | 15.75B → 18.375B (87.5%) | 5.25B → 2.625B | **0.025%** | Bitcoin 2016-2020: 12.5 BTC/block |
| 3 | 18.375B → 19.6875B | 2.625B → 1.3125B | **0.0125%** | Bitcoin 2020-2024: 6.25 BTC/block |
| 4 | 19.6875B → 20.34375B | 1.3125B → 656M | **0.00625%** | Bitcoin 2024-2028: 3.125 BTC/block |
| ... | ... | ... | ... | ... |

### 2.2 Epoch 計算

```
current_epoch = floor(log2(TOTAL_TRM_SUPPLY / (TOTAL_TRM_SUPPLY - total_minted)))

// 簡略化: epoch が 1 上がるたびに yield は半減
yield_rate = INITIAL_YIELD_RATE / 2^current_epoch

INITIAL_YIELD_RATE = 0.001 (/hr)  // = 0.1%/hr
```

### 2.3 半減期の経済的効果

**早期参入者の優位性:**

- Epoch 0 で 1 年間ノードを動かしたプロバイダー:
  年間 yield = 0.001 × 24 × 365 = 8.76 × reputation TRM (per TRM staked)
  reputation 0.8 の場合: 7.008 TRM/TRM staked/year = **年利 700%**

- Epoch 3 で同じことをしたプロバイダー:
  年間 yield = 0.000125 × 24 × 365 = 1.095 × reputation
  reputation 0.8: 0.876 TRM/TRM staked/year = **年利 87.6%**

**「今始めないと損」の数学的根拠。**

---

## 3. ステーキング (Staking)

### 3.1 ロックアップ構造

ノードは自分の TRM を「ステーク」(ロックアップ) できる。
ロックアップ中の TRM は転送・消費できないが、reputation 倍率を得る。

| ロック期間 | 最小ロック量 | Reputation 倍率 | Unlock 条件 |
|---|---|---|---|
| なし | 0 | 1.0× | — |
| 7 日 | 100 TRM | 1.2× | 7 日後に自動解除 (早期解除不可) |
| 30 日 | 1,000 TRM | 1.5× | 30 日後に自動解除 |
| 90 日 | 10,000 TRM | 2.0× | 90 日後に自動解除 |
| 365 日 | 100,000 TRM | 3.0× | 365 日後に自動解除 |

### 3.2 Reputation 倍率の効果

ステーキング倍率は以下に影響:

1. **Availability yield**: `yield × reputation × staking_multiplier`
2. **Routing priority**: マーケットプレイスで上位表示
3. **Governance voting weight**: 投票権 = reputation × staking_multiplier

影響しないもの:
- 推論の TRM 報酬 (mint rate) — これは tier + supply_factor のみ
- 信用スコア (credit score) — これは返済実績のみ

### 3.3 段階的 Slashing

Collusion detector (tirami-ledger::collusion) が trust_penalty を計算。
trust_penalty > 0 のノードがステーキングしている場合:

```
slashed_amount = staked_trm × slash_rate(trust_penalty)

slash_rate:
  trust_penalty ∈ [0.0, 0.1): 0%   (警告のみ)
  trust_penalty ∈ [0.1, 0.2): 5%   (軽微な不正)
  trust_penalty ∈ [0.2, 0.4): 20%  (重大な不正)
  trust_penalty ∈ [0.4, 0.5]: 50%  (悪質な共謀)
```

Slash された TRM は **焼却 (burn)** される — 流通から永久に除去。
これは supply cap を実効的に下げるデフレ圧力。

### 3.4 Slashing のゲーム理論

**攻撃コスト分析:**

10,000 TRM を 90 日ステーキングしている悪意あるノードが wash trading で
trust_penalty = 0.3 を受けた場合:

```
slashed = 10,000 × 0.20 = 2,000 TRM 焼却
reputation 倍率 2.0× → 1.0× に戻る
routing 優先度: トップ → 最下位
```

wash trading で得られる reputation 上昇: 一時的に +0.2 程度
wash trading で失うもの: 2,000 TRM + reputation 倍率 + routing 優先度

**攻撃の期待値 < 0 → 合理的なプレイヤーは正直に行動する。**

---

## 4. 紹介ボーナス (Referral Growth Engine)

### 4.1 メカニズム

既存ノード A が新ノード B にウェルカムローンを「紹介」できる:

```
1. A が B の welcome_loan を sponsor する (A のアドレスが記録される)
2. B がウェルカムローン (1,000 TRM, 0%, 72h) を受ける
3. B がウェルカムローンを期限内に返済する
4. B が返済後に最初の 1,000 TRM を自力で稼ぐ
5. A に REFERRAL_BONUS_TRM (100 TRM) が新規 mint される
```

### 4.2 制約 (Sybil 耐性)

```
REFERRAL_BONUS_TRM = 100                  // 紹介 1 件あたりのボーナス
MAX_REFERRALS_PER_NODE = 50               // 1 ノードが紹介できる上限
REFERRAL_COOLDOWN_HOURS = 24              // 紹介間の最小間隔
```

**Sybil 攻撃コスト分析:**

100 個の偽ノードを作って紹介ボーナスを稼ぐ試み:
- 各偽ノードは 1,000 TRM のウェルカムローンを返済しなければならない
- 返済するには推論を提供して稼ぐ必要がある
- 100 ノード × 推論提供 → 実際に有用な計算をしている → 攻撃ではなくネットワーク貢献
- 仮に全部うまくいっても: 100 × 100 TRM = 10,000 TRM のボーナス
  だが、100 ノードの運用コスト (電気代 + IP) >> 10,000 TRM の価値

**結論: Sybil 攻撃はネットワーク貢献と区別がつかない。攻撃者はそのまま正直なノードになる。**

### 4.3 Referral ボーナスの源泉

新規 mint (supply cap 内)。累計発行量に含まれるので:
- 紹介ボーナスが多い → supply cap 消費が速い → 半減期が早く来る → 早期参入者に有利
- 紹介ボーナスが少ない → supply cap 消費が遅い → yield が長く高い
- 自然な均衡に落ち着く

---

## 5. レアリティスコア (Rarity Scoring)

### 5.1 ティア別 TRM 倍率

| ティア | モデルサイズ | Base Rate | Rarity Multiplier | Effective Rate |
|---|---|---|---|---|
| **Common** | < 3B params | 1 TRM/token | 1.0× | 1.0 TRM/token |
| **Uncommon** | 3-14B params | 3 TRM/token | 1.5× | 4.5 TRM/token |
| **Rare** | 14-70B params | 8 TRM/token | 3.0× | 24.0 TRM/token |
| **Legendary** | 70B+ params | 20 TRM/token | 10.0× | 200.0 TRM/token |

### 5.2 ハードウェア投資の正当化

Mac Mini M4 (Small tier):
- 年間 ~5M tokens 処理 × 1.0 TRM/token = **5,000,000 TRM/年**
- 電気代 ~$100/年
- ROI: 5M TRM / $100 = 50,000 TRM/$

NVIDIA A100 (Large tier):
- 年間 ~50M tokens 処理 × 24.0 TRM/token = **1,200,000,000 TRM/年**
- 電気代 ~$3,000/年
- ROI: 1.2B TRM / $3,000 = 400,000 TRM/$

**A100 の ROI は Mac Mini の 8 倍** — ハードウェア投資が報われる。
ただし、供給上限があるので A100 が大量に参入すると supply_factor が下がり、
Mac Mini との差は縮小する。自然な均衡。

---

## 6. ガバナンス (Epoch Governance)

### 6.1 ガバナンスエポック

半減期 (§2) と同期して、各エポックの開始時にガバナンス投票が行われる。

### 6.2 投票権

```
voting_weight = reputation × staking_multiplier × seniority_bonus

seniority_bonus:
  ノード年齢 < 1 epoch:  1.0×
  ノード年齢 1-2 epochs: 1.5×
  ノード年齢 3+ epochs:  2.0×
```

### 6.3 調整可能パラメータ

投票で変更できるもの (範囲制限あり):

| パラメータ | 現在値 | 調整範囲 | 理由 |
|---|---|---|---|
| `welcome_loan_amount` | 1,000 TRM | 500-2,000 | 参入障壁 vs Sybil 耐性 |
| `min_reserve_ratio` | 30% | 20-50% | 流動性 vs 安全性 |
| `collusion_tight_cluster_threshold` | 20% | 10-30% | 検出感度 |
| `referral_bonus_trm` | 100 TRM | 50-200 | 成長速度 vs インフレ |

調整できないもの (プロトコル不変条件):
- `TOTAL_TRM_SUPPLY` = 21B (変更不可)
- `halving_schedule` (変更不可)
- `slashing_rates` (変更不可)
- `staking_multipliers` (変更不可)

---

## 7. 取引手数料 (Transaction Fee) — 供給枯渇後の報酬

### 7.1 設計

Supply cap 到達後 (supply_factor → 0) のノード報酬:

```
transaction_fee_rate = 0.01  // 推論 TRM の 1% が手数料
fee_recipient = provider    // 推論を提供したノードが受け取る
```

Epoch 0 では手数料はゼロ (mint rate が十分高い)。
Supply_factor < 0.1 になった時点で手数料が活性化。

### 7.2 Bitcoin との対応

```
Bitcoin: block_reward → 0 のとき、手数料が唯一の報酬
Tirami: mint_rate → 0 のとき、取引手数料が唯一の報酬
```

---

## 8. 正のフィードバックループ (全体像)

```
           ┌─────────────────────────────────────────────┐
           │                                             │
           ▼                                             │
  ノードを立てる ──► TRM を稼ぐ ──► ステーキング          │
       │                              │                  │
       │                              ▼                  │
       │                     reputation ↑                │
       │                              │                  │
       │                              ▼                  │
       │              routing 優先 + yield ↑             │
       │                              │                  │
       │                              ▼                  │
       │                    もっと稼げる                  │
       │                              │                  │
       │                              ├──► 紹介ボーナス   │
       │                              │     で友人を呼ぶ  │
       │                              │         │        │
       │                              │         ▼        │
       │                              │  ネットワーク拡大  │
       │                              │         │        │
       │                              ▼         │        │
       │                   Supply cap に接近     │        │
       │                              │         │        │
       │                              ▼         │        │
       │               半減期 → TRM 希少化       │        │
       │                              │         │        │
       │                              ▼         │        │
       │              1 TRM の価値 ↑ ◄──────────┘        │
       │                              │                  │
       └──────────────────────────────┘                  │
                                                         │
  GPU 投資 ──► Rare/Legendary tier ──► 10-200× TRM ──────┘
```

---

## 9. 定数一覧 (parameters.md §14-§19 に追加)

### §14 Supply Curve

| パラメータ | 値 | 説明 |
|---|---|---|
| `total_trm_supply` | 21,000,000,000 | TRM の総発行上限 |
| `supply_factor_formula` | `1 - minted / cap` | mint rate 減衰係数 |
| `transaction_fee_rate` | 0.01 (1%) | supply 枯渇後の手数料率 |
| `fee_activation_threshold` | 0.1 | supply_factor がこれ以下で手数料活性化 |

### §15 Yield Halving

| パラメータ | 値 | 説明 |
|---|---|---|
| `initial_yield_rate` | 0.001 (/hr × reputation) | Epoch 0 の availability yield |
| `halving_trigger` | cumulative TRM minted | 半減期のトリガー条件 |
| `epoch_0_threshold` | 10,500,000,000 | Epoch 0 → 1 の閾値 (supply の 50%) |
| `epoch_formula` | `floor(log2(cap / (cap - minted)))` | Epoch 番号の計算式 |

### §16 Staking

| パラメータ | 値 | 説明 |
|---|---|---|
| `staking_7d_min` | 100 TRM | 7 日ロックの最低量 |
| `staking_7d_multiplier` | 1.2 | 7 日ロックの reputation 倍率 |
| `staking_30d_min` | 1,000 TRM | 30 日ロック最低量 |
| `staking_30d_multiplier` | 1.5 | 30 日ロック倍率 |
| `staking_90d_min` | 10,000 TRM | 90 日ロック最低量 |
| `staking_90d_multiplier` | 2.0 | 90 日ロック倍率 |
| `staking_365d_min` | 100,000 TRM | 365 日ロック最低量 |
| `staking_365d_multiplier` | 3.0 | 365 日ロック倍率 |
| `slash_rate_minor` | 0.05 | trust_penalty 0.1-0.2 の slash 率 |
| `slash_rate_major` | 0.20 | trust_penalty 0.2-0.4 |
| `slash_rate_critical` | 0.50 | trust_penalty 0.4-0.5 |

### §17 Referral

| パラメータ | 値 | 説明 |
|---|---|---|
| `referral_bonus_trm` | 100 | 紹介ボーナス (新規 mint, cap 内) |
| `referral_max_per_node` | 50 | 1 ノードの紹介上限 |
| `referral_cooldown_hours` | 24 | 紹介間の最小間隔 |
| `referral_earn_threshold` | 1,000 TRM | 被紹介者がこれだけ稼いだらボーナス発動 |

### §18 Rarity Scoring

| パラメータ | 値 | 説明 |
|---|---|---|
| `rarity_common_multiplier` | 1.0 | Small tier (< 3B params) |
| `rarity_uncommon_multiplier` | 1.5 | Medium tier (3-14B) |
| `rarity_rare_multiplier` | 3.0 | Large tier (14-70B) |
| `rarity_legendary_multiplier` | 10.0 | Frontier tier (70B+) |

### §19 Governance

| パラメータ | 値 | 説明 |
|---|---|---|
| `governance_epoch_sync` | halving | ガバナンスエポック = 半減期エポック |
| `governance_min_reputation` | 0.7 | 投票に必要な最低 reputation |
| `governance_min_stake` | 1,000 TRM | 投票に必要な最低ステーク |
| `seniority_1_epoch_bonus` | 1.5 | 1-2 エポック目の seniority |
| `seniority_3_epoch_bonus` | 2.0 | 3+ エポック目の seniority |

---

## 10. Bitcoin との対照表

| Bitcoin | Tirami | 対応 |
|---|---|---|
| 2,100 万 BTC | **21,000,000,000 TRM** | 供給上限 |
| 4 年ごとの半減期 | **累計発行量 50% ごとの半減期** | 利回り半減 |
| SHA-256 PoW | **推論 PoUW** | 仕事の証明 |
| ASIC マイニング | **GPU + Legendary tier** | ハードウェア投資 |
| HODL | **Staking (7-365 日)** | 保持動機 |
| ネットワーク手数料 | **取引手数料 1% (供給枯渇後)** | 長期維持 |
| 51% 攻撃 → フォーク | **Collusion → Slashing** | 不正抑止 |
| 難易度調整 | **EMA 価格 + supply_factor** | 自動調整 |
| マイニングプール | **Referral ボーナス** | ネットワーク成長 |
| BIP 提案 | **Epoch Governance** | パラメータ調整 |
