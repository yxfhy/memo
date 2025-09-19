# 多次元IQA年表とSPAQ・PARAの潮流

はい、ご主人さま。
2019年からの深層学習IQAの年表（データセット／モデル中心、MDIQA背景向け）。PARAとSPAQを必須で明記しています。

2019

データセット

KonIQ-10k：10,073枚、クラウドソーシングで約120万評価。実運用環境に近い「in-the-wild」品質データセットの代表。あわせて提案モデル KonCept512 も提示。 

KADID-10k：81参照×25歪み×5段階＝10,125枚（人工歪み）。未ラベルのKADIS-700kも併設。 

PaQ-2-PiQ（preprint）：約4万枚の実画像＋12万パッチ、約393万の主観評価。パッチから画像へ品質を写像する大規模基盤。CVPR 2020採択。 


モデル・方式

KonCept512（InceptionResNet系を高解像度学習）。KonIQ-10kでのベースライン。 



2020

データセット

【必須】SPAQ（Smartphone Photography Attribute & Quality）：11,125枚・66機種、MOSに加えて画質（品質）＋5属性（明るさ・彩度・コントラスト・ノイズ感・シャープネス）、シーンカテゴリ、EXIFも付与。スマホ写真に特化した**多次元（属性）**アノテーションが特徴。 

PIPAL：GAN系復元を含む29k歪み画像／250参照／40歪み種類、Eloで113万比較評価。復元系の知覚指標検証に用いられる大型FR/NR評価基盤。 


モデル・方式

HyperIQA（CVPR’20）：自己適応ハイパーネットでin-the-wild NR-IQAを強化。 



2021

データセット

FLIVE：約39,810枚の大規模in-the-wild NR-IQA。以後のBIQA研究の標準ベンチの一つに。 


モデル・方式

MUSIQ（ICCV’21）：マルチスケールTransformerで任意解像度を直接扱うNR-IQA。 



2022

データセット（美学）

【必須】PARA（Personalized Image Aesthetics with Rich Attributes, CVPR’22）：31,220枚／438被験者、画像向け9属性＋人（主観者）向け4属性やパーソナリティ情報まで付与。個人化審美の多属性データで、**MDIQAの「審美側ディメンション」**の参照に有用。 


モデル・方式

MANIQA（CVPRW’22, NTIRE優勝）：GAN歪みに強い多次元アテンションNR-IQA。 



2023

データセット

AGIQA-3K（生成画像IQA）：AIGC画像の品質・テキスト整合性を評価する初期の代表的ベンチ。 


モデル・方式

Re-IQA（CVPR’23）：教師なしで高・低レベル表現を対比学習しNR-IQAを汎化。 

LIQE / Vision-Language系BIQA（CVPR’23）：視覚と言語の対応を活用した多タスクBIQA。 



2024

データセット

DepictQA-Wild（2024）：VLMを用い、説明・比較・FR/NR横断の多機能IQAタスクを定義。 

UHD-1（“Pushing the Boundaries…”）：4K中心・高美学寄りのNR-IQAデータ（6,073枚）。 


モデル・方式

TOPIQ（TIP’24）：セマンティクス→局所歪みのトップダウン指向（FR/NR対応）。 

SaTQA（AAAI’24）：SCL＋TransformerでNR-IQAを強化。 

DP-IQA（2024）：拡散モデル事前知識を背骨にしたNR-IQA。 



2025

データセット

DiffIQA（~18万枚）：参照が必ずしも完璧でない状況を想定したFR-IQAの新基盤。併せてSRIQA-Bench（SR特化）も構築。 


モデル・方式

A-FINE（CVPR’25）：忠実度（Fidelity）×自然さ（Naturalness）を適応的に結合する一般化FR-IQA。DiffIQA/SRIQA-Benchで検証。 

MDIQA（arXiv 2025/08）：技術5＋審美4＝計9次元で品質を多次元的に学習・統合し、重み可変で復元（IR）訓練の指標にも転用。SPAQ/PARA系の「属性・多次元」潮流の延長線に位置づく。 




---

MDIQAの背景としての要点（SPAQ・PARAを核に）

SPAQ（2020）は「技術的属性（明るさ/彩度/コントラスト/ノイズ/シャープネス）＋EXIF＋シーン」という多軸ラベルを、スマホ写真という現実的ドメインで整備。技術ディメンションの監督信号として極めて有用。 

PARA（2022）は「審美（美学）側の多属性」と個人属性（被験者プロファイル）まで含むパーソナライズド審美データ。審美ディメンションの監督信号の供給源。 

MDIQA（2025）は、上記の流れを統合し、技術×審美の多次元表現を別枝で学習→統合して最終品質を予測。さらに各次元の重みを可変にし、復元タスクの訓練損失としても使える設計。 


> 参考：2019以降の代表的なベンチ（KonIQ-10k / KADID-10k / PaQ-2-PiQ / SPAQ / PIPAL / FLIVE / AGIQA-3K / DepictQA-Wild / DiffIQA）と、代表的モデル（HyperIQA / MUSIQ / MANIQA / Re-IQA / LIQE-VLM / TOPIQ / A-FINE / DP-IQA）が、MDIQAの多次元フレームを裏打ちしています。 



以上です。



作成日時: 2025-09-19 12:29:14