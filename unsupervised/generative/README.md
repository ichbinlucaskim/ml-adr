# Training Pipeline

- Instead of *judging* data (classify, score, group), it **learns the distribution p(x) and then samples from it to create new data.** It's density estimation's other child — same root (model p(x)), opposite use (sample, don't flag rarity). Marked with ★ where it diverges.

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Generative)                                       │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    Goal: learn the data distribution p(x), then ★generate new samples★
    Decide what you need (it picks the model family) ★:
    - sharp images / audio          → GAN, Diffusion
    - need the latent space / encode → VAE
    - exact likelihood p(x)          → Normalizing Flow, autoregressive
    - text / sequences               → autoregressive Transformer (LLM)
                │
                ▼
[1] Data Collection
    raw data X only ★ (no labels — you model the data itself)
    ★quantity & quality dominate★ — generative models are data-hungry
    (conditional generation may add a label/prompt c → p(x|c))
                │
                ▼
[2] Data Split
    Train / (held-out for evaluation)
    ★no "test accuracy" — held-out set is used to check the model
      didn't just memorize, and to compute likelihood/FID★
                │
                ▼
[3] Preprocessing
    - clean, deduplicate (★memorization risk★ — dupes get regurgitated)
    - normalize inputs (e.g. images to [-1,1]); tokenize text
    - ★no target column — x IS the target★ (model learns to reproduce x)
                │
                ▼
[4] Architecture / Model Choice ★the family defines everything★
    - VAE        : encoder → latent z → decoder; smooth latent space,
                   blurrier samples
    - GAN        : generator vs discriminator (adversarial game) ★;
                   sharp samples, trickier/unstable training
    - Diffusion  : add noise step-by-step, learn to DENOISE in reverse;
                   current SOTA for images, stable but slow to sample
    - Autoregressive (LLM): predict next token given previous ★;
                   text, code — generates sequentially
    - Normalizing Flow : invertible nets → exact p(x)
                │
                ▼
┌────────────────── Training Loop (repeat per epoch) ──────────────────┐
│                                                                      │
│  [5] Forward (FW)                                                    │
│      ★objective differs by family — this is the big divergence★      │
│              │                                                       │
│              ▼                                                       │
│  [6] Loss Computation ★no single loss — family-specific★             │
│      - VAE      : reconstruction loss + KL term (ELBO) ★             │
│      - GAN      : adversarial loss (gen fools discriminator,         │
│                   discriminator catches fakes) — two nets, minimax ★ │
│      - Diffusion: predict the noise that was added (MSE on noise) ★  │
│      - Autoreg  : ★Cross-Entropy on next token★ (yes — same CE as    │
│                   classification, just "classify the next token")    │
│      + Regularization                                                │
│              │                                                       │
│              ▼                                                       │
│  [7] Backward (Back)  → compute gradients                            │
│              │                                                       │
│              ▼                                                       │
│  [8] Optimizer step  → update W (lr)                                 │
│      ★GAN: alternate updating generator & discriminator★             │
│              │                                                       │
│              ▼                                                       │
│  [9] Validation (periodically)                                       │
│      sample some outputs, monitor quality + stability ★              │
│      (GANs can diverge / mode-collapse → watch closely)              │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                │
                ▼
[10] Hyperparameter Tuning
     lr, latent dim, noise schedule (diffusion), architecture size...
     ★GAN balance (gen vs discriminator strength) is delicate★
                │
                ▼
[11] Sampling / Generation ★replaces "threshold/argmax"★
     draw new data from the learned distribution ★
     - VAE/GAN: feed random z → decoder/generator → sample
     - Diffusion: start from noise → iteratively denoise → sample
     - LLM: sample tokens one by one (temperature / top-p control) ★
                │
                ▼
[12] Evaluation ★hardest part — "quality" has no clean number★
     - Images: ★FID★ (distance to real distribution), Inception Score
     - Text:   perplexity, + human eval / benchmarks
     - Likelihood models: held-out log-likelihood / bits-per-dim
     - ★Always check: diversity (mode collapse?) + memorization★
       (a model that copies training data scores well but is useless)
     ※ no accuracy/F1 — there's no single "correct" output ★
                │
                ▼
[13] Deployment / Monitoring
     deploy → serve generations (with safety/quality filters) ★
     → monitor drift, misuse, and output quality over time
     (often + RLHF / fine-tuning to align outputs — see [D])
```

The essence in one line: **generative models replace "predict the label" with "learn p(x) and sample from it" — so the final step isn't a decision (threshold/argmax) but an act of *creation*, and evaluation has no single correct answer.**

Three things worth burning in:

First, **there is no single "generative loss" — the family defines the objective**, and this is the real branching point. A VAE balances reconstruction against a KL term; a GAN plays a two-network adversarial game; a diffusion model learns to predict-and-remove noise; an autoregressive LLM does plain cross-entropy on the next token. That last one is the surprising bridge: **an LLM is, mechanically, a multiclass classifier predicting the next token** — the same softmax + cross-entropy you built early on, just applied over a vocabulary, millions of times in sequence.

Second, **evaluation is the genuinely hard part.** With classification you had F1; here "is this a good sample?" resists a single number. FID for images, perplexity plus human judgment for text — and you must always watch two failure modes: **mode collapse** (the model produces only a narrow slice of the real variety) and **memorization** (it regurgitates training data). Both can look fine on a naive metric while being useless or harmful.

Third, **this branch connects back to where you started.** Density estimation feeds it (model p(x) → sample), and Reinforcement Learning often finishes it — **RLHF** takes a trained generative model and uses human preference as a reward to align its outputs. That's the [D] branch doing real work on top of [B].

That completes all four task families across the map you built — supervised (binary, multiclass, multilabel, regression), and unsupervised (clustering, dimensionality reduction, anomaly detection, generative). The only branches left unsketched as full pipelines are **self-supervised** and **reinforcement learning** themselves.