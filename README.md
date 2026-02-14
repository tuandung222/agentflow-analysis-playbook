# AgentFlow Analysis Playbook

This repository is for newcomers who want to:
- Understand the original `AgentFlow` source code (Stanford/Lupantech, ICLR 2026)
- Learn RL training foundations for LLMs in an **agentic planning + tool orchestration** setup
- Follow a practical path to read, run, debug, and extend the system

## Repository goals
- Separate **conceptual understanding** from **code-level flow analysis**
- Explain train/inference data flow in AgentFlow clearly
- Highlight common failure points for newcomers (GPU, Ray, vLLM, tool-calling)

## Target audience
- Engineers getting started with RLHF/GRPO
- Builders designing LLM systems with Planner-Executor-Verifier patterns
- Teams creating internal onboarding material

## Documentation structure
- `docs/00-source-summary.md`: Quick summary of the original repository
- `docs/01-system-architecture.md`: Module architecture and inference lifecycle
- `docs/02-flow-grpo-training.md`: Flow-GRPO/verl training pipeline
- `docs/03-hands-on-setup.md`: Setup, smoke tests, and environment checks
- `docs/04-concepts-primer.md`: RL + orchestration primer for LLM systems
- `docs/05-mastery-roadmap.md`: 2–4 week roadmap to master the codebase
- `docs/06-extension-guide.md`: How to add tools, datasets, rewards, and benchmarks
- `docs/07-code-deep-dive.md`: Code-path deep dive for inference and training internals
- `docs/08-verl-training-preparation-vi.md`: Vietnamese guide on required files for `verl` RL training and AgentFlow adaptation
- `docs/09-four-subagents-runtime-analysis-vi.md`: Vietnamese deep analysis of 4 specialized sub-agents, runtime flow, and comparisons with ReAct/CodeAct/Planner-Worker
- `docs/10-subagent-tuning-analysis-vi.md`: Vietnamese deep analysis of sub-agent tuning strategy, data format, and pseudocode for `verl` training
- `docs/advanced-planner-tuning/README-vi.md`: Advanced Vietnamese tutorial series (PhD/AI Research focus) for planner tuning with fixed worker/verifier/generator

## Original repository analyzed
- Source: https://github.com/lupantech/AgentFlow
- Local clone snapshot: `/Users/admin/TuanDung/repos/AgentFlow`

## Suggested reading order
1. `docs/04-concepts-primer.md`
2. `docs/00-source-summary.md`
3. `docs/01-system-architecture.md`
4. `docs/03-hands-on-setup.md`
5. `docs/02-flow-grpo-training.md`
6. `docs/06-extension-guide.md`
7. `docs/05-mastery-roadmap.md`
8. `docs/07-code-deep-dive.md`

## Scope and limits
- Scope: architecture analysis, execution flow, implementation behavior, and extension points
- Out of scope: copying the entire implementation from the original repository
