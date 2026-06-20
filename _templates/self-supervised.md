# Problem name

| | |
|---|---|
| Path | `self-supervised / <model> / <this-problem>` |
| Summary | One sentence: what representation is learned and for what downstream use. |
| Status | Exploring / In progress / Done / Shelved |
| Date | YYYY-MM |

## 1. Problem

- **Objective.** The goal is a reusable representation, not the pretext task itself. State the downstream task(s) the representation is intended to serve.
- **Rationale for self-supervision.** Why labeled data is scarce or expensive while unlabeled data is abundant, and why a purely supervised approach is insufficient.
- **Two-stage plan.** Pre-training without human labels, followed by transfer or fine-tuning with limited labels.

## 2. Data

| Field | Detail |
|-------|--------|
| Pre-training data | Source and size; unlabeled. |
| Downstream data | The labeled set used for fine-tuning. |
| Split | Held-out data for pretext validation and downstream evaluation. |

Preprocessing that materially affected results: deduplication (to limit memorization), tokenization, augmentation.

## 3. Pretext task and architecture

The defining decision is how labels are derived from the data itself.

- **Pretext task.** Masked prediction, next-token prediction, contrastive, or reconstruction, and why it should induce structure useful for the downstream goal.
- **Backbone.** The encoder retained after pre-training; the pretext head is discarded.
- **Rationale.** Why this backbone and pretext task over the alternatives.

| Decision | Alternatives | Selected | Rationale |
|----------|-------------|----------|-----------|
| | | | |

## 4. Pre-training

| Field | Detail |
|-------|--------|
| Loss | Cross-entropy (masked / next-token), InfoNCE (contrastive), or MSE (reconstruction). |
| Optimizer / LR / schedule / scale | Note compute, as these methods are data-intensive. |
| Stability notes | Representation collapse, augmentation strength, negative sampling. |

## 5. Transfer and evaluation

Quality is judged on the downstream task, not the pretext objective.

- **Transfer method.** Linear probe, full fine-tune, frozen backbone with new head, or adapters / LoRA.

| Downstream task | Metric | Value | vs from-scratch baseline |
|-----------------|--------|-------|--------------------------|
| | | | |

Record whether low pretext loss did or did not translate into downstream gains.

## 6. Retrospective

- **Transfer outcome.** Where the representation helped most and least.
- **What to change next time.** Pretext design, backbone size, augmentations.
- **Open questions and next steps.**

## Assets

Embedding visualizations, linear-probe curves: `./assets/`.