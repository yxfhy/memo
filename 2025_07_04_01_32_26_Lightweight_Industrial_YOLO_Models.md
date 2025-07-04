# 産業向け軽量高速物体検出モデル比較

1位：YOLOv6（Anchor-free, YOLOv6-N/S）

概要：Meituan（中国企業）発表の産業用に最適化したYOLO系モデル。モデルサイズを小さく抑えつつネットワーク構造や学習法を工夫し、高速化を極限まで追求しています。

速度・精度：YOLOv6-N（最小版）はCOCO mAP 37.5%で、NVIDIA Tesla T4 GPU上で約1187 FPSという驚異的な高速性を達成。やや大きいYOLOv6-Sでも45.0% mAPで484 FPSと、同規模の他モデルを上回ります。

モデル規模：YOLOv6-Nは約7.0Mパラメータ、YOLOv6-Sは約21.2Mで、ファイルサイズも10～30MB程度に収まります。

実績・利用例：論文名に「industrial applications」とある通り、工業検査や組み込み用途を念頭に開発されています。実運用例は少ないものの、高速カメラ検査ラインなどへの適用が想定されています。

実行環境：PyTorch実装でCPU推論も可能（ただしGPUより低速）。TensorRTやONNX化によるGPU高速化・量子化モデル（INT8）化にも対応し、Jetsonなどエッジデバイスへの展開も容易です。公式GitHubが公開されており、学習済み重みも提供されています。


2位：YOLOv7-tiny-SiLU（Anchor-based, YOLOv7系）

概要：YOLOv7の軽量版で、LeakyReLU版とSiLU版があるがここではSiLU版を紹介。元論文は「高速検出の王者」とされるYOLOv7で、tiny版はその極小モデルです。

速度・精度：COCO mAPは約38.7%。NVIDIA V100上で約286 FPSと非常に高速です。Micro UAVや製造ラインの欠陥検知などで十分実用的な精度を保ちつつ、計算量を抑えています。

モデル規模：パラメータ数約6.2M、ファイルサイズは10MB以下と非常にコンパクト。

実績・利用例：工場監視やエッジ検出のチュートリアル事例に登場するほか、台湾Pineapple検出や鉄鋼欠陥検出などの研究でも利用実績があります。

実行環境：PyTorch実装でONNX/TensorRT変換が容易なため、Jetson Nano／Xavierなどへのデプロイ実例があります。軽量のためCPU（ARM）上でも数十FPSが期待できます。学習済みモデルはUltralytics公式リポジトリで公開されています。


3位：YOLOv8 Nano（Anchor-free, Ultralytics YOLOv8n）

概要：最新YOLOシリーズYOLOv8の「Nano」版。Ultralytics社提供で、Python APIやCLIによる利用が簡便です。

速度・精度：COCO mAPは37.3%。NVIDIA A100上でTensorRT高速化時に約0.99 ms（≈1010 FPS）という報告があります。非常に高速で、リアルタイム性を重視する用途に向きます。

モデル規模：パラメータ数約3.2M、重量ファイルも5～10MB程度と超小型です。

実績・利用例：鉄鋼表面欠陥データ（NEU-DET）での検証例があり、YOLOv8をベースにした改良モデル（SLF-YOLO）がYOLOv8比で精度向上を示しています。Ultralyticsコミュニティでは汎用性の高い最新モデルとして広く利用されています。

実行環境：PyTorch実装でマルチバックエンド対応。ONNX/TensorRT変換でJetsonシリーズにもデプロイ可能です。CPUでも動作しますが、GPUやTensorRT活用で真価を発揮します。Ultralytics公式パッケージ（pip）で簡単に利用でき、学習済み重みも配布されています。


4位：PP-YOLOE-S（Anchor-based, PaddlePaddle YOLO系）

概要：Baidu（パドルディテクション）開発のYOLO派生モデル。手法はYOLO系ですが、「Pseudo-Label Boosted YOLO」（PP）シリーズの進化形で、軽量かつ高速な構造を持ちます。

速度・精度：COCO mAPは43.1%と高く、NVIDIA V100上で約208 FPSの高速推論が可能です。同規模のYOLO系モデルと比べて非常に高速でありながら、優れた精度を両立しています。

モデル規模：約7.9Mパラメータ、ネットワーク深度・幅のバランスが最適化されています。

実績・利用例：主にBaidu社内および中国圏で注目されているが、鉄道や製造ラインの異物検知での応用研究例があります（TensorRT化によるFP16推論で実測150FPS報告例あり）。オープンソースはPaddleDetection版で提供されており、推論速度重視の検出タスクで候補になります。

実行環境：PaddlePaddleフレームワークで実装。TensorRTやONNXなどを介してJetsonで動かす試みも可能ですが、主にGPU環境で使われます。学習済みモデルはPaddle公式Hubなどから入手可能です。


5位：EfficientDet-D0（BiFPN を用いた効率的モデル）

概要：Google AutoMLチーム提案のEfficientDetシリーズの最小モデル。EfficientNetバックボーン＋BiFPNによる複合スケーリング設計で、「最小限のパラメータ数で効率的な検出」を実現しています。

速度・精度：COCO mAPは34.6%と控えめですが、そのぶん推論は高速です。TensorRT最適化済みでTesla T4上およそ3.92ms（約255 FPS）との報告があります。PyTorch版やTAO Toolkit版での推論例では、Jetsonや軽量GPUでのリアルタイム実行が可能です。

モデル規模：約3.9Mパラメータ、Weightファイルも数MBで非常に小型です。メモリ・計算資源が極端に限られる環境に強みがあります。

実績・利用例：産業検査での直接的な事例は少ないものの、クラウドバッチ処理や組み込み用途で利用される例があります。欠点は小物検出精度が低めなことで、精度よりもモデル軽量性・推論効率が最優先される場面に適します。

実行環境：TensorFlow/TensorRTでの推論が一般的。NVIDIA TAO Toolkit（旧Transfer Learning Toolkit）でのサポートもあります。CPU上でも動作しますが、やはりGPU推論（TensorRT）前提で設計されたモデルです。


以上の5モデルは、軽量かつ高速であると同時に実用的な精度を持ち、産業欠陥検出のコストパフォーマンス（精度÷計算コスト）に優れています。などの参考文献で示される数値を踏まえ、総合的に評価しました。


作成日時: 2025-07-04 10:32:22