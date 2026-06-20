# Training Pipeline

- Marked with ★ where it diverges from the supervised and other unsupervised pipelines.

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Anomaly Detection)                                │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    Goal: find points that deviate from "normal" ★
    ★Key question: how rare/labeled are the anomalies?★
    - no labels at all      → unsupervised (most common)
    - only normal labeled   → semi-supervised (one-class)
    - both labeled but rare → imbalanced supervised
    Decide: cost of false alarm vs missed anomaly (drives the threshold)
                │
                ▼
[1] Data Collection
    raw data X; anomalies are ★rare or absent in training★
    (you usually have lots of "normal," few/no "abnormal" examples)
                │
                ▼
[2] Data Split
    ★If unsupervised: fit on (mostly-normal) data, no label-based split★
    ※ If you have some labeled anomalies, keep them ONLY for the
       test/validation set to measure detection — never to train on ★
                │
                ▼
[3] Preprocessing
    - missing values
    - encoding categoricals
    - ★SCALING IS ESSENTIAL★ — most methods are distance/density based,
      so unscaled features distort what "far" or "rare" means
    - (often) dimensionality reduction first (high-dim hides anomalies)
                │
                ▼
[4] Method Choice ★replaces "architecture" — pick by data shape★
    ┌────────────────────────────────────────────────────────────────┐
    │  Anomaly Detection (goal: find outliers)                       │
    │      │                                                         │
    │      ├─ Approach 1: Density-Estimation based ★overlap point★   │
    │      │     estimate p(x) → low-p(x) (rare) point = anomaly     │
    │      │     e.g., GMM, KDE, Normalizing Flow                    │
    │      │                                                         │
    │      ├─ Approach 2: Distance / Density (neighbor) based        │
    │      │     point far from its neighbors = anomaly              │
    │      │     e.g., kNN, LOF (Local Outlier Factor), DBSCAN noise │
    │      │                                                         │
    │      ├─ Approach 3: Boundary based                             │
    │      │     learn a boundary enclosing the normal region →      │
    │      │     anything outside = anomaly                          │
    │      │     e.g., One-Class SVM, Isolation Forest               │
    │      │                                                         │
    │      └─ Approach 4: Reconstruction based (DL)                  │
    │            an Autoencoder trained on normal data fails to      │
    │            reconstruct anomalies → high error = anomaly        │
    │            e.g., Autoencoder, VAE                              │
    └────────────────────────────────────────────────────────────────┘
                │
                ▼
[5] Fit ★replaces forward+loss+backprop (method-dependent)★
    - density (GMM/KDE): fit the distribution on normal data
    - distance (LOF/kNN): index neighbors (often no "training" at all) ★
    - boundary (IsoForest/OC-SVM): learn the enclosing boundary
    - reconstruction (AE/VAE): THIS trains via forward → reconstruction
      loss (MSE) → backprop on normal data ★ (the DL-style member)
    → output per point: an ★anomaly SCORE★ (not a class yet)
                │
                ▼
[6] Threshold Selection ★score → normal/anomaly★
    turn the continuous anomaly score into a 0/1 decision
    - set cut by: expected contamination rate, PR curve,
      or business cost (false alarm vs miss) ★
    - this is the anomaly-detection analog of binary's threshold step
                │
                ▼
[7] Evaluation ★label-scarce, so metrics differ★
    - IF some true anomaly labels exist (in test only):
      ★Precision/Recall, F1, ROC-AUC, and especially PR-AUC★
      (PR-AUC matters most because anomalies are extremely imbalanced)
    - IF no labels: judge by inspection / known-case recall /
      whether flagged points are genuinely interesting ★
    ※ Accuracy is misleading here — predicting "all normal" can score
       99%+ while catching zero anomalies ★
                │
                ▼
[8] Deployment / Monitoring
    deploy → score incoming points in real time → alert above threshold
    ★critical: "normal" drifts over time★ — yesterday's normal may be
    today's anomaly (and vice versa) → recalibrate threshold / refit
    → watch the alert rate: sudden spikes = drift or a real incident
```

The essence in one line: **anomaly detection produces a *score of "how unusual,"* then a *threshold* turns it into a flag — it borrows binary classification's threshold step but trains almost entirely on "normal" because anomalies are too rare to learn directly.**

Two traps worth burning in:

First, **accuracy is a trap.** When anomalies are 0.1% of the data, a model that says "everything is normal" scores 99.9% accuracy while being completely useless. That's why **PR-AUC and recall** are the honest metrics — you care about *catching the rare positives*, not overall correctness.

Second, **the four approaches differ in what they need.** Reconstruction (Autoencoder/VAE) is the only one that trains like your earlier DL pipelines — forward, MSE loss, backprop on normal data. The other three (density, distance, boundary) are mostly classical and some (like LOF/kNN) barely "train" at all — they just index the data and score by neighborhood. So pick by your data: high-dimensional and lots of normal examples → reconstruction; small/tabular → Isolation Forest or LOF; well-behaved distribution → density-based.

That completes the unsupervised trio's "find structure" side (clustering, dim-reduction, anomaly detection). The remaining branch is **Generative** — which, as we noted, is density estimation's other child (model p(x), then *sample* from it instead of flagging rarity).