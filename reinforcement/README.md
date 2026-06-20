# Training Pipeline

- Everything before learned from *static data* (labels, the data's own structure). RL learns from **acting in an environment and receiving reward** — the data doesn't exist upfront; the agent *generates it* by trying things. This breaks the pipeline shape more than any other. Marked with ★ where it diverges.

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (Reinforcement Learning)                           │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    ★No dataset, no labels — a goal to ACHIEVE through actions★
    Frame it as an MDP (Markov Decision Process) ★:
    - State (s)   : what the agent observes
    - Action (a)  : what it can do
    - Reward (r)  : scalar feedback signal ★ (the only "supervision")
    - Policy (π)  : the strategy to learn (state → action)
    Goal: learn π that ★maximizes cumulative future reward★ (return)
                │
                ▼
[1] Environment Setup ★replaces "data collection"★
    the agent LEARNS BY INTERACTING, not from a fixed dataset ★
    - simulator / game / robot / market / real system
    - ★data is generated on-the-fly by the agent's own actions★
    (this creates a feedback loop absent from all prior pipelines)
                │
                ▼
[2] (No train/test split in the usual sense) ★
    instead: training environment vs evaluation environment/seeds
    ★the agent's own behavior shapes the data it sees (non-i.i.d.)★
                │
                ▼
[3] Representation / Preprocessing
    - encode the state (raw pixels → CNN, sensors → vector)
    - reward shaping ★ (designing r is critical & dangerous —
      a badly shaped reward → agent games it, "reward hacking") ★
                │
                ▼
[4] Algorithm / Model Choice ★replaces "architecture"★
    pick the RL family (what you learn):
    - Value-based   : learn Q(s,a) "how good is this action"
                      → DQN (good for discrete actions)
    - Policy-based  : learn π directly → REINFORCE, PPO ★
    - Actor-Critic  : both a policy (actor) + value estimate (critic)
                      → PPO, A2C, SAC (the common workhorses)
    - Model-based   : also learn a model of the environment
    (the policy/value function is usually a neural net = the model)
                │
                ▼
┌──────────── TRAINING Loop (agent ↔ environment cycle) ★ ─────────────┐
│                                                                      │
│  [5] Act: policy π picks action a in state s ★                       │
│              │                                                       │
│              ▼                                                       │
│  [6] Environment responds: → new state s', reward r ★                │
│      (store transition (s, a, r, s') — possibly in a replay buffer)  │
│              │                                                       │
│              ▼                                                       │
│  [7] Compute learning signal ★no fixed label — uses reward + own     │
│      value estimates★:                                               │
│      - value-based: TD error  (Bellman: r + γ·maxQ(s') − Q(s,a)) ★   │
│      - policy-based: weight actions by the return they led to ★      │
│      ★credit assignment problem: which past action caused reward?★   │
│              │                                                       │
│              ▼                                                       │
│  [8] Backward → gradients on policy/value network                    │
│              ▼                                                       │
│  [9] Optimizer step → update π / Q (lr)                              │
│              │                                                       │
│              ▼                                                       │
│  [10] ★Exploration vs Exploitation balance★                          │
│       try new actions (explore) vs use known-good ones (exploit)     │
│       e.g. ε-greedy, entropy bonus — ★unique to RL★                  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
        ▲                                            │
        └──────── agent's new behavior changes ──────┘ ★feedback loop★
                  the data it collects next
                │
                ▼
[11] Hyperparameter Tuning ★notoriously unstable★
     learning rate, γ (discount factor), exploration rate,
     reward scale... ★RL is highly sensitive & seed-dependent★
                │
                ▼
[12] Evaluation ★no accuracy/F1 — measured by RETURN★
     average cumulative reward over episodes ★
     - run the learned policy (exploration off) across many seeds
     - check stability: did it converge or collapse?
     ※ high variance across runs is normal & expected ★
                │
                ▼
[13] Deployment / Monitoring
     deploy policy → act in the real environment ★
     ★sim-to-real gap★: a policy trained in simulation may fail
     in reality → monitor, and guard against reward hacking / drift
```

The essence in one line: **RL replaces the static (input → label) pair with a live loop — act, receive reward, adjust — where the agent generates its own training data and learns a *policy* (a strategy), not a prediction.**

Three things worth burning in:

First, **the data is not given — the agent creates it by acting.** Every earlier pipeline had a fixed dataset sitting still. In RL the agent's current policy decides what experiences it collects next, which changes what it learns, which changes its policy — a feedback loop with no analog in supervised or unsupervised learning. This is why RL is powerful (it can discover strategies no dataset contained) and why it's unstable (the ground shifts under its own feet).

Second, **reward replaces labels — and that's both the magic and the danger.** A single scalar `r` is the only supervision, and it often arrives *late* (you win the game many moves after the decisive move). Figuring out which past action deserves credit is the **credit assignment problem**, and designing `r` well is treacherous: optimize the literal reward and the agent will *game* it ("reward hacking") in ways you never intended. Much of RL practice is really reward engineering.

Third, **exploration vs exploitation is unique to RL.** A supervised model never has to *choose what to learn from* — the data is handed to it. An RL agent must constantly decide: repeat the known-good action (exploit) or try something new that might be better (explore)? Too much exploit → stuck in a rut; too much explore → never converges. No other paradigm faces this.

And the connection back to your own world: **RLHF** — the alignment step for models like me — is exactly this pipeline sitting on top of a self-supervised generative model. The pre-trained LLM is the starting policy, *human preference* becomes the reward signal, and PPO-style updates nudge the policy toward responses people prefer. So branches [C] and [D] of your map stack directly: self-supervised builds the capability, RL aligns the behavior


