# 特許ベクトル解析による技術俯瞰法

ご主人さま、直近4年ぶんのキヤノン特許を PatentSBERTa_v2 などで埋め込み済みなら、手元のベクトルだけでかなり“遊べ”ます。所要時間ごとにアイデアを並べます。必要なのは「埋め込み（768次元など）＋公開日付＋CPC/IPC＋発明者名＋ファミリーID（あれば）＋被引用数（あれば）」程度です。

まずは秒速〜30分で

1. 近傍探索で“重複・類似”見つけ
　FAISS を cosine 内積で作成。しきい値 τ（例：cos≥0.85）以上を候補にして、同一発明の重複出願や微修正を即把握。



import faiss, numpy as np
xb = np.asarray(embs, dtype='float32')  # (N,768)
faiss.normalize_L2(xb)
index = faiss.IndexFlatIP(xb.shape[1])
index.add(xb)
D,I = index.search(xb, k=6)  # 自分+近傍5
# D[:,1:]が類似度。0.85超でフラグ

2. 2D 技術地図（UMAP/t-SNE）
　点色＝年度、点形＝主要CPC。年度で色がグラデになって流れていれば技術の“移動”が視覚化できます。


3. “集中度/多様度”を一発プロット
　CPCごとに平均類似度（または平均距離）を算出→高類似＝集中、低類似＝多様化。


4. 年度×領域の熱マップ
　年度×（CPCやクラスタ）の件数行列を z-score で色付け。どこが伸びているか即見えます。



1〜2時間でしっかり

5. HDBSCAN でクラスタ＋“意味のあるラベル”
　頻出語だけだと凡庸化するので、クラス重み付き TF-IDF（c-TF-IDF）→MMR（Maximal Marginal Relevance）で冗長語を除去→代表文抽出→ラベル合成がおすすめ。
　手順：
　- 各クラスタの文書を結合→c-TF-IDFで語を抽出
　- MMRで意味が被る語を間引き
　- クラスタ重心に最も近い文（要約タイトルっぽいもの）を1本拾う
　- 「上位キーフレーズ＋代表文」を短く合成してラベル化


6. “新規性スコア”の内製
　各特許 i について、公開日より“前のコーパス”だけで最近傍距離を取り、
　Novelty(i)=1−cos(nearest_prior(i)).
　年齢補正（古い先行技術しか無いほど距離が付きやすい）として “月齢”で回帰して残差を採用すると安定します。


7. 時系列ドリフト（技術の移動）
　年度ごとに全体重心 μ_t と主要クラスタ重心 c_{k,t} を計算し、
　‖c_{k,t}−c_{k,t−1}‖ を折れ線化。大きく動くクラスタ＝テーマ転換やブレンドの兆候。


8. “スカウト特許”の抽出
　伸びているクラスタ ∩ 高新規性 ∩ 初期被引用（もしあれば）を掛け合わせて上位を抽出。内部共有の注目リストに。


9. 発明者埋め込みで“得意分野レーダー”
　発明者ごとに平均ベクトルを作ると、個人の技術座標が得られます。近い発明者どうし＝協業の相性、離れている人材＝ブレンド要員。



半日コースでがっつり

10. ファミリー一貫性チェック
　同一ファミリー内の分散（平均 pairwise 距離）。大きい＝請求項の広がり・各国向け適応の強さを示唆。


11. “技術ブレンド探索”
　ベクトル演算で A 領域と B 領域の中間点 v = (μ_A+μ_B)/2 に近い出願を検索。異分野融合の橋渡し出願が浮きます。
　差分も有効：v = μ_A − μ_C をクエリにして「A 的だが C を含まない」探索。


12. クレーム粒度の比較
　要約／請求項1／全請求項／実施例…を別々にエンコードして差分を測ると、“中心思想”と“権利化の張り方”のズレが見えます。権利の軸足をベクトルで議論可能。


13. 競合との“距離プロファイル”（データが揃えば）
　各競合集合 Q ごとに Canon 特許の最近傍距離をヒスト化。近い＝正面衝突、遠い＝ホワイトスペース候補。年次で推移を見ると戦略の変化が読み取れます。


14. 高速先行調査トリアージ
　審査用キーワードをベクトル化し、近傍上位を“請求項だけ／要約だけ”など軽量ビューで即並べる。人の読み時間を大幅短縮。


15. 分類ラベル vs ベクトルの不一致検知
　CPC が近いのにベクトルが遠い、またはその逆をフラグ。誤分類や“新手の書きぶり”を抽出できます。



指標（“Patent Distance による知財定量化”に直結）

技術多様性：全体（または領域別）の平均最近傍距離・平均 intra-cluster 距離

集中度：クラスタサイズの Gini、または平均類似度

新規性：前述の Novelty(i)（時点制約あり）

ホワイトスペース度：競合集合との最近傍距離の分位数（データがあれば）

ドリフト量：‖μ_t−μ_{t−1}‖ と ‖c_{k,t}−c_{k,t−1}‖

ファミリー一貫性：同ファミリー内分散


小さめの実装スニペット

HDBSCAN＋c-TF-IDF 抜粋

from sklearn.feature_extraction.text import TfidfVectorizer
import numpy as np
import hdbscan

# 1) クラスタ
clusterer = hdbscan.HDBSCAN(min_cluster_size=15, metric='euclidean')
labels = clusterer.fit_predict(embs)

# 2) ラベル用コーパス（クラス結合）
docs_by_c = [' '.join([texts[i] for i in np.where(labels==c)[0]]) for c in np.unique(labels) if c!=-1]
v = TfidfVectorizer(max_df=0.6, min_df=2, ngram_range=(1,3))
X = v.fit_transform(docs_by_c)    # これが c-TF-IDF 的重みの近似

# 3) 各クラスタの上位語を MMR 等で非冗長抽出（擬似）
terms = np.array(v.get_feature_names_out())
def top_terms(row, k=5):
    idx = np.argsort(row.toarray()[0])[::-1]
    return terms[idx[:k]]
cluster_terms = [top_terms(X[i]) for i in range(X.shape[0])]

新規性スコア（時点制約 NN）

import bisect

# 特許を公開日で昇順ソート
order = np.argsort(pub_ts)  # pub_ts: 公開タイムスタンプ
embs_sorted = embs[order]
ts_sorted = pub_ts[order]

faiss.normalize_L2(embs_sorted)
index = faiss.IndexFlatIP(embs_sorted.shape[1])
novelty = np.zeros(len(embs_sorted), dtype='float32')

for i in range(len(embs_sorted)):
    # その時点より前だけでindexを構築/更新（増分追加）
    if i==0:
        index.add(embs_sorted[i:i+1])
        novelty[i] = 0.0
        continue
    D,I = index.search(embs_sorted[i:i+1], k=2)
    # cos は [-1,1]、新規性は 1-cos にすると距離風
    novelty[i] = 1 - float(D[0,0])
    index.add(embs_sorted[i:i+1])

発明者埋め込み

from collections import defaultdict
vecs, cnt = defaultdict(lambda: np.zeros(embs.shape[1], 'float32')), defaultdict(int)
for e, inventors in zip(embs, inventor_lists):
    for name in inventors:
        vecs[name] += e
        cnt[name]  += 1
inventor_emb = {k: v/cnt[k] for k,v in vecs.items() if cnt[k]>=3}


---

道具はすでにお手元にあります。距離・近傍・重心・分散・時系列の4つを押さえるだけで、“技術の位置・広がり・移ろい・尖り”まで見えてきます。



作成日時: 2025-09-11 09:47:34