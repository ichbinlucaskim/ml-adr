# Training Pipeline

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Binary Classification)                            │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    Decide task type (binary/multi/regression), metric, success criteria
    └─ loss · metric · threshold direction is already set here
                │
                ▼
[1] Data Collection
    raw data, labels (y), candidate features
                │
                ▼
[2] Data Split  ★often skipped★
    Train / Validation / Test  (prevent leakage)
    → split "before preprocessing" (fit on train only)
                │
                ▼
[3] Preprocessing / Feature Engineering
    - missing values, outliers
    - encoding (one-hot, embedding)
    - scaling (normalize/standardize) ← fit on train, transform val·test only
    - feature creation/selection
    - (if imbalanced) resampling / class weight
                │
                ▼
[4] Architecture / Model Definition
    - model choice (GCN, MLP, tree, ...)
    - layer structure, activation
    - Weight Initialization ★missing★ (Xavier/He, etc.)
                │
                ▼
┌────────────────── Training Loop (repeat per epoch) ──────────────────┐
│                                                                      │
│  [5] Forward (FW)                                                    │
│      input → model → raw score (logit)                               │
│              │                                                       │
│              ▼                                                       │
│  [6] Activation (output)                                             │
│      sigmoid → 0~1 probability  (binary)                             │
│              │                                                       │
│              ▼                                                       │
│  [7] Loss Computation                                                │
│      BCE (binary) / CE (multi) / MSE (reg)                           │
│      + Regularization (L2, dropout, etc.) ★missing★                  │
│              │                                                       │
│              ▼                                                       │
│  [8] Backward (Back)                                                 │
│      compute gradients (backprop)                                    │
│              │                                                       │
│              ▼                                                       │
│  [9] Optimizer step ★missing★                                        │
│      SGD/Adam → update W (learning rate)                             │
│              │                                                       │
│              ▼                                                       │
│  [10] Validation (every epoch)                                       │
│       monitor val loss/metric                                        │
│       → Early Stopping / checkpoint                                  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                │
                ▼
[11] Hyperparameter Tuning ★missing★
     lr, number of layers, dropout, batch size...
     → repeat [4]~[10] based on val
                │
                ▼
[12] Threshold Selection
     0~1 probability → 0/1
     → decide cut by val/PR curve, F1·cost (not fixed at 0.5)
                │
                ▼
[13] Final Evaluation (Test set, once only)
     Accuracy, Precision/Recall, F1, ROC-AUC, PR-AUC,
     Confusion Matrix
                │
                ▼
[14] Deployment / Monitoring ★missing★
     deploy → inference → monitor data drift → retrain
```