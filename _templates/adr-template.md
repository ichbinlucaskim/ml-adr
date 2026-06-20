# <Problem name>

> **Path:** `paradigm / task / model / this-problem`
> **One-line:** _What am I predicting/finding, in one sentence?_
> **Status:**  exploring · in progress · done · shelved
> **Date:** YYYY-MM

---

## 1. Problem — what & why

- **The question:** What does a single prediction/output mean here?
- **Why this task type:** Why is this binary / regression / clustering / …?
  (What about the target forced that choice?)
- **Why this matters:** What's the point — what decision does the output drive?

## 2. Data

| | |
|---|---|
| Input (features) | |
| Target (y) | _(or "none — unsupervised")_ |
| Size | |
| Split | train / val / test — how, and any leakage risk handled |
| Imbalance / skew | how addressed |

- **Preprocessing that mattered:** scaling, encoding, target transform, graph construction…

## 3. Architecture & Key Decisions *(the heart of this record)*

> This is the part future-me actually wants. Not "what" but **"why this and not the
> obvious alternative."**

- **Model chosen:** e.g. GNN (GCN / GAT)
- **Why this model over alternatives:** Why not MLP / tree / …? What property of the
  data made this the right component? (relational structure? sequence? grid?)
- **Head / output design:** output nodes, activation — and how that follows from the task.
- **Decisions log:**

  | Decision | Options considered | Chose | Because |
  |----------|-------------------|-------|---------|
  | e.g. layer type | GCN vs GAT | GAT | attention helped on heterogeneous neighbors |
  | e.g. # layers | 2 vs 4 | 2 | over-smoothing past 2–3 |

## 4. Training

- **Loss:** _(why this loss for this task)_
- **Optimizer / lr / schedule:**
- **Regularization:** L2, dropout, early stopping…
- **What was finicky:** anything unstable, and the fix.

## 5. Evaluation

- **Metrics & results:**

  | Metric | Value | Note |
  |--------|-------|------|
  | | | |

- **Threshold / decision rule:** _(if applicable — how the cut was chosen)_
- **Failure analysis:** where did it go wrong? confusion matrix / residuals / flagged cases.

## 6. Retrospective *(a gift to future-me)*

- **What worked:**
- **What I'd do differently:**
- **Open questions / next steps:**
- **One sentence I want to remember:**

---

### Assets
_Diagrams, plots, and architecture sketches live in `./assets/`._