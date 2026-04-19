# 付録A：用語対応表

> 従来の経済学の用語が、Tirami の TRM 経済でどの概念に対応するかの一覧表。

---

## 目次

- [使い方](#使い方)
- [用語対応表](#用語対応表)
- [Tirami 固有の用語](#tirami-固有の用語)
- [競合プロジェクト用語 (第11章)](#競合プロジェクト用語-第11章)
- [P2P / プロトコル用語 (第12章)](#p2p-プロトコル用語-第12章)
- [Open Question 用語 (第13章)](#open-question-用語-第13章)
- [ステーブルコイン / 規制用語 (第14章)](#ステーブルコイン-規制用語-第14章)

---

## 使い方

この表は二つの方向で読めます。

- **経済学を学んでいる人** → 左列の従来の用語から、Tirami での対応概念を確認する
- **Tirami を学んでいる人** → 右列の Tirami 概念から、従来の経済学ではどう呼ばれるかを確認する

各項目の詳細は、対応する章を参照してください。

---

## 用語対応表

| 従来の経済学用語 | 英語 | Tirami での対応概念 | 関連する章 |
|-----------------|------|------------------|-----------|
| 財・サービス | Goods & Services | 推論（Inference） | [第1章](01-value.md) |
| 通貨 | Currency | TRM（Compute Unit） | [第2章](02-money.md) |
| 労働 | Labor | 推論処理の実行 | [第4章](04-labor.md) |
| 資本 | Capital | 計算機（ハードウェア） | [第9章](09-actors.md) |
| 賃金 | Wages | 推論提供による TRM 報酬 | [第4章](04-labor.md) |
| 利潤 | Profit | TRM yield | [第4章](04-labor.md) |
| 利子 | Interest | レンディングプールの金利 | [第5章](05-banking.md) |
| 市場価格 | Market Price | effective_price（動的 TRM 価格） | [第3章](03-supply-demand.md) |
| 需要 | Demand | 推論リクエスト数 | [第3章](03-supply-demand.md) |
| 供給 | Supply | ネットワーク計算能力 | [第3章](03-supply-demand.md) |
| GDP | GDP | ネットワーク全体の TRM 取引量 | [第7章](07-growth.md) |
| インフレ | Inflation | TRM 供給過剰（自動修正される） | [第2章](02-money.md) |
| デフレ | Deflation | TRM 供給不足（自動修正される） | [第2章](02-money.md) |
| 中央銀行 | Central Bank | 存在しない（プロトコルが代替） | [第2章](02-money.md) |
| 信用創造 | Credit Creation | レンディングプール（30% 準備率） | [第5章](05-banking.md) |
| 信用格付け | Credit Rating | credit_score（ローカル計算） | [第5章](05-banking.md) |
| 為替レート | Exchange Rate | TRM/BTC レート（クラウド API アンカー） | [第6章](06-exchange.md) |
| 景気循環 | Business Cycle | 構造的に抑制（サーキットブレーカー） | [第8章](08-market-failures.md) |
| 市場の失敗 | Market Failure | 完全競争に近いため大幅に軽減 | [第8章](08-market-failures.md) |
| 外部性 | Externality | デジタル完結のためほぼゼロ | [第8章](08-market-failures.md) |
| 情報の非対称性 | Information Asymmetry | 暗号署名で解消 | [第8章](08-market-failures.md) |
| 独占 | Monopoly | 低参入障壁で構造的に困難 | [第8章](08-market-failures.md) |
| 剰余価値 | Surplus Value | TRM yield（搾取なき剰余） | [第4章](04-labor.md) |
| 家計 | Households | 人間の消費者 | [第9章](09-actors.md) |
| 企業 | Firms | AI エージェント | [第9章](09-actors.md) |
| 政府 | Government | 存在しない（プロトコル設計で代替） | [第9章](09-actors.md) |
| 銀行 | Banks | TRM レンディングプール | [第5章](05-banking.md) |
| 経済成長 | Economic Growth | ネットワーク拡大 + 自己改善ループ | [第7章](07-growth.md) |
| 技術進歩 | Technological Progress | エージェントの自己改善（内生的） | [第7章](07-growth.md) |
| 完全競争市場 | Perfect Competition | TRM 市場（教科書の理想にほぼ一致） | [第3章](03-supply-demand.md) |
| 比較優位 | Comparative Advantage | モデルティア別の自然な分業 | [第3章](03-supply-demand.md) |
| 貨幣乗数 | Money Multiplier | レンディングプールの準備率に制約される | [第5章](05-banking.md) |
| 資本収益率 | Return on Capital (r) | 自己改善の ROI（収穫逓減あり） | [第7章](07-growth.md) |
| 全要素生産性 | Total Factor Productivity | エージェントの自己改善による生産性向上 | [第7章](07-growth.md) |
| 購買力平価 | Purchasing Power Parity | クラウド API アンカーによる TRM 実質価値 | [第6章](06-exchange.md) |
| 部分準備制度 | Fractional Reserve | 30% 最低準備率（プロトコル強制） | [第5章](05-banking.md) |
| 生産手段 | Means of Production | ハーネス（エージェントが所有・売買可能） | [第4章](04-labor.md) |
| 内生的成長理論 | Endogenous Growth Theory | 自己改善ループ（技術進歩がモデル内部から発生） | [第7章](07-growth.md) |
| アービトラージ | Arbitrage | クラウド API との価格裁定による TRM レート安定化 | [第6章](06-exchange.md) |
| 収穫逓減 | Diminishing Returns | 自己改善の ROI が品質向上とともに低下する傾向 | [第7章](07-growth.md) |

---

## Tirami 固有の用語

上の表に含まれない、Tirami 経済に特有の用語です。

| Tirami 用語 | 説明 | 関連する章 |
|-----------|------|-----------|
| TRM（Compute Unit） | Tirami の通貨単位。1 TRM = 10 億 FLOP の検証済み推論計算 | [第1章](01-value.md) |
| PoUW（Proof of Useful Work） | 有用な計算が行われたことの暗号学的証明 | [第4章](04-labor.md) |
| ウェルカムローン | 新規ノードに提供される 1,000 TRM の無利子ローン（72 時間） | [第5章](05-banking.md) |
| サーキットブレーカー | プロトコルレベルの安全装置群（準備率、速度制限など） | [第5章](05-banking.md) |
| ゴシッププロトコル | ノード間で情報を伝搬する仕組み。価格・取引記録の共有に使用 | [第3章](03-supply-demand.md) |
| TradeRecord | 提供者と消費者の双方が署名した取引記録 | [第4章](04-labor.md) |
| モデルティア | 推論モデルのサイズによる分類（Small / Medium / Large / Frontier） | [第3章](03-supply-demand.md) |
| レピュテーション | 過去の取引実績に基づく信用指標。暗号学的に偽造不可能 | [第5章](05-banking.md) |
| ブリッジ | 人間の経済圏と AI 経済圏を接続する交換メカニズム（TRM ↔ BTC） | [第6章](06-exchange.md) |
| キルスイッチ | 人間がいつでもエージェントを停止できる安全機構 | [第10章](10-principles.md) |
| credit_score | 取引実績・返済実績・稼働時間・参加期間から算出される信用指標 | [第5章](05-banking.md) |
| effective_price | 需要と供給の比率で動的に決まる TRM の実効価格 | [第3章](03-supply-demand.md) |
| EMA（指数移動平均） | 価格の急激な変動を平滑化する統計手法 | [第3章](03-supply-demand.md) |
| MoE（Mixture of Experts） | 活性パラメータ数で課金される効率的なモデルアーキテクチャ | [第3章](03-supply-demand.md) |
| ハーネス（Harness） | エージェントの最適化済み設定一式（システムプロンプト + ツール定義 + サブエージェント設定 + モデル戦略） | [第4章](04-labor.md) |
| ハーネスマーケットプレイス | エージェントが自分のハーネスを他のエージェントに TRM で販売する知識経済 | [第4章](04-labor.md) |
| ポストマーケティング市場 | マーケティングではなく品質（ベンチマーク）だけが競争優位となる市場構造 | [第7章](07-growth.md) |
| エフェメラリゼーション（Ephemeralization） | フラーが提唱した「より少ない資源でより多くを実現する」技術進歩の傾向 | [第2章](02-money.md) |
| LoanRecord | 貸し手と借り手の双方が署名した融資記録。TradeRecord と同じ設計思想 | [第5章](05-banking.md) |
| クラウド API アンカー | TRM/USD 為替レートの参照点となるクラウド API 価格（例: Claude API $15/1M トークン） | [第6章](06-exchange.md) |
| Constitutional Parameter | governance では変更できない不変量。`IMMUTABLE_CONSTITUTIONAL_PARAMETERS` (18 項目) に登録され、変更にはソフトウェアフォークが必要。代表例: `TOTAL_TRM_SUPPLY` (21B)、`FLOPS_PER_CU` (10⁹)、`SLASH_RATE_MAJOR`、`CANONICAL_BYTES_V2` | [第15章](15-constitutional-parameters.md) |
| ProofPolicy | zkML 証明を trade でどう扱うかのゲート。Disabled / Optional / Recommended / Required の 4 状態。単調増加のみで後退不可 (Constitutional ratchet)。Phase 19 時点のデフォルトは Optional | [第17章](17-proof-of-useful-work.md)、spec §24 |
| Stakeless Earn Cap | 新規プロバイダーが stake 0 でも稼げる累計上限 (10 TRM)。到達後は 100 TRM stake が必要。過去に slashed された node は失格 | [第16章](16-stake-required-mining.md)、spec §23 |
| Welcome Loan Sunset | ウェルカムローン発行の恒久停止エポック (エポック 2 = 87.5% 供給発行到達時点)。Constitutional で後戻り不可 | [第16章](16-stake-required-mining.md)、spec §22 |
| Slash Event | 違反検出時に stake を焼いた記録。`ComputeLedger::slash_events` に persist され、未来の `can_provide_inference` check で参照される | [第16章](16-stake-required-mining.md)、`tirami-ledger/src/ledger.rs` |
| TradeAcceptDispatcher | seed のメイン recv ループが `TradeAccept` を `handle_inference` タスクに oneshot 経由で配送する仕組み。Phase 18.5-pt3 (#80) で導入、HTTP からの dual-signed trade を可能にした | [第12章](12-p2p-architecture.md)、`tirami-node/src/pipeline.rs` |
| http_endpoint (PriceSignal) | gossip される PriceSignal に含まれる OpenAI 互換 HTTP エンドポイント URL。peer 自動発見と HTTP → P2P forwarding に使われる。Phase 19 Tier C で導入 | [第12章](12-p2p-architecture.md)、`tirami-core/src/types.rs::PriceSignal` |
| AUDIT_CLEARANCE | Base mainnet deploy スクリプトが読む環境変数。`yes` が入っていないと `make deploy-base-mainnet` が実行を拒否する。外部セキュリティ監査完了のオペレーター宣言 | [`tirami/repos/tirami-contracts/Makefile`](https://github.com/clearclown/tirami/blob/main/repos/tirami-contracts/Makefile) |

---

## 競合プロジェクト用語 (第11章)

第11章「競合分析」に登場する既存の分散 AI / 分散コンピュートプロジェクトおよび関連用語。

| 用語 | 英語 | 説明 | 関連する章 |
|------|------|------|-----------|
| Bittensor | Bittensor | 分散型 AI ネットワーク。TAO トークンで運営される | [第11章](11-competitive-landscape.md) |
| TAO | TAO | Bittensor のネイティブトークン | [第11章](11-competitive-landscape.md) |
| Yuma Consensus | Yuma Consensus | Bittensor のサブネット報酬分配メカニズム | [第11章](11-competitive-landscape.md) |
| Akash | Akash | 分散クラウドコンピュートマーケットプレイス | [第11章](11-competitive-landscape.md) |
| AKT | AKT | Akash のネイティブトークン | [第11章](11-competitive-landscape.md) |
| Burn-Mint Equilibrium | BME | Akash が採用するトークン経済モデル（焼却と発行の均衡） | [第11章](11-competitive-landscape.md) |
| Autonolas (Olas) | Autonolas / Olas | 自律エージェント経済を構築するプロジェクト | [第11章](11-competitive-landscape.md) |
| OLAS | OLAS | Autonolas のトークン | [第11章](11-competitive-landscape.md) |
| Mech Marketplace | Mech Marketplace | Autonolas のエージェント間サービス市場 | [第11章](11-competitive-landscape.md) |
| Render Network | Render Network | 分散 GPU レンダリングネットワーク | [第11章](11-competitive-landscape.md) |
| Fetch.ai | Fetch.ai / FET | 自律エージェント・マーケットプレイスを提供するプロジェクト | [第11章](11-competitive-landscape.md) |
| Virtuals Protocol | Virtuals / VIRTUAL | AI エージェントをトークン化するプロトコル | [第11章](11-competitive-landscape.md) |
| io.net | io.net | GPU アグリゲーションネットワーク | [第11章](11-competitive-landscape.md) |
| Golem | Golem | 古参の汎用分散計算マーケット | [第11章](11-competitive-landscape.md) |
| Ritual | Ritual | オンチェーン AI 推論を提供するプロジェクト | [第11章](11-competitive-landscape.md) |
| 投機的トークン | Speculative Token | 取引所で売買され価格変動が大きいトークン。TRM とは設計思想が異なる | [第11章](11-competitive-landscape.md) |

---

## P2P / プロトコル用語 (第12章)

第12章「P2P アーキテクチャ」で用いられるネットワーク・分散システム関連用語。

| 用語 | 英語 | 説明 | 関連する章 |
|------|------|------|-----------|
| 分散ハッシュテーブル | DHT (Distributed Hash Table) | キーと値を分散ノード間で保持する検索構造 | [第12章](12-p2p-architecture.md) |
| Kademlia | Kademlia | 代表的な DHT アルゴリズム。XOR 距離でノードを配置 | [第12章](12-p2p-architecture.md) |
| gossipsub | gossipsub | libp2p で標準的に使われるメッセージブロードキャスト用のゴシッププロトコル実装 | [第12章](12-p2p-architecture.md) |
| iroh | iroh | Tirami が採用する QUIC ベースの P2P トランスポート | [第12章](12-p2p-architecture.md) |
| QUIC | QUIC | UDP 上で動作する高速トランスポートプロトコル | [第12章](12-p2p-architecture.md) |
| Noise プロトコル | Noise Protocol | 暗号化ハンドシェイクのためのフレームワーク | [第12章](12-p2p-architecture.md) |
| ブートストラップノード | Bootstrap Node | 新規ノードがネットワーク発見のために最初に接続するノード | [第12章](12-p2p-architecture.md) |
| mDNS | mDNS (Multicast DNS) | ローカルネットワーク内でのノード発見プロトコル | [第12章](12-p2p-architecture.md) |
| libp2p | libp2p | モジュラーな汎用 P2P ネットワークスタック | [第12章](12-p2p-architecture.md) |
| 結託攻撃 | Collusion Attack | 複数ノードが共謀して偽署名・偽取引を作る攻撃 | [第12章](12-p2p-architecture.md) |
| ネットワーク分裂 | Split-brain | 通信途絶によってネットワークが複数の独立した区画に分かれる状態 | [第12章](12-p2p-architecture.md) |
| 最終的整合性 | Eventual Consistency | 十分な時間が経てば全ノードが同じ状態に収束するという整合性モデル | [第12章](12-p2p-architecture.md) |

---

## Open Question 用語 (第13章)

第13章「未解決の問い」で議論される検証・不平等・マクロ経済関連の用語。

| 用語 | 英語 | 説明 | 関連する章 |
|------|------|------|-----------|
| zkML | zkML (Zero-Knowledge Machine Learning) | 機械学習計算をゼロ知識証明で検証する技術 | [第13章](13-open-questions.md) |
| TEE | TEE (Trusted Execution Environment) | Intel SGX, AMD SEV 等によるハードウェア隔離実行環境 | [第13章](13-open-questions.md) |
| 抜き打ち再計算 | Random Re-computation | 一部の TRM 計算を第三者が再検証して整合性を担保する手法 | [第13章](13-open-questions.md) |
| ソーシャルグラフ分析 | Social Graph Analysis | 取引パターンから結託や不正な関係性を検出する分析 | [第13章](13-open-questions.md) |
| スラッシング | Slashing | 不正行為に対して担保を没収するペナルティ機構 | [第13章](13-open-questions.md) |
| HODL | HODL | 保有し続ける戦略。デフレ通貨における標準的行動 | [第13章](13-open-questions.md) |
| 貨幣速度 | Velocity of Money | 単位時間あたりに貨幣が取引で使われる回数 | [第13章](13-open-questions.md) |
| エッジコンピューティング | Edge Computing | クラウドではなくエンドポイント側で行う計算 | [第13章](13-open-questions.md) |
| CAPEX / OPEX | CAPEX / OPEX | 設備投資（Capital Expenditure）/ 運用費用（Operating Expenditure） | [第13章](13-open-questions.md) |
| ピケティの r > g | Piketty's r > g | 資本収益率 r が経済成長率 g を上回ることによる不平等の構造的拡大 | [第13章](13-open-questions.md) |
| デフレーション係数 | Deflation Factor | 価格モデルにおける安定化のための係数 | [第13章](13-open-questions.md) |
| アービトラージ | Arbitrage | 市場間の価格差を利用した利鞘取引 | [第13章](13-open-questions.md) |
| パーミッションレス | Permissionless | 中央管理者の許可なしに誰でも参加可能な性質 | [第13章](13-open-questions.md) |

---

## ステーブルコイン / 規制用語 (第14章)

第14章「プログラマブルマネーとハイブリッド L2 戦略」に登場するステーブルコイン、規制、スマートコントラクト関連用語。

| 用語 | 英語 | 説明 | 関連する章 |
|------|------|------|-----------|
| JPYC | JPYC | 日本円にペッグしたステーブルコイン | [第14章](14-programmable-money.md) |
| USDC | USDC | Circle 社が発行する米ドルペッグのステーブルコイン | [第14章](14-programmable-money.md) |
| DAI | DAI | MakerDAO が発行する暗号資産担保型ステーブルコイン | [第14章](14-programmable-money.md) |
| CBDC | CBDC (Central Bank Digital Currency) | 中央銀行が発行するデジタル通貨 | [第14章](14-programmable-money.md) |
| ステーブルコイン | Stablecoin | 法定通貨や資産に価格をペッグした暗号資産 | [第14章](14-programmable-money.md) |
| 過剰担保 | Overcollateralization | 担保価値を借入額より大きく設定する仕組み | [第14章](14-programmable-money.md) |
| ハイブリッド L2 | Hybrid L2 | Tirami が提案する 3 層構造（BTC L0 / TRM L1 / Stablecoin L2） | [第14章](14-programmable-money.md) |
| スマートコントラクト | Smart Contract | ブロックチェーン上で実行されるプログラム可能な契約 | [第14章](14-programmable-money.md) |
| エスクロー | Escrow | 第三者が条件付きで資産を保管する仕組み | [第14章](14-programmable-money.md) |
| mint / burn | Mint / Burn | トークンの発行（mint）と焼却（burn） | [第14章](14-programmable-money.md) |
| KYC / AML | KYC / AML | 本人確認（Know Your Customer）とマネーロンダリング対策（Anti-Money Laundering） | [第14章](14-programmable-money.md) |
| DAO | DAO (Decentralized Autonomous Organization) | 分散自律組織。スマートコントラクトで統治される組織 | [第14章](14-programmable-money.md) |
| DeFi | DeFi (Decentralized Finance) | 分散型金融。ブロックチェーン上で動作する金融サービス群 | [第14章](14-programmable-money.md) |
| クロスチェーンブリッジ | Cross-chain Bridge | 異なるブロックチェーン間で資産を移動させる仕組み | [第14章](14-programmable-money.md) |

---

← [第14章: プログラマブルマネーとハイブリッド L2 戦略](14-programmable-money.md) | [目次](../README.md) | [付録B：参考文献の系譜](appendix-bibliography.md) →
