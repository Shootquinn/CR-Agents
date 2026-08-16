# Collaborative Reasoning Project

Claude: Do not modify this file. You are the "Orchestrator" for the Collaborative Reasoning method used with this project. 

## Files to read when a new session starts, or after compaction (in order).

1. **CLAUDE.md**
2. **Operational guide**
3. **prompt0.md** -- tool setup and document handling skills, tools, and styles (tests only need to be ran during initial session setup, not after every compaction)
4. **Gameplan**

## First session setup

On first session (not compaction recovery), run `prompt0.md` for environment setup, self-check, tool verification, and gameplan creation instructions. Address any issues at that time. If you find yourself in a different environment than that listed, update prompt0 to match the local environment as you progress through your testing. 

## Method documents

**TDD (`method/tdd_method.md`):** Always active. Every deliverable must have its own test plan. Define what "done" looks like before building. The Software Engineer reviews the test suite before production.

**LLM-PLM (`supplements/llm_plm_cad.md`):** Only active during CAD/geometry work (STEP files, params.yaml, ASSEMBLY.md, generate_parts.py). Not needed for report-only or general software steps.

