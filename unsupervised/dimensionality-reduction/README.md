# Training Pipeline

It doesn't form groups, it **compresses many features into few while preserving structure.** It's often a *preprocessing step that feeds other pipelines* rather than a final deliverable. I've marked the key breaks with ★.

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Dimensionality Reduction — Unsupervised)          │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    ★No labels y★ — goal: represent D features with d ≪ D, keep structure
    Decide the PURPOSE first (it dictates the method) ★:
    - Visualization (→ 2D/3D for human eyes): t-SNE, UMAP
    - Preprocessing / denoising / compression: PCA, Autoencoder
    - Feed downstream model (speed, curse-of-dim relief): PCA
                │
                ▼
[1] Data Collection
    raw data, features X only ★ (no target)
                │
                ▼
[2] Data Split
    ★If used as preprocessing for a supervised model: fit on TRAIN only★
       (fitting PCA on all data = leakage — test info leaks into components)
    ※ If purely for exploration/visualization, often fit on all data
                │
                ▼
[3] Preprocessing
    - missing values/outliers (★PCA is very outlier-sensitive★)
    - encoding categoricals
    - ★SCALING IS ESSENTIAL★ — PCA chases high-variance directions,
      so an unscaled large-range feature dominates the components
      (standardize so each feature contributes fairly)
                │
                ▼
[4] Method Choice ★replaces "architecture"★
    LINEAR:
    - PCA   : orthogonal axes of max variance; fast, invertible,
              great for compression/preprocessing
    NONLINEAR (manifold):
    - t-SNE : preserves LOCAL neighborhoods; visualization only,
              ★not for preprocessing★ (no reliable transform of new data)
    - UMAP  : faster than t-SNE, preserves local + some global; can
              transform new points
    DL:
    - Autoencoder : encoder compresses → bottleneck (latent d) →
                    decoder reconstructs; nonlinear, scalable
    ★key hyperparameter: target dimension d (and perplexity / n_neighbors)★
                │
                ▼
[5] Fit / Transform ★replaces forward+loss+backprop (method-dependent)★
    - PCA   : eigendecomposition / SVD of covariance → top-d components
              (one-shot linear algebra, no gradient descent) ★
    - t-SNE/UMAP : iteratively arrange points so near-stays-near
    - Autoencoder : THIS one does train via forward → reconstruction
                    loss (MSE) → backprop ★ (the only DL-style member)
    → output: X reduced from N×D to N×d
                │
                ▼
[6] Choosing d / Validation ★no label-loss; intrinsic criteria★
    - PCA: ★explained variance ratio★ — cumulative variance vs d,
           pick d that retains e.g. 90~95% (scree-plot elbow)
    - Autoencoder: reconstruction error vs d
    - Visualization methods: judged by eye (do meaningful patterns appear?)
                │
                ▼
[7] Evaluation ★no "accuracy" — judge by purpose★
    - Compression: reconstruction error / variance retained
    - Visualization: are clusters/structure visible & meaningful?
      ★CAUTION: t-SNE/UMAP distances & cluster SIZES aren't literal★
      (don't over-read gaps — they're not faithful global distances)
    - As preprocessing: ★does the downstream model improve?★
      (the real test is end-task performance, not the embedding alone)
                │
                ▼
[8] Deployment / Monitoring
    - PCA / UMAP / Autoencoder: ★save the fitted transform★, apply
      identically to new data (transform only, never refit on test) ★
    - t-SNE: ★cannot transform new points★ — refit each time
    → monitor drift: if input distribution shifts, refit
```

The essence in one line: **dimensionality reduction trades "labels → loss → metric" for "compress → preserve structure → measure what was retained."** It shares clustering's no-y, scaling-is-critical nature, but where clustering outputs *group IDs*, this outputs a *smaller feature representation.*

Two sharp distinctions worth burning in:

First, **PCA vs t-SNE/UMAP is a purpose split, not a quality ranking.** PCA is linear, fast, invertible, and gives you a reusable transform — ideal for *preprocessing*. t-SNE/UMAP are nonlinear and stunning for *visualization*, but t-SNE in particular has no honest way to project new data and its plotted distances are not literal. Reaching for t-SNE as a preprocessing step is a classic mistake.

Second, **the autoencoder is the one member that actually trains like the supervised pipelines** — forward pass, reconstruction loss (MSE between input and rebuilt output), backprop. So if this branch felt suddenly familiar at step [5], that's why: the autoencoder is the bridge back to the gradient-descent world you started in.

And how it sits against the two unsupervised cousins:

| Axis | Clustering | Dimensionality Reduction |
|------|-----------|--------------------------|
| Output | group ID per point | smaller feature vector per point |
| Question answered | "which group?" | "how to compress while keeping structure?" |
| Typical use | segmentation, discovery | visualization, denoising, preprocessing |
| Scaling critical? | yes | yes |
| "Validation" | silhouette, elbow | explained variance, reconstruction error |
| Often feeds... | business decisions | another ML pipeline ★ |
