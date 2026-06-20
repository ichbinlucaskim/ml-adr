# Training Pipeline

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Multiclass Classification)                        │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    Predict 1 of K classes, decide metric (accuracy/macro-F1)
    ※ distinguish multiclass (only one) vs multilabel (several at once) ★
                │
                ▼
[1] Data Collection
    raw data, label y ∈ {0,1,...,K-1}, candidate features
                │
                ▼
[2] Data Split
    Train / Validation / Test  (prevent leakage)
    → Stratified split ★important★
      (keep class ratios identical across train/val/test)
                │
                ▼
[3] Preprocessing / Feature Engineering
    - missing values/outliers
    - X encoding, scaling (fit on train, transform val·test only)
    - ★decide label representation★
      · integer labels as-is → pairs with CrossEntropyLoss
      · one-hot vectors      → when using softmax+log directly
    - handle class imbalance ★
      (class weight, focal loss, resampling)
                │
                ▼
[4] Architecture / Model Definition
    - model choice (MLP, CNN, tree, GNN...)
    - Weight Initialization (Xavier/He)
    - ★output nodes = K★ (binary had 1, here one per class)
                │
                ▼
┌────────────────── Training Loop (repeat per epoch) ──────────────────┐
│                                                                      │
│  [5] Forward (FW)                                                    │
│      input → model → logits (K-dim raw score vector) ★               │
│              │                                                       │
│              ▼                                                       │
│  [6] Activation (output) ★sigmoid → softmax★                         │
│      softmax : K scores → probability distribution summing to 1      │
│      [2.1, 0.3, -1.0] → [0.78, 0.15, 0.07]                           │
│              │                                                       │
│              ▼                                                       │
│  [7] Loss Computation ★BCE → Cross-Entropy★                          │
│      CE = -log(probability) of the correct class                     │
│      · integer labels → CrossEntropyLoss                             │
│        (softmax is built in → some implementations skip [6])         │
│      · one-hot        → categorical CE                               │
│      + Regularization (L2, dropout)                                  │
│      + (if imbalanced) class weight / focal                          │
│              │                                                       │
│              ▼                                                       │
│  [8] Backward (Back)  → compute gradients                            │
│              │                                                       │
│              ▼                                                       │
│  [9] Optimizer step  → update W (lr)                                 │
│              │                                                       │
│              ▼                                                       │
│  [10] Validation (every epoch)                                       │
│       monitor val loss / macro-F1                                    │
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
[12] Prediction Decision ★threshold → argmax★
     pick the class with the largest of the K probabilities
     argmax([0.78,0.15,0.07]) → class 0
     ※ not a single cut (0.5) but "the position of the maximum"
     (can vary for top-k, cost-sensitive, or reject options)
                │
                ▼
[13] Final Evaluation (Test set, once only)
     Accuracy
     Precision/Recall/F1 → ★distinguish macro / micro / weighted★
       · macro   : simple average per class (minority classes treated equally)
       · weighted: weighted by class size
       · micro   : based on all samples
     K×K Confusion Matrix (which classes get confused with each other)
                │
                ▼
[14] Deployment / Monitoring
     deploy → inference → monitor drift → retrain
```     