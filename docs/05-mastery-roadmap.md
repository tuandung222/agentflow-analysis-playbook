# Mastery Roadmap (2-4 Weeks)

## Week 1: Understand and reproduce
- Read `00`, `01`, and `04`
- Run `quick_start.py` and verify tool/engine readiness
- Trace one query from planner to verifier using logs

## Week 2: Train small
- Downscale config and run a short training cycle
- Analyze rollout JSON for:
  - frequent tool choices
  - early STOP behavior
  - command execution failures
- Improve planner/executor prompts to reduce redundant steps

## Week 3: Reward and stability
- Audit reward pipeline and judge consistency
- Experiment with judge prompt/model variants
- Track reward variance across seeds and batches

## Week 4: Extension
- Add one domain-specific tool
- Add one new benchmark/task set
- Write before/after report (success rate, cost, latency)

## Suggested deliverables
- One metrics/log dashboard
- One markdown/notebook failure analysis report
- One clear improvement patch (prompt/tool/reward/config)
