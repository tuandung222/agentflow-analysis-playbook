# System Architecture

## End-to-end inference loop
In `solver.solve()`:
1. `planner.analyze_query()`
2. Step loop (`max_steps`, `max_time`):
   - `planner.generate_next_step()` -> context + sub_goal + tool
   - `executor.generate_tool_command()`
   - `executor.execute_tool_command()`
   - `memory.add_action()`
   - `verifier.verificate_context()` -> STOP/CONTINUE
3. Loop termination:
   - `planner.generate_final_output()` (detailed)
   - `planner.generate_direct_output()` (concise)

## Module responsibilities
- Planner:
  - Uses LLM(s) to analyze goals and choose tools step by step
  - Separates `llm_engine` and `llm_engine_fixed` for controlled behavior
- Executor:
  - Converts sub-goals into executable `tool.execute(...)` code
  - Parses generated commands, executes with timeout, stores outputs
- Verifier:
  - Checks whether memory is sufficient to answer the query
  - Returns STOP/CONTINUE control signal
- Memory:
  - Stores action history (tool, sub-goal, command, result)

## Tool subsystem
- `Initializer` scans `tools/**/tool.py`, loads metadata, and caches tool instances
- Includes short/long tool-name mapping for robust resolution
- Supports parallel tool loading to reduce startup latency

## Practical technical notes
- Prompt templates in code strongly affect behavior quality
- Executor uses generated code execution (`exec`) for flexibility, but this raises runtime/safety risks
- Production usage should harden timeout, sandboxing, and command validation
