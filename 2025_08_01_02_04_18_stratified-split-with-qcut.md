# 連続値の層化分割テクニック

import pandas as pd
from sklearn.model_selection import train_test_split, StratifiedKFold

y_bins = pd.qcut(y, q=10, labels=False)        # 10ビン例
X_tr, X_tmp, y_tr, y_tmp = train_test_split(
    X, y, test_size=0.3, stratify=y_bins, random_state=42)

# val / test も同じノウハウで分割

作成日時: 2025-08-01 11:04:11