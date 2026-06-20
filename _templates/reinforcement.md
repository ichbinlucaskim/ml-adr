# Problem name

| | |
|---|---|
| Path | `reinforcement / <model> / <this-problem>` |
| Summary | One sentence: what behavior or policy is learned. |
| Status | Exploring / In progress / Done / Shelved |
| Date | YYYY-MM |

## 1. Problem

- **Objective.** What the agent should learn to do. The output is a policy, not a prediction.
- **Rationale for RL.** Why the problem cannot be framed as supervised learning; the signal arrives only as the result of actions.
- **MDP formulation.**

| Element | Definition |
|---------|-----------|
| State (s) | What the agent observes. |
| Action (a) | The action space; discrete or continuous. |
| Reward (r) | The scalar signal, its shaping, and the risk of reward hacking. |
| Episode | What begins and ends an episode. |

## 2. Environment

Training data is generated through interaction rather than supplied as a fixed dataset.

| Field | Detail |
|-------|--------|
| Environment | Simulator, game, robot, or production system. |
| State representation | Raw pixels via CNN, sensor vector, etc. |
| Reward design | The shaping rule and its justification; note any exploitable loopholes. |
| Sim-to-real gap | If trained in simulation, how production conditions differ. |

## 3. Algorithm and key decisions

- **Algorithm family.** Value-based (DQN), policy-based (REINFORCE, PPO), actor-critic (PPO, SAC), or model-based.
- **Rationale.** Discrete vs continuous actions, sample efficiency, stability, on- vs off-policy.
- **Network.** Architecture of the policy or value network.

| Decision | Alternatives | Selected | Rationale |
|----------|-------------|----------|-----------|
| | | | |

## 4. Training

| Field | Detail |
|-------|--------|
| Learning signal | TD error (Bellman), policy gradient on returns, or advantage. |
| Exploration | Epsilon-greedy, entropy bonus, and the schedule. |
| Discount, replay, on/off-policy | |
| Stability notes | RL is sensitive to seeds and hyperparameters; record divergence or collapse and the resolution. |

## 5. Evaluation

Performance is measured by return, not accuracy.

| Metric | Value | Note |
|--------|-------|------|
| Mean return (across seeds) | | High variance across seeds is expected. |
| Sample efficiency | | Steps to reach a target return. |

Evaluate the learned policy with exploration disabled; confirm convergence rather than collapse. Inspect behavior to verify the policy solves the task rather than exploiting the reward function.

## 6. Retrospective

- **What worked.**
- **Reward hacking.** Whether observed, and how detected or corrected.
- **What to change next time.** Reward shaping, algorithm, exploration strategy.
- **Open questions and next steps.**

## Assets

Reward curves, learned-behavior recordings or plots: `./assets/`.