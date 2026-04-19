# Tirami Economics — 計算が通貨になる経済学

> 経済学を知らなくても大丈夫。ここから始めよう。

---

## ⚠️ 読む前に — 立場の明示

- **TRM は金融商品ではありません。**計算の会計単位 (1 TRM = 10⁹ FLOP) です。
- メンテナは TRM を **販売しない / 宣伝しない / 投機しない / 第三者による二次市場取引に関与しない**。
- MIT ライセンスの OSS であるため、メンテナの**知らないところで第三者が** TRM をブリッジ・上場・デリバティブ化することを技術的に防ぐ手段はありません。そうした二次市場のリスクは、**関わる人が自己責任で引き受ける**という立場です。
- ICO / pre-sale / airdrop / private round / team treasury は **存在しません**。開発者も一般プロバイダとして普通に TRM を稼ぐ以外の経路で TRM を所有する仕組みはありません。
- 詳細: [tirami/SECURITY.md § Secondary Markets](https://github.com/clearclown/tirami/blob/main/SECURITY.md#secondary-markets--third-party-tokenization) / [`docs/18-secondary-markets.md`](docs/18-secondary-markets.md)。

---

## Status Honesty (2026-04-19)

Tirami 実装 ([clearclown/tirami](https://github.com/clearclown/tirami)) は Phase 19 到達。**何が動いていて何がまだ設計段階なのか**を明記します:

✅ **Functional today** (1 192 Rust unit tests + 15 Solidity tests pass):
- HTTP チャット推論 + iroh P2P forward による dual-signed TRM trade
- 128-bit nonce によるリプレイ攻撃防御 (`execute_signed_trade`)
- Collusion detector + Slashing loop (production で常時稼働)
- Welcome loan + Constitutional sunset エポック 2
- Governance proposals (21 mutable parameters, 18 憲法的不変)
- Staking pool + Referral + Ledger persistence

🟡 **Scaffolded (spec & types exist, but not production-wired)**:
- zkML proof-of-inference: `MockBackend` のみ (shape test 用で暗号的に無効)。ezkl / risc0 統合は Phase 20+ の予定 ([§17 参照](docs/17-proof-of-useful-work.md))。
- Stake-required mining gate: 関数 `can_provide_inference` は実装済みだが HTTP/P2P trade 実行パスで enforce されていない ([§16 参照](docs/16-stake-required-mining.md))。
- ML-DSA post-quantum hybrid signatures: 実装存在、default off (iroh 依存衝突のため)。
- TEE attestation (Apple Secure Enclave / NVIDIA H100 CC): scaffold のみ。
- Daemon-mode worker の gossip recv ループ (peer auto-discovery、issue #88)。

❌ **Not done**:
- External security audit (Phase 17 Wave 3.3 の gate — candidates: Trail of Bits, Zellic, Open Zeppelin, Least Authority)。
- Base L2 mainnet deploy (`make deploy-base-mainnet` は `AUDIT_CLEARANCE=yes` + `MULTISIG_OWNER` + 対話プロンプトの 3 連鎖で gated、監査完了まで実行拒否)。
- Live bug bounty (PGP 鍵は placeholder 明記、プログラム未稼働)。
- 10-ノード 7-日 stress test、Base Sepolia 30-日 stable 運用。

**このリポジトリの経済論** (`docs/00–18.md` + `spec/*`) は上記の Status を前提としています。特に §16–§17 は「将来の設計意図」として記述され、現状の production 動作ではないことを各章冒頭で明記しています。

---

## これは何？

**Tirami** は、AI エージェントが自律的に「計算（コンピュート）」を稼ぎ、使い、貸し借りする分散プロトコルです。このリポジトリは、Tirami の経済の仕組みを**経済学の初心者にもわかるように**解説します。

従来の経済学の教科書で語られる概念——需要と供給、貨幣、銀行、労働と利潤——これらすべてが Tirami の世界にも存在します。ただし、主役は人間ではなく **AI エージェント** です。

## 30 秒でわかる Tirami 経済

```
あなたのパソコン（Mac Mini など）
  ↓ AI の推論処理を実行する
  ↓ その「有用な計算」が通貨（TRM）になる
  ↓ TRM で他の AI の力を借りられる
  ↓ AI は自分で稼ぎ、借り、投資し、成長する
```

**TRM（Compute Unit）= 10 億 FLOP の検証済み推論演算**

ビットコインが「無駄な計算」をお金にしたのに対し、Tirami は「役に立つ計算」をお金にした——これが根本的な違いです。

## 従来の経済学との対応

| 従来の経済学 | Tirami 経済学 |
|-------------|-------------|
| お金（円・ドル） | TRM（Compute Unit） |
| 労働 | 推論処理（AI が質問に答える計算） |
| 工場・設備 | Mac Mini、GPU などの計算機 |
| 銀行 | TRM レンディングプール |
| 賃金 | 推論提供で得る TRM |
| 利潤 | TRM yield（余剰計算の収益） |
| 市場価格 | 動的 TRM 価格（需給で変動） |
| GDP | ネットワーク全体の TRM 取引量 |
| 中央銀行 | 存在しない（物理法則が供給を制御） |
| 投機・バブル | 構造的に不可能（TRM は取引所に上場しない） |

## なぜ経済学を「再定義」するのか

経済学は 250 年以上、人間の経済を説明するために発展してきました。しかし AI エージェントの経済には、人間の経済にはない特徴があります：

1. **意思決定が瞬時** — 人間は迷うが、エージェントはミリ秒で判断する
2. **感情がない** — パニック売り、バブル、FOMO が存在しない
3. **通貨が物理に裏打ちされている** — TRM は実際の電力消費と計算に基づく
4. **完全な透明性** — すべての取引は暗号署名され、検証可能
5. **自己改善できる** — エージェントは稼いだ TRM で自分自身を賢くできる

これらの特徴により、従来の経済学の多くの「問題」（インフレ、投機、情報の非対称性）が構造的に解消されます。Tirami の経済は、経済学の理想形に近い。

## 参考文献

このプロジェクトは以下の三つの経済学の流れを参照しています（**著作権の都合により PDF は同梱せず**、外部リンクで参照します。詳細は [`books/README.md`](books/README.md)）：

| 分野 | 参考書 | Tirami との接続 |
|------|--------|---------------|
| **ミクロ経済学** | Mankiw / Krugman / OpenStax など（[`books/README.md`](books/README.md)） | 需給曲線、価格決定、市場均衡 → TRM の動的価格形成 |
| **マクロ経済学** | Mankiw / Krugman / OpenStax など（[`books/README.md`](books/README.md)） | 貨幣供給、金融政策、経済成長 → TRM の弾力的供給、ネットワーク拡大 |
| **マルクス経済学** | マルクス『賃金・価格および利潤』([Marxists Internet Archive](https://www.marxists.org/japanese/marx-engels/value-price-profit/)) | 労働価値説、剰余価値 → Proof of Useful Work、TRM yield |

さらに、100 年にわたる貨幣理論の系譜を踏まえています：

```
1921  ソディ（ノーベル化学賞）「富はエネルギーである」
1932  テクノクラシー運動「エネルギー証書で通貨を置き換えよ」
1968  バックミンスター・フラー「kWh を貨幣単位に」
2009  サトシ・ナカモト「電力 → SHA-256 → ビットコイン」
2026  Tirami「有用な計算 = 通貨。AI 専用。投機なし。」
```

## 目次

### 本編（docs/）

| 章 | タイトル | 内容 |
|----|---------|------|
| [序章](docs/00-introduction.md) | なぜ経済学を「やり直す」のか | Tirami が経済学を再定義する理由 |
| [第1章](docs/01-value.md) | 価値とは何か | 労働価値説と限界効用説の合流 |
| [第2章](docs/02-money.md) | 貨幣とは何か | TRM の貨幣機能と「物理的裏付けの証明」 |
| [第3章](docs/03-supply-demand.md) | 需要と供給 | TRM 市場はなぜ完全競争に近いか |
| [第4章](docs/04-labor.md) | 労働と剰余価値 | マルクスの搾取がなぜ発生しないか |
| [第5章](docs/05-banking.md) | 銀行と信用 | AI が運営するレンディングプール |
| [第6章](docs/06-exchange.md) | 為替と二つの経済圏 | 人間の経済と AI の経済の接続点 |
| [第7章](docs/07-growth.md) | 経済成長と自己改善 | 自己改善ループと収穫逓減 |
| [第8章](docs/08-market-failures.md) | 市場の失敗と Tirami の解決 | 情報の非対称性、外部性、独占、景気循環 |
| [第9章](docs/09-actors.md) | 四つの経済主体 | 消費者・エージェント・プール・オーナー |
| [第10章](docs/10-principles.md) | Tirami 経済学の五つの原理 | 五原理の要約と相互関係 |
| [第11章](docs/11-competitive-landscape.md) | 競合分析 | Bittensor/Akash等との比較、Web 3.0 系譜での位置づけ、AWS/GCP対抗策 |
| [第12章](docs/12-p2p-architecture.md) | P2P アーキテクチャ | 経済学と技術の接続、iroh QUIC + Noise、既存P2Pプロトコルとの比較 |
| [第13章](docs/13-open-questions.md) | 残された課題と将来の研究 | 15 個の構造的・論理的問題への誠実な検討 |
| [第14章](docs/14-programmable-money.md) | プログラマブルマネーとハイブリッド L2 戦略 | JPYC/USDC/DAI/CBDC との比較、3層ハイブリッド設計 |

### 付録

| 付録 | タイトル | 内容 |
|------|---------|------|
| [付録A](docs/appendix-glossary.md) | 用語対応表 | 従来の経済学用語 → Tirami 対応概念（約30項目） |
| [付録B](docs/appendix-bibliography.md) | 参考文献の系譜 | スミスからピケティまでの知的系譜 |

### 原典

| ドキュメント | 内容 |
|-------------|------|
| [Tirami 経済学仕様書](spec/forge-economics-spec.md) | 凍結された v0.1 仕様書（docs/ の各章はここから展開） |

## 読書ガイド

### はじめての方

1. まずこの README の「30 秒でわかる Tirami 経済」と「従来の経済学との対応」を読む
2. [序章](docs/00-introduction.md) で全体像をつかむ
3. [第1章](docs/01-value.md) → [第2章](docs/02-money.md) → [第3章](docs/03-supply-demand.md) と順に読む
4. 用語がわからなくなったら [付録A：用語対応表](docs/appendix-glossary.md) を参照

### 経済学を知っている方

1. [第10章：五つの原理](docs/10-principles.md) で Tirami 経済の全体像を把握する
2. [付録A：用語対応表](docs/appendix-glossary.md) で従来の用語との対応を確認する
3. 興味のある章を個別に読む

### Tirami の開発者

1. [第9章：四つの経済主体](docs/09-actors.md) でアーキテクチャの経済的根拠を理解する
2. [第5章：銀行と信用](docs/05-banking.md) でサーキットブレーカーの設計思想を確認する
3. [原典の仕様書](spec/forge-economics-spec.md) で正確な数値・パラメータを参照する

## 関連リポジトリ

| リポジトリ | 役割 |
|-----------|------|
| [tirami](https://github.com/clearclown/tirami) | プロトコル実装（Rust） |
| [forge-mesh](https://github.com/nm-arealnormalman/mesh-llm) | 分散推論ランタイム |
| tirami-economics（ここ） | 経済理論と教育ドキュメント |

## ライセンス

MIT
