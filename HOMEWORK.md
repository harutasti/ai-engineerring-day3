# SIGGRAPH 2024 物理シミュレーション ― 任意課題レポート
## 0. 概要
本課題では、SIGGRAPH 2024 の物理シミュレーション分野を解説した長文レポート（約15k word）を自作 **検索用コーパス** とし、  
- **ベース LLM（RAG なし）**  
- **RAG（検索 + GPT‑4o-mini）**  
の 2 系統で **6 問 QA** を実施し、精度を比較した。

---

## 1. 質問設計の観点と意図
|  | 質問 | 意図 | 必要コンテキスト |
|---|---|---|---|
| 1 | SIGGRAPHとは | 基本定義。LLM だけでも答え得る易問。 | 序論 |
| 2 | Cyclogenesis でシミュレーション対象となる気象現象は？ | 固有名詞 + 専門用語。RAG 効きやすい。 | II‑A |
| 3 | 「Primal-Dual Non-Smooth Friction for Rigid Body Animation」 が扱う課題は？ | 論文タイトル → 要旨抽出。 | IV‑A |
| 4 | 「Mesh Mortal Combat: Real-Time Voxelized Soft-Body Destruction」 はどの賞を受賞？ | Real‑Time Live! の情報を引く。 | III‑B |
| 5 | Best Paper Award を受賞した研究と分野は？ | 2 要素抽出。検索・合成力テスト。 | V‑B |
| 6 | ML を利用した物理simulation強化例を 2 件比較 | 複数チャンク統合 + 要約生成。 | V‑A |

> *難易度バランス*  
> 易 2 問 / 中 2 問 / 難 2 問 に設定し、RAG の効果を段階的に観察できるようにした。

---

## 2. RAG 実装と工夫点
| 項目 | 実装 |
|------|------|
| **分割 (chunking)** | `RecursiveCharacterTextSplitter(chunk_size=400, overlap=80, separators=[\n\n, 。, 、])` で段落ベースでの分割を行った|
| **検索** | FAISS cosine sim 上位 **k=3** の使用でで重複抑制を行った。|
| **評価** | Accuracy / Completeness / Relevance の三つの指標で LLM 採点 **(0‑2‑4)** を行った。| 

---

## 3. 結果と考察

| ケース | 観測 | 考察 |
|--------|------|------|
| #2 Cyclogenesis | Baseline が「気象現象」を曖昧に回答した (Accuracy=1)。しかし、RAG はレポート該当チャンクを引き当てて **ハリケーン・竜巻** と正答した (Accuracy=4)。 | 固有名詞 + 専門用語は外部知識が効果的ことが確認できた。 |
| #4 Mesh Mortal Combat | Baseline でも **Best in Show** を抽出した(Accuracy=3)。 — おそらくLLM の汎用知識に残っていた。RAG では完全一致した (Accuracy=4)。 | 受賞情報はモデルが学習済みであった。 |
| #5 Best Paper Award | どちらも誤答だった (Accuracy=0)。レポート内の「Repulsive Shells」の前後が 2 チャンクに跨り、検索が外れた。 | チャンク分割位置の悪さが原因である可能性が高い。overlap を増やすと改善余地あり。 |
| #6 ML 例の比較 | 両手法とも 0であった。**要約を 2 件要求**する難問で token 上限を超え生成が途中で切れた。 | `max_tokens` 拡大 or step-by-step 抽出→合成が必要。 |

---

## 4. 発展的な改善案
1. **チャンク戦略の最適化**  
   - セクション見出しごとに `metadata` を付与し、検索時にフィルタリンをする。  
2. **Retrieval‐Augmented Rerankers**  
   - overlapの調整を行い、#5 のような改善が期待できる。  



