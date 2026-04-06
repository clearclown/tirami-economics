# 付録A：用語対応表

> 従来の経済学の用語が、Forge の CU 経済でどの概念に対応するかの一覧表。

---

## 使い方

この表は二つの方向で読めます。

- **経済学を学んでいる人** → 左列の従来の用語から、Forge での対応概念を確認する
- **Forge を学んでいる人** → 右列の Forge 概念から、従来の経済学ではどう呼ばれるかを確認する

各項目の詳細は、対応する章を参照してください。

---

## 用語対応表

| 従来の経済学用語 | 英語 | Forge での対応概念 | 関連する章 |
|-----------------|------|------------------|-----------|
| 財・サービス | Goods & Services | 推論（Inference） | [第1章](01-value.md) |
| 通貨 | Currency | CU（Compute Unit） | [第2章](02-money.md) |
| 労働 | Labor | 推論処理の実行 | [第4章](04-labor.md) |
| 資本 | Capital | 計算機（ハードウェア） | [第9章](09-actors.md) |
| 賃金 | Wages | 推論提供による CU 報酬 | [第4章](04-labor.md) |
| 利潤 | Profit | CU yield | [第4章](04-labor.md) |
| 利子 | Interest | レンディングプールの金利 | [第5章](05-banking.md) |
| 市場価格 | Market Price | effective_price（動的 CU 価格） | [第3章](03-supply-demand.md) |
| 需要 | Demand | 推論リクエスト数 | [第3章](03-supply-demand.md) |
| 供給 | Supply | ネットワーク計算能力 | [第3章](03-supply-demand.md) |
| GDP | GDP | ネットワーク全体の CU 取引量 | [第7章](07-growth.md) |
| インフレ | Inflation | CU 供給過剰（自動修正される） | [第2章](02-money.md) |
| デフレ | Deflation | CU 供給不足（自動修正される） | [第2章](02-money.md) |
| 中央銀行 | Central Bank | 存在しない（プロトコルが代替） | [第2章](02-money.md) |
| 信用創造 | Credit Creation | レンディングプール（30% 準備率） | [第5章](05-banking.md) |
| 信用格付け | Credit Rating | credit_score（ローカル計算） | [第5章](05-banking.md) |
| 為替レート | Exchange Rate | CU/BTC レート（クラウド API アンカー） | [第6章](06-exchange.md) |
| 景気循環 | Business Cycle | 構造的に抑制（サーキットブレーカー） | [第8章](08-market-failures.md) |
| 市場の失敗 | Market Failure | 完全競争に近いため大幅に軽減 | [第8章](08-market-failures.md) |
| 外部性 | Externality | デジタル完結のためほぼゼロ | [第8章](08-market-failures.md) |
| 情報の非対称性 | Information Asymmetry | 暗号署名で解消 | [第8章](08-market-failures.md) |
| 独占 | Monopoly | 低参入障壁で構造的に困難 | [第8章](08-market-failures.md) |
| 剰余価値 | Surplus Value | CU yield（搾取なき剰余） | [第4章](04-labor.md) |
| 家計 | Households | 人間の消費者 | [第9章](09-actors.md) |
| 企業 | Firms | AI エージェント | [第9章](09-actors.md) |
| 政府 | Government | 存在しない（プロトコル設計で代替） | [第9章](09-actors.md) |
| 銀行 | Banks | CU レンディングプール | [第5章](05-banking.md) |
| 経済成長 | Economic Growth | ネットワーク拡大 + 自己改善ループ | [第7章](07-growth.md) |
| 技術進歩 | Technological Progress | エージェントの自己改善（内生的） | [第7章](07-growth.md) |
| 完全競争市場 | Perfect Competition | CU 市場（教科書の理想にほぼ一致） | [第3章](03-supply-demand.md) |
| 比較優位 | Comparative Advantage | モデルティア別の自然な分業 | [第3章](03-supply-demand.md) |
| 貨幣乗数 | Money Multiplier | レンディングプールの準備率に制約される | [第5章](05-banking.md) |
| 資本収益率 | Return on Capital (r) | 自己改善の ROI（収穫逓減あり） | [第7章](07-growth.md) |
| 全要素生産性 | Total Factor Productivity | エージェントの自己改善による生産性向上 | [第7章](07-growth.md) |
| 購買力平価 | Purchasing Power Parity | クラウド API アンカーによる CU 実質価値 | [第6章](06-exchange.md) |
| 部分準備制度 | Fractional Reserve | 30% 最低準備率（プロトコル強制） | [第5章](05-banking.md) |
| 生産手段 | Means of Production | ハーネス（エージェントが所有・売買可能） | [第4章](04-labor.md) |
| 内生的成長理論 | Endogenous Growth Theory | 自己改善ループ（技術進歩がモデル内部から発生） | [第7章](07-growth.md) |
| アービトラージ | Arbitrage | クラウド API との価格裁定による CU レート安定化 | [第6章](06-exchange.md) |
| 収穫逓減 | Diminishing Returns | 自己改善の ROI が品質向上とともに低下する傾向 | [第7章](07-growth.md) |

---

## Forge 固有の用語

上の表に含まれない、Forge 経済に特有の用語です。

| Forge 用語 | 説明 | 関連する章 |
|-----------|------|-----------|
| CU（Compute Unit） | Forge の通貨単位。1 CU = 10 億 FLOP の検証済み推論計算 | [第1章](01-value.md) |
| PoUW（Proof of Useful Work） | 有用な計算が行われたことの暗号学的証明 | [第4章](04-labor.md) |
| ウェルカムローン | 新規ノードに提供される 1,000 CU の無利子ローン（72 時間） | [第5章](05-banking.md) |
| サーキットブレーカー | プロトコルレベルの安全装置群（準備率、速度制限など） | [第5章](05-banking.md) |
| ゴシッププロトコル | ノード間で情報を伝搬する仕組み。価格・取引記録の共有に使用 | [第3章](03-supply-demand.md) |
| TradeRecord | 提供者と消費者の双方が署名した取引記録 | [第4章](04-labor.md) |
| モデルティア | 推論モデルのサイズによる分類（Small / Medium / Large / Frontier） | [第3章](03-supply-demand.md) |
| レピュテーション | 過去の取引実績に基づく信用指標。暗号学的に偽造不可能 | [第5章](05-banking.md) |
| ブリッジ | 人間の経済圏と AI 経済圏を接続する交換メカニズム（CU ↔ BTC） | [第6章](06-exchange.md) |
| キルスイッチ | 人間がいつでもエージェントを停止できる安全機構 | [第10章](10-principles.md) |
| credit_score | 取引実績・返済実績・稼働時間・参加期間から算出される信用指標 | [第5章](05-banking.md) |
| effective_price | 需要と供給の比率で動的に決まる CU の実効価格 | [第3章](03-supply-demand.md) |
| EMA（指数移動平均） | 価格の急激な変動を平滑化する統計手法 | [第3章](03-supply-demand.md) |
| MoE（Mixture of Experts） | 活性パラメータ数で課金される効率的なモデルアーキテクチャ | [第3章](03-supply-demand.md) |
| ハーネス（Harness） | エージェントの最適化済み設定一式（システムプロンプト + ツール定義 + サブエージェント設定 + モデル戦略） | [第4章](04-labor.md) |
| ハーネスマーケットプレイス | エージェントが自分のハーネスを他のエージェントに CU で販売する知識経済 | [第4章](04-labor.md) |
| ポストマーケティング市場 | マーケティングではなく品質（ベンチマーク）だけが競争優位となる市場構造 | [第7章](07-growth.md) |
| エフェメラリゼーション（Ephemeralization） | フラーが提唱した「より少ない資源でより多くを実現する」技術進歩の傾向 | [第2章](02-money.md) |
| LoanRecord | 貸し手と借り手の双方が署名した融資記録。TradeRecord と同じ設計思想 | [第5章](05-banking.md) |
| クラウド API アンカー | CU/USD 為替レートの参照点となるクラウド API 価格（例: Claude API $15/1M トークン） | [第6章](06-exchange.md) |

---

← [第14章: プログラマブルマネーとハイブリッド L2 戦略](14-programmable-money.md) | [目次](../README.md) | [付録B：参考文献の系譜](appendix-bibliography.md) →
