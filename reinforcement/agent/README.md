# Training Pipeline

- The closest relative to RL — an agent still **acts in an environment and pursues a goal** — but the "learning" is mostly displaced from gradient updates onto **a pre-trained LLM + prompt/tool/memory scaffolding**, and the hard part shifts to **orchestration and evaluation of multi-step trajectories**. Marked with ★ where it diverges from RL and all prior pipelines.

```
┌──────────────────────────────────────────────────────────────────────┐
│  Full ML Pipeline (LLM Agent)                                        │
└──────────────────────────────────────────────────────────────────────┘

[0] Problem Definition
    ★A goal to ACHIEVE through multi-step action — like RL — but★
    ★the "policy" is mostly a FROZEN pre-trained LLM, not learned★
    Frame the agent loop:
    - Observation (o) : context the agent sees (prompt, tool results, memory)
    - Action (a)      : a tool call, an API call, code, or a final answer
    - Policy (π)      : ★the LLM + prompt + scaffolding★ (often no weights updated)
    - Objective       : complete the task correctly & efficiently
    ★No reward signal by default — success is judged after the fact (eval)★
                │
                ▼
[1] Environment / Tool Setup ★replaces "data collection"★
    define what the agent can DO and SEE ★:
    - tools / functions (search, code-exec, DB, browser, APIs)
    - tool schemas (name, args, when-to-use descriptions) ★critical★
    - memory store (short-term context, long-term vector/DB) ★
    - the system prompt = the agent's "operating manual" ★
    ★the action space is DEFINED IN TEXT, not a fixed discrete/continuous set★
                │
                ▼
[2] (No train/test split) ★
    instead: a curated ★eval set of tasks★ (with known good outcomes)
    held separate from the tasks used to iterate on prompts/tools ★
                │
                ▼
[3] Representation / Context Engineering ★replaces preprocessing★
    - how observations are packed into the context window ★
    - retrieval (RAG): fetch relevant docs/memories into context ★
    - context management: summarize/trim history (window is finite) ★
    - tool-result formatting (the agent reads text — format matters) ★
    ※ analog of "feature engineering" — but for what the LLM SEES ★
                │
                ▼
[4] Agent Architecture ★replaces "model/algorithm choice"★
    pick the orchestration pattern (what the loop looks like):
    - ReAct          : interleave Reason → Act → Observe ★ (workhorse)
    - Plan-and-Execute: make a plan, then execute steps
    - Reflexion      : act → self-critique → retry ★
    - Multi-agent    : router + specialist sub-agents that talk ★
    - Tool-use / function-calling is the primitive underneath all of these
    (the LLM weights are usually FIXED — you design the SCAFFOLD) ★
                │
                ▼
┌──────────── AGENT Loop (agent ↔ environment cycle) ★ ────────────────┐
│                                                                      │
│  [5] Observe: assemble context (prompt + history + tool results) ★   │
│              │                                                       │
│              ▼                                                       │
│  [6] Reason / Decide: LLM forward pass → picks next action ★         │
│      (NO gradient step here — inference only) ★                      │
│              │                                                       │
│              ▼                                                       │
│  [7] Act: execute the chosen tool / API / code ★                     │
│              │                                                       │
│              ▼                                                       │
│  [8] Environment responds: → tool output, new state, error ★         │
│      (append to context / memory → becomes next observation)         │
│              │                                                       │
│              ▼                                                       │
│  [9] Termination check: goal met? give-up? step/budget limit? ★      │
│      (loops can run away → ★must cap steps & cost★) ★                │
│              │                                                       │
│              ▼                                                       │
│  [10] ★Error handling / recovery★ (unique emphasis):                 │
│       tool failed, hallucinated args, wrong path →                   │
│       retry / re-plan / self-correct (Reflexion) ★                   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
        ▲                                            │
        └──────── agent's actions change the ────────┘ ★feedback loop★
                  context it reasons over next      (like RL — but the
                                                     loop is at INFERENCE,
                                                     not during training)
                │
                ▼
[11] Iteration / Tuning ★replaces hyperparameter tuning★
     ★you tune the SCAFFOLD, not weights (mostly)★:
     - prompt wording, tool descriptions, few-shot examples
     - which tools to expose, memory strategy, step budget
     - model choice (bigger/smaller LLM per sub-task) ★
     ※ optional deeper layer: fine-tune / RL the base model
       (RLHF, agentic RL on tool-use trajectories) ★ — this is where
       it folds BACK into the RL pipeline ★
                │
                ▼
[12] Evaluation ★the hardest part — output is a TRAJECTORY, not a label★
     measure over the eval task set ★:
     - Task success rate / goal completion ★ (the headline metric)
     - Tool-call accuracy (right tool, right args) ★
     - Trajectory quality: efficiency (# steps), cost (tokens/$), latency ★
     - ★Credit assignment: WHICH step caused the failure?★ (like RL)
     - LLM-as-judge for open-ended outputs ★ (judge with another model)
     ※ high variance & non-determinism are normal (sampling) ★
     ※ failure-mode analysis: classify HOW it failed (wrong tool,
        bad plan, hallucination, loop, context overflow) ★
                │
                ▼
[13] Deployment / Monitoring
     deploy the agent → acts in the real environment ★
     - ★guardrails★: permission checks, sandboxing, human-in-the-loop
       on irreversible actions (send, delete, pay) ★
     - monitor: success rate drift, cost/latency, tool-error rates ★
     - ★prompt-injection / tool-misuse risk★ (the agent reads untrusted
       text as input → it can be hijacked) ★ — new attack surface
     - log full trajectories → feed failures back into the eval set ★
```

The essence in one line: **an LLM agent reuses RL's act–observe–loop shape, but the "policy" is a frozen pre-trained model wrapped in prompt/tool/memory scaffolding — so the work moves from updating weights to designing the orchestration and, above all, to evaluating multi-step trajectories where success has no single label.**

Three things worth burning in:

First, **most "learning" is not gradient descent — it's scaffolding design.** In supervised/RL pipelines you improve by updating weights. In a default agent pipeline the LLM is frozen; you improve by changing the *prompt, the tools, the memory, the control flow.* The loop at [5]–[10] runs at **inference time**, not training time — every cycle is a forward pass, not a backward one. (The optional exception is fine-tuning / agentic-RL the base model at [11], which is exactly where the agent pipeline folds back into RL.)

Second, **evaluation is the real bottleneck — because the output is a trajectory.** A classifier emits one label you can score against one ground truth. An agent emits a *sequence* of decisions, tool calls, and intermediate outputs, and "did it succeed?" is often open-ended. This forces task-level success metrics, LLM-as-judge, and the **same credit-assignment problem RL has** (which step broke the chain?). If you can't measure it, you can't iterate it — so eval-set construction is the highest-leverage work.

Third, **the action space is open and dangerous in a new way.** RL acts in a bounded, defined environment. An agent's actions are *real tool calls* — it can send email, run code, spend money, hit production APIs — and its observations include *untrusted text* it might obey (prompt injection). So guardrails, sandboxing, step/cost caps, and human-in-the-loop on irreversible actions aren't optional polish; they're core pipeline stages with no real analog in any earlier paradigm.

How it sits relative to the families you've built:

| Axis | RL | LLM Agent |
|------|-----|-----------|
| What's learned | a policy π (from scratch) | ★mostly nothing — frozen LLM + scaffold★ |
| Where the loop runs | training time | ★inference time★ |
| Supervision | scalar reward, online | ★post-hoc eval (success/judge), offline★ |
| Action space | fixed (discrete/continuous) | ★open-ended tools defined in text★ |
| Core hard problem | credit assignment, exploration | ★trajectory evaluation + orchestration★ |
| Main risk | reward hacking, sim-to-real | ★prompt injection, unsafe real actions★ |
| Folds back into RL via | — | ★fine-tuning / agentic-RL the base model★ |