# Templates

One ADR template per learning paradigm. All four share a common spine — Problem (section 1), Architecture and key decisions (section 3), and Retrospective — so records remain consistent across the repository. The intermediate sections differ because each paradigm has distinct concerns.

| Paradigm | Template | Distinguishing concerns |
|----------|----------|-------------------------|
| Supervised | [`adr-supervised.md`](adr-supervised.md) | Target labels, decision threshold or argmax, accuracy / F1 / RMSE. |
| Unsupervised | [`adr-unsupervised.md`](adr-unsupervised.md) | No target; scaling and distance metric critical; intrinsic metrics; interpretation as the deliverable. |
| Self-supervised | [`adr-self-supervised.md`](adr-self-supervised.md) | Two stages — pretext task with derived labels, then downstream transfer and evaluation. |
| Reinforcement | [`adr-reinforcement.md`](adr-reinforcement.md) | MDP formulation, reward design, policy output, return over seeds, exploration. |

## Rationale for separate templates

A single supervised-oriented template does not transfer to the other paradigms:

- Unsupervised work has no target and no accuracy measure. Quality is intrinsic (e.g. silhouette) and ultimately a matter of interpretability, so the template centers evaluation on judgment supported by metrics rather than a single score.
- Self-supervised work is judged on a downstream task, not its pretext objective. The template enforces the two-stage structure that a flat template would obscure.
- Reinforcement learning has no fixed dataset. Its vocabulary is state, action, reward, and policy, so the data section becomes an environment section and evaluation is measured as return across seeds.

The constant across all four is section 3 — each decision recorded as a choice between alternatives with its rationale — and the retrospective.