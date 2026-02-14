# Concepts Primer

## RL for LLMs (short version)
- SFT: teaches imitation from supervised examples
- RL stage: optimizes policy toward target rewards
- PPO/GRPO family: updates policy with constraints to limit destructive drift

## Why AgentFlow differs from single-model tool use
- Single-model approach: one LLM reasons + calls tools in one long context
- AgentFlow approach: separates modules with explicit responsibilities, improving control and modularity

## Planner / Executor / Verifier pattern
- Planner: chooses next action
- Executor: translates action into tool interaction
- Verifier: checks if evidence is sufficient to stop
- This mirrors a systems-control feedback loop

## Reward design in orchestration tasks
- Final-answer correctness is not enough
- Trajectory quality matters (tool choice, efficiency, consistency)
- Sparse rewards (0/1) often cause high variance; practical systems may need shaping/proxy metrics

## Core difficulty in this domain
- Long-horizon credit assignment
- Tool/API non-determinism
- Online training latency and cost
