# Problem name

| | |
|---|---|
| Path | `unsupervised / <task> / <model> / <this-problem>` |
| Summary | One sentence: what structure is being sought. |
| Status | Exploring / In progress / Done / Shelved |
| Date | YYYY-MM |

## 1. Problem

- **Goal.** Clustering, dimensionality reduction, anomaly detection, or generative. What the output represents (group ID, compressed vector, anomaly score, sample).
- **Absence of labels.** Whether labels are genuinely unavailable, prohibitively expensive, or simply undefined.
- **Success criteria.** No ground truth exists, so define in advance what makes the result useful: interpretable segments, clean separation, anomalies correctly surfaced.
- **Decision supported.** What the discovered structure is used for.

## 2. Data

| Field | Detail |
|-------|--------|
| Input features | X only; no target. |
| Size | |
| Assignment of new points | If the model must generalize, how unseen points are handled. |
| Outliers | Handling, given their influence on clusters and variance. |

Preprocessing that materially affected results. Note that feature scaling and the distance metric are decisive for distance- and variance-based methods; record whether dimensionality reduction was applied first.

## 3. Method and key decisions

- **Method.** K-means, DBSCAN, GMM, PCA, autoencoder, Isolation Forest, etc.
- **Rationale.** Cluster geometry, whether K must be set in advance, outlier handling, linear vs nonlinear, ability to transform new points.
- **Primary hyperparameter.** K (clusters), target dimension d, contamination rate, or epsilon.
- **Fitting procedure.** Centroid iteration, EM, eigendecomposition, or reconstruction.

| Decision | Alternatives | Selected | Rationale |
|----------|-------------|----------|-----------|
| | | | |

## 4. Fitting

| Field | Detail |
|-------|--------|
| Objective | Within-cluster variance, modularity, reconstruction error, or none. |
| Hyperparameter selection | How K / d / threshold was chosen (elbow, scree plot, contamination estimate). |
| Stability notes | Scaling sensitivity, initialization seeds, convergence. |

## 5. Evaluation

Accuracy and F1 do not apply. Use intrinsic measures, and external measures only where partial labels exist.

| Measure | Value | Note |
|---------|-------|------|
| Internal (silhouette, Davies-Bouldin, explained variance, reconstruction error) | | |
| External (ARI, NMI, or PR-AUC / recall for anomaly detection) | | |

The primary deliverable is interpretation: profile each cluster, inspect the embedding, or examine flagged anomalies.

## 6. Retrospective

- **What worked.**
- **What to change next time.**
- **Stability.** Whether the structure holds across seeds and subsamples, or appears to be an artifact.
- **Open questions and next steps.**

## Assets

Embedding plots, dendrograms, cluster profiles: `./assets/`.