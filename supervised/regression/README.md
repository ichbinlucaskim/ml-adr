# Training Pipeline

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Regression)                                       │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    Predict continuous value (y), decide metric (RMSE/MAE/R²)
    └─ task is regression → output·loss·metric set follows
                │
                ▼
[1] Data Collection
    raw data, target y (continuous), candidate features
                │
                ▼
[2] Data Split
    Train / Validation / Test  (prevent leakage)
    → split first, fit on train only
    ※ if y distribution is skewed, take care when splitting ★
                │
                ▼
[3] Preprocessing / Feature Engineering
    - missing values/outliers ★outliers heavily affect MSE★
    - encoding (one-hot, embedding)
    - scaling (standardize X) ← fit on train, transform val·test only
    - target transformation ★easily missed★
      (log(y), Box-Cox, etc. — when y has a long one-sided tail)
    - feature creation/selection
                │
                ▼
[4] Architecture / Model Definition
    - model choice (Linear/Ridge/Lasso, MLP, tree, GNN...)
    - Weight Initialization (Xavier/He)
    - ★output layer: 1 node, no activation (linear)★
      (the sigmoid slot from binary is empty = raw value output as-is)
                │
                ▼
┌────────────────── Training Loop (repeat per epoch) ──────────────────┐
│                                                                      │
│  [5] Forward (FW)                                                    │
│      input → model → ŷ (predicted continuous value, no transform) ★  │
│              │                                                       │
│              ▼                                                       │
│  [6] Loss Computation ★the key difference★                           │
│      MSE  : penalizes large errors heavily, sensitive to outliers    │
│      MAE  : robust to outliers, targets the median                   │
│      Huber: compromise between MSE+MAE (based on δ)                  │
│      + Regularization (L2=Ridge, L1=Lasso)                           │
│              │                                                       │
│              ▼                                                       │
│  [7] Backward (Back)                                                 │
│      compute gradients (backprop)                                    │
│              │                                                       │
│              ▼                                                       │
│  [8] Optimizer step                                                  │
│      SGD/Adam → update W (learning rate)                             │
│              │                                                       │
│              ▼                                                       │
│  [9] Validation (every epoch)                                        │
│      monitor val RMSE/MAE                                            │
│      → Early Stopping / checkpoint                                   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                │
                ▼
[10] Hyperparameter Tuning
     lr, number of layers, regularization strength (α), batch size...
     → repeat based on val
                │
                ▼
[11] ★No Threshold step★
     binary's [12] threshold disappears
     (no probability→0/1 conversion needed, prediction is the answer)
     instead: if target was transformed → inverse transform required ★
            if ŷ was log(y), revert with exp(ŷ) to get real units
                │
                ▼
[12] Final Evaluation (Test set, once only)
     RMSE, MAE, R² (explanatory power), MAPE (% error)
     + residual analysis (residual plot): check for leftover patterns ★
                │
                ▼
[13] Deployment / Monitoring
     deploy → inference → monitor data/target drift → retrain
```