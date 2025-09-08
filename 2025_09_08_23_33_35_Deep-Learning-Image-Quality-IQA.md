# 深層学習による画像品質評価の最前線

ディープラーニングによる画像品質評価（IQA）の主要アプローチ

ディープラーニングを用いた画像品質評価（Image Quality Assessment, IQA）は、従来の手法では難しかった人間の知覚に近い品質評価を可能にし、Full-Reference (FR) と No-Reference (NR) の両分野で急速に発展しています。FR-IQAは劣化画像と完璧な参照画像を比較して品質を評価し、NR-IQA（または盲目的IQA）は参照なしで単一画像の品質を推定します。カメラ映像や医療画像、監視カメラ映像、印刷物の品質管理など、Canonの事業領域でもIQAの応用が期待されており、実運用では参照画像を用意できない場合が多いためNR-IQAが重要ですが、製品評価やアルゴリズム開発段階ではFR-IQAも有用です。以下では、それぞれの分野における深層学習ベースの主要アプローチを、「アーキテクチャ設計」や「学習戦略」の観点からいくつかの流派に分類し、技術的背景や最新動向を整理します。

Full-Reference（FR）IQA における深層学習アプローチ

CNNベースのフルリファレンスIQAモデル（サイアミーズネットワーク型）

ディープラーニング初期には、FR-IQAに畳み込みニューラルネットワーク (CNN) を直接適用する試みが登場しました。その代表例がBosseらによる手法で、参照画像と劣化画像をペアで入力し品質スコアを出力する深いCNNモデルです。このモデルは10層のCNN特徴抽出部と回帰用の全結合層からなり、局所品質とその重み（各領域が全体品質に与える重要度）を同時に学習するユニークな構造を持ちます。従来のIQA手法のような人間視覚モデルや手工芸的特徴に頼らず純粋にデータ駆動で学習し、LIVEやTID2013など標準ベンチマークで当時の最先端性能を達成しました。このようなCNNによるFR-IQAは、画像処理アルゴリズムの評価（例：圧縮や超解像結果をゴールド標準と比較）や、印刷物のスキャン画像と元データの照合による品質検査などに応用が考えられます。

代表的論文: Sebastian Bosse et al., “Deep Neural Networks for No-Reference and Full-Reference Image Quality Assessment,” IEEE TIP, 27(1):206-219, 2018（※FR/NR統合モデル） ;  Kim et al., “Deep Learning of Human Visual Sensitivity in Image Quality Assessment,” ACM MM, 2017（DeepQA; FR専用CNNモデル）など。

知覚特徴に基づく類似度指標（LPIPS・PieAPP・DISTS など）

ディープラーニングの強力な特徴抽出能力を利用し、画像間の「知覚的距離」を学習するアプローチもFR-IQAで大きな成功を収めました。代表例の一つがLPIPS（Learned Perceptual Image Patch Similarity）で、Zhangらにより提案された手法です。LPIPSは事前学習された深層CNN（例：VGG）の中間特徴空間で参照画像と劣化画像の差異を計測し、そのチャンネル毎の差をユーザテストに基づき線形重み付けして距離を算出します。簡単に言えば、従来のピクセルごとの差分ではなく、高レベル特徴空間での距離を「知覚上の違い」として定義したものです。LPIPSは画像復元分野で用いられる知覚損失（perceptual loss）の延長線上にあり、人間の主観評価との相関が従来指標より大幅に向上することが示されています。またPrashnaniらのPieAPPは、2枚の劣化画像のうちどちらが参照に近いかというペアワイズ比較で大規模にデータを収集し、それを学習することで絶対的な品質スコアを予測可能にした手法です。このペア比較アプローチにより、従来は困難だった微小な知覚差異の学習に成功し、提案手法PieAPPは従来手法比で3倍もの精度向上を達成したと報告されています。さらにDingらのDISTS（Deep Image Structure and Texture Similarity）は、深層特徴から構造とテクスチャの情報を分離して比較する指標で、人間の知覚における「形」と「質感」の両面の差異を捉えることを狙っています。これら学習ベースの知覚指標（LPIPSやPieAPP、DISTS）は、古典的指標（PSNRやSSIMなど）が失敗しがちな**高次元な歪み（例：GANによる人工的なアーティファクトや質感の変化）**に強く、画像生成・変換アルゴリズムの評価にも広く用いられ始めています。実用面では、例えばカメラの画像処理アルゴリズム開発において、人間の視覚で好ましい改良が定量的に測れるメリットがあります。

代表的論文: Richard Zhang et al., “The Unreasonable Effectiveness of Deep Features as a Perceptual Metric,” CVPR 2018 (LPIPS);  Ekta Prashnani et al., “PieAPP: Perceptual Image-Error Assessment through Pairwise Preference,” CVPR 2018;  Keyan Ding et al., “Image Quality Assessment: Unifying Structure and Texture Similarity,” IEEE TPAMI, 2021 (DISTS).

Transformerを活用した最新FR-IQAモデル

近年はVision Transformerに代表される自己注意（Self-Attention）機構がIQAにも導入され、FR-IQAの性能向上に寄与しています。LG電子のCheonらはImage Quality Transformer (IQT)を提案し、CNNで抽出した参照画像・劣化画像それぞれの特徴マップをTransformerエンコーダ・デコーダに投入して両者の対応関係を学習させました。Transformerによりグローバルな文脈を捉えたIQTモデルは、標準ベンチマークで従来手法を上回る精度を示し、特にGANベースの画像生成モデルが生み出す高品質だが偽のディテール（従来指標では扱いが難しい）に対して有効性を発揮しました。実際、NTIRE 2021が主催した知覚画像品質評価チャレンジにおいてIQTは13チーム中1位を獲得しており、Transformerを用いた手法の有望性を示しています。Transformer型モデルは計算コストが高い傾向にありますが、FR-IQAの文脈ではオフライン評価であるため許容されるケースも多く、今後も応用が広がると考えられます。

代表的論文: Manri Cheon et al., “Perceptual Image Quality Assessment with Transformers,” CVPR Workshops 2021 (NTIREチャレンジ優勝手法IQT);  Jaehee Lee et al., “STEM: Spatial Transormer-based Entropic Measure for FR-IQA,” ECCV 2022 (Transformerによる空間的注意機構を組み込んだFR-IQA) など。

その他の最新動向：半教師あり・一般化FR-IQA

深層学習FR-IQAの発展に伴い、データ不足や参照画像の質への対処といった新たな課題にも取り組む流派があります。SidorovらのConIQAはCGH (計算機合成ホログラム) 分野向けに設計されたFR-IQAで、サイアミーズ型ネットワークを基盤に一部のラベル付きデータと大量の未ラベルデータを組み合わせて学習する半教師あり手法を導入しました。具体的には、ランダムな画像変換（水平反転や参照画像との重み付き平均など）による入力変動で予測スコアの一貫性を損失関数に加え、限られた人手評価データでも過学習せず高い一般化性能を引き出しています。一方、Maらの最新研究A-FINEは「参照画像＝完璧」という前提を緩和し、参照のほうが劣っている場合でも出力画像を高品質と評価できる柔軟なFRモデルを提案しました。A-FINEは劣化画像の忠実度（Fidelity）と自然さ（Naturalness）の二軸で品質を評価し、状況に応じて両者を適応的に統合します。これにより、例えば高度な超解像技術で元の参照よりも鮮明な画像が得られた場合でも、その鮮明さを正当に評価できるようになります。こうしたアプローチは、今後の高度な画像生成・改善技術（医療画像のノイズ除去や監視映像の向上など）におけるIQAの信頼性向上に寄与すると期待されます。

代表的論文: S. Sidorov et al., “ConIQA: Deep Learning-Based Image Quality Assessment for Optical Holography,” Scientific Reports, 13, 2023 (半教師あり学習の導入);  Du Chen et al., “Toward Generalized Image Quality Assessment: Relaxing the Perfect Reference Quality Assumption,” CVPR 2025 (A-FINEモデル：忠実度と自然さの適応評価).


---

No-Reference（NR）IQA における深層学習アプローチ

NR-IQA（盲目的品質評価）は参照画像が得られない実世界の応用（カメラで撮影された写真の自動画質評価、ネットワーク越しの監視映像の品質モニタ、医療スキャン画像の品質チェック等）で特に重要です。しかし参照なしで「この一枚の画像がどれだけ良い画質か？」を判定するのは極めて難しく、長らく自然画像統計（NSS）に基づく手法（例：ブロックノイズやぼやけを検出する手工的特徴）が主流でした。2010年代中頃からディープラーニングがこの分野にも導入され、現在では深層学習モデルが既存のNR-IQA手法を凌駕する精度を示しています。ここではNR-IQAにおける主要な深層学習ベース手法を、その歴史的発展とともに分類します。

初期のCNNベースNR-IQA（パッチ学習・浅層ネット）

最初期の試みとして、Kangらは2014年に5層程度の浅いCNNで画像パッチから品質を予測するモデルを提案しました。このネットワークは空間領域の生画像パッチを直接入力とし、畳み込み層とプーリング、2層の全結合層を経てパッチごとの品質スコアを出力します。その後画像全体のスコアはパッチスコアの平均で算出されました。当時主流だったNSS特徴ベースの手法を抑え、LIVEデータセットで最高性能を達成したほか、学習に用いていない別データセットにも良好に汎化することが報告されています。浅いとはいえCNNが特徴抽出と回帰を一体化して学習する利点を示したこの研究は、ディープラーニングによるNR-IQAの端緒となりました。もっとも、この手法では学習データ（LIVE：約800枚）の制約上、見たことのない歪みや多様な画像内容への適応には限界がありました。その後の研究では、ネットワークの深化とともにデータの拡張（自動生成や大規模収集）が重要なテーマとなっていきます。

代表的論文: Le Kang et al., “Convolutional Neural Networks for No-Reference Image Quality Assessment,” CVPR 2014 (初のCNN盲目的IQAモデル) ;  P. Ye et al., “From Coarse to Fine: A CNN-Based Blind Quality Assessment for Induction Machine Images,” IEEE SPL, 2016 (浅いCNNによるNR-IQAのバリエーション)。

ランキング学習・事前学習を活用した手法

ディープラーニングNR-IQAの発展において、「限られた主観評価データをどう補うか」という問題に対し、他タスクでの事前学習やデータ合成による学習戦略が考案されました。代表例が LiuらのRankIQAで、FR-IQA指標を教師信号として活用する巧妙な手法です。まず高品質画像に対し強弱さまざまな人工的歪み（ブロックノイズ、ぼかし等）を加えて大量の画像ペアを生成し、どちらが高画質かというランク（相対的優劣）を自動的にラベル付けします。この大量のデータでSiameseネットワーク（2つの入力画像を比較する二重構造）を訓練し、その後得られた重みを単一画像入力のCNNに知識蒸留することで、絶対品質スコアを予測するモデルへとファインチューニングします。RankIQAはTID2013データセットで当時のSOTAを5%以上更新し、LIVEデータセットにおいては参照ありIQAの性能さえ上回る品質予測精度を達成しました。この成果は「FR-IQAが教師となりNR-IQAモデルを育てる」新奇なアプローチであり、以降も歪み画像のデータ拡張や合成データによる事前学習は盛んに研究されています。例えば最近では、未知の実世界歪みに強いモデルを得るため主観スコアなしで大規模合成データ上でコントラスト学習を行い（劣化の程度を区別する自己教師）、その特徴表現を本番のNR-IQAタスクに活かすといった手法も登場しています。

代表的論文: Xialei Liu et al., “RankIQA: Learning from Rankings for No-Reference Image Quality Assessment,” ICCV 2017 ;  Jinsong Shi et al., “Transformer-based NR-IQA via Supervised Contrastive Learning,” AAAI 2024（SaTQA: 合成データでコントラスト学習を行い未知歪みに対応）。

歪み多様性への対応：マルチブランチ・歪み識別型モデル

実環境での画像劣化は多種多様で、従来のNR-IQA手法は個別の歪み種類（ブロックノイズ専用、ぼかし専用など）に特化したものも多く見られました。深層学習時代にも、複数の歪み種にまたがる汎用モデルを目指す一方、歪みの種類ごとに特徴が異なる点に着目したアーキテクチャが提案されています。その一つがWeixia ZhangらのDBCNN (Deep Bilinear CNN)です。DBCNNは2つのCNNサブネットワークから構成されており、一方は合成歪みに特化して大規模訓練（歪みのタイプや強度を分類する事前学習）、もう一方はImageNet分類で学習済みのCNNを流用して自然画像の内容特徴を抽出します。この2系列の特徴マップをビリニアプーリングで統合し品質を推定することで、人工的な歪みにも撮影画像由来の自然な劣化（照明ノイズやピンぼけなど）にも単一モデルで対処できるようにしました。実際、DBCNNは合成歪みデータセットと実写歪みデータセットの双方で最先端の精度を示し、公開されていない画像に対しても高い一般化性能を確認しています。このアプローチは内容理解（画像内容の違いに頑健な特徴）と劣化検出（歪み種に敏感な特徴）の両面を組み込んだ点が画期的であり、特に監視カメラ映像や医療画像など、多様な品質劣化要因が混在する状況で有用です。また、派生研究としてはまず画像から歪みの種類を分類器で推定し、その情報を用いて品質スコア推定を行う二段階法や、チャネル注意機構で劣化成分に着目する手法など、歪み多様性への対応を強化する方向性が見られます。

代表的論文: Weixia Zhang et al., “Blind Image Quality Assessment Using a Deep Bilinear CNN,” IEEE TCSVT, 30(1):36-47, 2020 (DBCNN);  Kede Ma et al., “End-to-end Blind Image Quality Assessment with Content Understanding and Multi-distortion Adaptation,” IEEE TIP, 29:4058-4070, 2020 など。

大規模実写データセット活用による汎用モデル

ディープラーニングの性能はデータによって左右されるため、IQA分野でも大規模データセットの構築が精度向上を大きく後押ししました。近年、学術界・産業界から高解像度かつ様々な内容・歪みを含む写真に対し平均意見評点（MOS）を付与したデータセットが複数公開されています。例えばKonIQ-10kはYFCC100Mから多様な写真1万枚を抽出してMOSを収集したもので、同じくモバイル写真1万枚規模のSPAQ, 人工歪みと人間評価12万件を対で集めたPaQ-2-PiQ（約4万枚）などがあります。こうした大規模データにより、従来より深いCNNモデル（ResNetやInception系列など）を一から学習したり、全画像を縮小せず高精細のまま扱う手法が現実的になりました。HosuらはKonIQ-10kを用いてEfficientNetベースのモデルを学習し高い汎化性能を示したほか、Yingらは複数データセットを混ぜて学習することで照明歪みなど新奇な劣化への頑健性を高めています。またGoogleの提案したMUSIQ (Multi-scale Image Quality Transformer)は画像を複数解像度にスケーリングしたパッチ系列をTransformerに入力することで、画像全体の構図や細部の両方を捉える工夫をしました。この手法では従来CNNでは難しかったアスペクト比や解像度の異なる画像間での一括学習が可能となり、KonIQ-10kやSPAQ等の実写データセットで当時のSOTAを達成しています。実験的にもマルチスケールでの特徴抽出が実世界画像の品質評価に有効であることが報告されており、人間の視覚がマルチスケール（全体の印象と細部の両方）で品質判断する性質に合致する結果となっています。総じて、大規模データと高度なモデル（深層CNNやTransformer）の組み合わせにより、汎用NR-IQAモデルの実用性が飛躍的に高まってきました。これは、カメラで撮影された写真の画質自動チェックやプリンター出力の品質管理といったCanonの実応用において、従来は困難だった様々な状況に対する一貫した画質評価を可能にしつつあります。

代表的論文: Hosu et al., “KonIQ-10k: An Ecologically Valid Database for Deep Learning of Blind IQA,” IEEE TIP, 29:4041-4056, 2020 (大規模実画像IQAデータセット);  Junjie Ke et al., “MUSIQ: Multi-Scale Image Quality Transformer,” ICCV 2021;  Yuling Pan et al., “A Survey on Blind Image Quality Assessment in the Age of Deep Learning,” Frontiers in AI, 2022（深層学習IQAの総説）。

ビジョントランスフォーマー・高度な学習戦略による最先端NR-IQA

最後に、現在の最新トレンドとしてVision Transformerの活用や高度な学習戦略を組み合わせたNR-IQAを紹介します。前述のMUSIQに加え、Gu *et al.*のMANIQAはVision Transformerに基づくグローバル特徴抽出と多次元の注意機構でGAN生成画像の歪みに対して頑健な評価を実現しました。またShiらのSaTQAではTransformerモデルに教師ありコントラスト学習を組み込み、まず大規模合成データで主観スコア不要の事前学習を行って劣化パターンを学習し、その後得られた劣化特徴表現を本来のNR-IQAタスクに活かすという二段階学習を採用しています。SaTQAは7つの標準データセットで従来法を上回る性能を示し、特に未知の実写歪みに対する適応能力が向上しています。これらの研究は、モデルの表現力と汎化力を一層高める方向であり、今後は映像のIQA（Video QA）やタスク指向のIQA（例：医療診断に足る画質かどうか判定するモデル）への発展も期待されます。実務的には、Transformerを用いたモデルは計算資源こそ必要なものの、カメラや監視システムから得られる膨大な画像ストリームの中で品質劣化を自動検知しアラートを出す、といった高度な運用が可能になるでしょう。

代表的論文: Lin Ma et al., “Biological Vision Inspired Blind Quality Assessment for Camera Systems,” CVPR 2022 (視覚皮質モデル＋Transformer) ;  J. You, J. Korhonen, “Transformer for Image Quality Assessment,” arXiv:2107.10834, 2021 (ViTによるIQA) ;  Shuwei Jiang et al., “No-Reference IQA via Multi-Task and Contrastive Learning,” CVPR 2022 など。


---

以上、ディープラーニングを活用したIQA手法をFRとNRに分けて主要な流派を解説しました。それぞれ学術的に興味深い発展（例：知覚的類似度学習やTransformer導入）と実用上の意義（例：リアルタイム品質モニタリングや専門領域への適用）を持ち合わせており、カメラ・医療・監視・印刷といった分野での品質評価に寄与し得ます。今後もデータセットの拡充とモデルの進化により、人間の感じる「画質」を定量化する精度と汎用性は一層高まっていくでしょう。



作成日時: 2025-09-09 08:33:30