# Training Pipeline

- Marking with ★ where it diverges from multiclass, since that's the closest cousin.

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Multilabel Classification)                        │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    Each sample can have SEVERAL labels at once (0 to K labels) ★
    ※ multiclass = exactly one; multilabel = independent yes/no per label
    e.g., one photo → {dog, grass, frisbee}; one article → {politics, economy}
                │
                ▼
[1] Data Collection
    raw data, label y = K-dim binary vector ★ e.g., [1,0,1,0,1]
    (each position is an independent 0/1)
                │
                ▼
[2] Data Split
    Train / Validation / Test  (prevent leakage)
    → Iterative/multilabel stratification ★
      (single-label stratify breaks with label combinations;
       keep each label's positive ratio balanced across splits)
                │
                ▼
[3] Preprocessing / Feature Engineering
    - missing values/outliers
    - X encoding, scaling (fit on train, transform val·test only)
    - ★label representation: multi-hot binary matrix (N × K)★
    - per-label imbalance ★ (each label has its own rare/common rate
      → pos_weight per label, not a single class weight)
                │
                ▼
[4] Architecture / Model Definition
    - model choice (MLP, CNN, Transformer, GNN...)
    - Weight Initialization (Xavier/He)
    - ★output nodes = K, but each is an INDEPENDENT head★
      (same node count as multiclass, but they don't compete)
                │
                ▼
┌────────────────── Training Loop (repeat per epoch) ──────────────────┐
│                                                                      │
│  [5] Forward (FW)                                                    │
│      input → model → logits (K-dim raw score vector)                 │
│              │                                                       │
│              ▼                                                       │
│  [6] Activation (output) ★softmax → sigmoid (per node)★              │
│      sigmoid on EACH of the K nodes independently                    │
│      → K independent probabilities, sum need NOT be 1                │
│      [2.1, 0.3, -1.0] → [0.89, 0.57, 0.27]  (each its own 0~1)       │
│              │                                                       │
│              ▼                                                       │
│  [7] Loss Computation ★Cross-Entropy → BCE per label, summed★        │
│      BCEWithLogits over K outputs (= K binary problems at once)      │
│      · effectively: average of K independent BCE terms               │
│      + Regularization (L2, dropout)                                  │
│      + (if imbalanced) pos_weight per label / focal loss             │
│              │                                                       │
│              ▼                                                       │
│  [8] Backward (Back)  → compute gradients                            │
│              │                                                       │
│              ▼                                                       │
│  [9] Optimizer step  → update W (lr)                                 │
│              │                                                       │
│              ▼                                                       │
│  [10] Validation (every epoch)                                       │
│       monitor val loss / macro-F1 / mAP ★                            │
│       → Early Stopping / checkpoint                                  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                │
                ▼
[11] Hyperparameter Tuning
     lr, number of layers, dropout, regularization strength, batch size...
     → repeat based on val
                │
                ▼
[12] Prediction Decision ★argmax → per-label threshold★
     apply a threshold to EACH of the K probabilities independently ★
     [0.89, 0.57, 0.27] with cut 0.5 → labels {0, 1} on, {2} off
     ※ NOT argmax (would force exactly one) — each label gated separately
     ※ thresholds can be tuned per-label, not one global 0.5
                │
                ▼
[13] Final Evaluation (Test set, once only)
     ★different metric family★
     - per-label Precision/Recall/F1
     - macro / micro / weighted F1 (averaging across labels)
     - Hamming loss (fraction of wrong label slots) ★
     - subset accuracy (entire label set exactly right — strict) ★
     - mAP (mean Average Precision across labels)
     (no single K×K confusion matrix — one 2×2 per label instead)
                │
                ▼
[14] Deployment / Monitoring
     deploy → inference → monitor drift → retrain
```