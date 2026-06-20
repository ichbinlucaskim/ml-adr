# Problem name

| | |
|---|---|
| Path | `supervised / <task> / <model> / <this-problem>` |
| Summary | One sentence: what is being predicted. |
| Status | Exploring / In progress / Done / Shelved |
| Date | YYYY-MM |

## 1. Problem

- **Prediction target.** What a single prediction represents.
- **Task type and rationale.** Why this is binary / multiclass / multilabel / regression — specifically, what property of the target determines it.
- **Learning signal.** What the labels are and how they were obtained.
- **Decision supported.** What downstream decision the output drives.

## 2. Data

| Field | Detail |
|-------|--------|
| Input features | |
| Target (y) | |
| Size | |
| Split | Train / validation / test strategy; leakage controls (time ordering, grouping). |
| Imbalance / skew | Handling: class weighting, resampling, target transform. |

Preprocessing that materially affected results: scaling, encoding, target transform, feature or graph construction.

## 3. Architecture and key decisions

State each decision as a choice between alternatives, with the reason.

- **Model.** The component selected and the data property that justifies it (relational structure, sequence, grid, tabular).
- **Output head.** Number of output nodes and activation, derived from the task (sigmoid + 1 / softmax + K / sigmoid x K / linear + 1).

| Decision | Alternatives | Selected | Rationale |
|----------|-------------|----------|-----------|
| | | | |

## 4. Training

| Field | Detail |
|-------|--------|
| Loss | And why it fits the task (BCE / cross-entropy / MSE / Huber). |
| Optimizer / LR / schedule | |
| Regularization | L2, dropout, early stopping. |
| Stability notes | Anything unstable during training and the resolution. |

## 5. Evaluation

| Metric | Value | Note |
|--------|-------|------|
| | | |

- **Decision rule.** Threshold selection method (PR curve, cost trade-off) or argmax.
- **Error analysis.** Confusion matrix, residuals, or characteristic failure cases.

## 6. Retrospective

- **What worked.**
- **What to change next time.**
- **Open questions and next steps.**

## Assets

Diagrams and plots: `./assets/`.