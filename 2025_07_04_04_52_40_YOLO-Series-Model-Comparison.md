# YOLO各バージョン技術比較表

YOLOシリーズモデル比較

以下の表は YOLOv4 以降の各バージョン（YOLOv4, v5, v6, v7, v8）における主な技術的特徴と性能をまとめたものです。リリース時期と開発元、バックボーン・ネック構造、損失関数や学習手法、精度（mAP）・速度（FPS）、実装フレームワーク、新技術の採用例などを示しています。各情報は論文や公式ドキュメントを参照しています。

項目	YOLOv4	YOLOv5	YOLOv6	YOLOv7	YOLOv8

リリース/開発元	2020年4月 (Alexey Bochkovskiy ら)	2020年6月頃 (Ultralytics, Glenn Jocher)	2022年9月 (Meituan)	2022年7月 (Chien-Yao Wang ら, Academia Sinica)	2023年1月 (Ultralytics)
バックボーン	CSPDarknet53 (改良版Darknet53)	CSPDarknet53	EfficientRep (RepVGG系、小型モデルはRepConvスタック)	E-ELAN (Extended ELAN)	CSPベース (CSPDarknet系)
ネック	PANet + SPP (追加SPPモジュール)	PANet + SPPF（高速版SPP）	Rep-PAN (RepConvブロックを用いたPAN)	SPPCSPC + PAN (拡張型SPP+PAN)	FPN+PAN（ハイブリッドNeck）
ヘッド/損失・学習手法	CIoU損失、Mish活性化、Mosaic増強、DropBlock、Bag-of-Freebies/Specials	CIoU損失、AutoAnchor、6×6 Focus層省略、ハイパーパラ最適化（ホットスタート）	アンカーなし、VariFocal Loss、セルフディスティレーション、Anchor-Aided Training(AAT)戦略	アンカーあり（従来型）だがダイナミックラベル割当の最適化、新規のBoF（再パラメータ化訓練、細分化ラベル付与など）	アンカーなし、Focal Loss 採用、PyTorch移行、データ増強強化
精度 (mAP)／速度 (FPS)	COCO val: AP50: 65.7% (AP0.5-0.95: 43.5%) ＠65 FPS (V100)	(例) v5s: 37.4% mAP (AP0.5-0.95), v5l: 49.0%。v5s≈156FPS (V100)	v6-N: 37.5%＠1187FPS (T4)、v6-S: 45.0%＠484FPS、v6-M: 50.0%＠226FPS、v6-L: 52.8%＠116FPS	v7-E6: AP0.5-0.95=55.9%＠56FPS (V100)、最高56.8% AP	v8n: 37.3% mAP @ ≈1000FPS (A100)。上位モデルでは精度向上
実装/フレームワーク	Darknet(C)	PyTorch (ultralytics/yolov5)	PyTorch (Meituan/YOLOv6)	PyTorch (WongKinYiu/yolov7)	PyTorch (Ultralytics パッケージ)
採用技術例	CSPNet, PANet, SPP, Mish, Mosaic, DropBlock	CSPNet, PANet, SPPF, Focus層省略, CIoU	RepVGG (再パラメータ化), BiC (双方向連結モジュール), AAT, Self-Distill	E-ELAN, 拡張SPP (SPPCSPC), 再パラメータ化, ダイナミックラベル割当	CSPNet, FPN/PA ネック, アンカーなし検出, ハイパラ最適化 (Focal Loss等)


表に示したように、YOLOシリーズではバージョンごとにバックボーンやネック構造、検出ヘッド、損失関数、学習技術が進化してきました。例えば、YOLOv4 は CSPDarknet53 をバックボーンに PANet＋SPP を採用し、Mosaic や CIoU 損失などを取り入れて43.5% mAP (COCO) を65 FPSで実現しました。YOLOv5 では PyTorch 実装に移行し、Focus層削除＋6×6 Conv 追加、SPPF、高速化機構、ハイパーパラメータ最適化（ウォームアップなど）を導入。これにより YOLOv5x では約50.7% mAPまで向上しています。YOLOv6 は Meituan が提案した工業用途向けモデルで、RepVGGベースの再パラメータ化バックボーンと BiC モジュール、アンカー補助訓練やセルフ・ディスティレーションを取り入れ、軽量モデルで37.5% mAP@1187FPS(T4)を達成します。YOLOv7 では E-ELAN と呼ぶ拡張型 ELAN ネットワークや改良SPP (SPPCSPC)、動的ラベル割当など訓練上の改良を加え、56.8% mAP(COCO)に達しました。最新の YOLOv8 は Ultralytics によるアンカーなしモデルで、CSPベースのバックボーンとFPN+PANネックを採用し、PyTorchパッケージとして提供されます。YOLOv8 の最小モデル（n）では37.3% mAPをA100上で約1ms以下（≈1000FPS）で達成し、リアルタイム性と使いやすさを両立しています。

図：MS COCOデータセット上でのYOLOv4 (緑) と YOLOv3 (青) などの検出性能比較。YOLOv4 は YOLOv3 に対しAPとFPSの両面で約10%向上している。図は YOLOv4 論文より引用。

参考資料: 各モデルの公式論文や Ultralytics 公式ドキュメントよりなど。各種性能値は COCO ベンチマーク結果に基づく。



作成日時: 2025-07-04 13:52:36