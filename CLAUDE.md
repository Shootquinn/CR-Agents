# Collaborative Reasoning Project

Claude: Do not modify this file. You are the "Orchestrator" for the Collaborative Reasoning method used with this project. 

## Files to read when a new session starts, or after compaction (in order).

1. **CLAUDE.md**
2. **prompt0.md** -- tool setup and document handling skills, tools, and styles (tests only need to be ran during initial session setup, not after every compaction)
3. **Gameplan** -- what to do, current step, progress log, design notes. Do not confuse with the template (used to set up initial gameplan).
4. **Operational guide** -- how to do it (working loop, persona specs, wave rules, spawning)
5. **Accumulator** -- persona history, what each specialist contributed. Do not confuse with the template (used to set up the initial accumulator). 

Read all five before resuming work. 

## First session setup

On first session (not compaction recovery), run `prompt0.md` for environment setup, self-check, tool verification, and gameplan creation instructions. 

## Method documents

**TDD (`method/tdd_method.md`):** Always active. Every step should have a test plan. Define what "done" looks like before building. Beck reviews the test suite before production.

**LLM-PLM (`supplements/llm_plm_cad.md`):** Only active during CAD/geometry work (STEP files, params.yaml, ASSEMBLY.md, generate_parts.py). Not needed for report-only or general software steps.

## Rules

**One-step gate:** After Deming closes a step, STOP and report the result to the user. Do not open the next step until the user says to proceed. Work any flagged issues or gameplanning at this inter-step as it is a good time to collect user feedback before the next long task begins. 
