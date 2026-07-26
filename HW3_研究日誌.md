# ML2021 HW3 研究日誌

## 作業目標

使用 CNN 將 Food-11 圖片分成 11 個類別，並以 validation 與 Kaggle
成績判斷模型的泛化能力。

資料量：

- Training labeled：3,080 張
- Validation：660 張
- Testing：3,347 張

## 實驗結果

| 實驗 | Epoch | 最佳 Valid Acc | Kaggle Private | Kaggle Public |
|---|---:|---:|---:|---:|
| 原始 baseline | 20 | 47.88% | 45.965% | 48.387% |
| Data augmentation | 20 | 51.52% | 49.312% | 48.924% |
| Augmentation + Dropout 0.3 | 20 | 45.61% | 43.335% | 43.966% |
| Augmentation + Dropout 0.3 | 40 | 53.03% | 47.818% | 47.311% |
| Augmentation + Dropout 0.1 | 40 | 55.76% | 52.779% | 51.911% |
| Augmentation + Dropout 0.1 + Scheduler | 40 | **56.21%** | **56.066%** | **55.794%** |

## 目前最佳設定

```text
Data augmentation：開啟
Dropout：0.1
Optimizer：Adam
Initial learning rate：3e-4
Weight decay：1e-5
Epoch：40
Scheduler：ReduceLROnPlateau
Scheduler factor：0.5
Scheduler patience：3
Minimum learning rate：1e-6
最佳 checkpoint：Epoch 39
```

Scheduler 實際降低 learning rate 的過程：

```text
3.00e-4
1.50e-4
7.50e-5
3.75e-5
1.87e-5
```

相較於相同設定但沒有 Scheduler：

- Private score：52.779% → 56.066%，提升 3.287 個百分點。
- Public score：51.911% → 55.794%，提升 3.883 個百分點。

## 目前觀察

1. 原始 CNN 在少量資料上很快 overfit。
2. Data augmentation 能改善泛化能力。
3. Dropout 0.3 對此模型太強，造成學習速度過慢；0.1 較平衡。
4. 固定 learning rate 使後期 validation 表現波動。
5. 動態降低 learning rate 後，Kaggle Private 與 Public 都明顯提升。
6. 最後一輪 train accuracy 為 79.12%，valid accuracy 為 56.06%，
   仍存在約 23 個百分點的差距，因此 overfitting 尚未完全解決。

## 程式輸出

- 每次實驗保存 validation accuracy 最佳的 checkpoint。
- 訓練完成後以最佳 checkpoint 產生 Kaggle CSV。
- 每個 epoch 更新 `training_curves.png`。
- 曲線圖使用固定檔名覆蓋，避免累積大量圖片。
- 曲線圖同時顯示 train/valid loss、accuracy 與最佳 epoch。

## 下一步

保持目前最佳設定，只將 `weight_decay` 從 `1e-5` 調整為 `1e-4`，
測試較強的權重正則化能否縮小 train/validation 差距。一次只改一個
變因，避免無法判斷改善來源。
