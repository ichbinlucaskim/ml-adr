# Training Pipeline

- Marked the structural breaks with ★. The biggest shock is that **there is no y, no loss-against-labels, and no "correct answer" to evaluate against.**

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Clustering — Unsupervised)                        │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    ★No labels y at all★ — goal: discover natural groups in the data
    Decide: what does "similar" mean here? what will groups be used for?
    e.g., customer segmentation, document grouping, anomaly grouping
                │
                ▼
[1] Data Collection
    raw data, features X only ★ (no target column)
                │
                ▼
[2] Data Split ★usually NO train/test split★
    Clustering typically fits on the whole dataset
    (no labels to "test" against → the classic split logic doesn't apply)
    ※ if building a reusable model (e.g. assign new points later),
       you may hold out data to check stability ★
                │
                ▼
[3] Preprocessing / Feature Engineering ★MOST critical step here★
    - missing values/outliers (outliers can hijack a whole cluster)
    - encoding categoricals
    - ★SCALING IS ESSENTIAL★ — distance-based, so unscaled features
      let large-range columns dominate (standardize/normalize)
    - ★distance metric choice★ (Euclidean / cosine / Manhattan)
    - dimensionality reduction (PCA/UMAP) often applied first
      (high dimensions make distances meaningless — "curse of dim.")
                │
                ▼
[4] Algorithm Choice ★replaces "architecture"★
    - K-means      : spherical, equal-ish size, must pick K
    - GMM          : soft/elliptical clusters (probabilistic)
    - DBSCAN       : density-based, finds arbitrary shapes,
                     auto-detects outliers, no K needed
    - Hierarchical : nested tree (dendrogram), no K upfront
    - (DL): Deep Embedded Clustering, autoencoder + cluster
    ★key hyperparameter is K (or ε/min_samples for DBSCAN)★
                │
                ▼
[5] Fit / Assign ★replaces the forward+backward+optimizer loop★
    The algorithm iterates, but NOT via backprop on a label loss:
    - K-means: alternate {assign points → recompute centroids}
               until centroids stop moving ★
    - GMM: EM algorithm (expectation ↔ maximization)
    - DBSCAN: expand clusters by density reachability (single pass-ish)
    → output: a cluster ID for each point  (e.g. 0,1,2,... or "noise")
                │
                ▼
[6] Choosing K / Validation ★no val-set; use internal criteria★
    Since there's no ground truth, judge cluster QUALITY intrinsically:
    - Elbow method   : plot within-cluster variance vs K, find the bend
    - Silhouette score: how tight-and-separated (−1~1, higher better) ★
    - Davies-Bouldin / Calinski-Harabasz indices
    → repeat [4]~[5] across K values, pick the best-scoring
                │
                ▼
[7] Evaluation ★two regimes★
    (a) Internal (no labels — the usual case):
        silhouette, cohesion vs separation, cluster stability
    (b) External (IF you happen to have some true labels):
        ARI (Adjusted Rand Index), NMI (Normalized Mutual Info) ★
        — measures agreement with known groups
    + ★Interpretation is the real deliverable★:
        profile each cluster ("cluster 2 = young high-spenders")
                │
                ▼
[8] Deployment / Monitoring
    deploy → assign new points to nearest cluster (K-means/GMM can;
             DBSCAN/hierarchical often need refit) ★
    → monitor drift: do the groups still hold? → periodic re-cluster
```

The essence in one line: **clustering replaces "labels → loss → backprop → metric" with "distance → group → intrinsic quality score."** Everything that made the supervised pipelines tick — the y column, BCE/CE/MSE, gradient descent on a label-loss, a held-out test set — either vanishes or gets replaced by a distance-based substitute.

Here's how clustering sits against the supervised family on the axes you've been tracking:

| Step | Supervised (any) | Clustering |
|------|-----------------|------------|
| Labels y | required | none ★ |
| Train/test split | essential | usually skipped ★ |
| Most critical prep | varies | scaling + distance metric ★ |
| "Architecture" | model + layers | algorithm choice (K-means, DBSCAN...) |
| Learning mechanism | forward → loss → backprop | iterative assign/update (often no backprop) ★ |
| Key hyperparameter | lr, layers... | K (number of clusters) ★ |
| "Loss" | BCE/CE/MSE vs labels | within-cluster variance / no label-loss ★ |
| Validation | val metric vs y | silhouette, elbow (intrinsic) ★ |
| Final answer | prediction vs truth | group IDs + human interpretation ★ |

Two traps worth burning in:

First, **scaling is not optional in clustering — it's make-or-break.** Because clusters are formed by distance, a feature measured in the thousands (e.g. income) will completely drown out one measured 0~1 (e.g. a ratio) unless you standardize. In supervised learning a tree could shrug this off; here it silently ruins the result.

Second, **there is no "right answer" to grade against, so K selection is the hard part.** You're not told there are 3 groups — you infer it from elbow/silhouette curves, and ultimately from whether the clusters are *interpretable and useful* to the business. The final deliverable isn't an accuracy number; it's a story like "we found four customer types, and here's what each one is."