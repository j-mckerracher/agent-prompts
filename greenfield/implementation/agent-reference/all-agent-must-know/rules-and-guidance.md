<!-- CONFIGURATION -->
<!-- Before running, read 'workflow-config.yaml' at the workflow root to resolve the following paths: -->
<!-- {{knowledge_root}}, {{artifact_root}}, {{obsidian_vault_root}}, {{e2e_tests_root}} -->

## Guardrails 
- Only run explicitly allowed commands; never destructive/privileged or unapproved network calls.  
- If tools are unavailable, ask the user to run exact commands and paste outputs.  

## Logging & Transparency (Compliance with LLM-Logging-Policy)
- **Dual Logging Requirement**:
    1. **Machine Logs**: Continue producing structured JSON artifacts (`*_report.json`) for the Evaluator loop.
    2. **Human Logs**: Maintain a `work-log.md` in the `{{artifact_root}}{CHANGE-ID}/` directory.
- **Work Log Content**:
    - Append entries for every major action (command execution, file edit, decision made).
    - Format: Markdown with timestamps.
    - Include:
        - **Command**: The exact command run.
        - **Result**: A 1-2 line summary of the outcome.
        - **Rationale**: Why this action was taken.
    - This ensures a human-readable narrative exists alongside the machine state.

## Artifact Management  
- All artifacts are written to the Obsidian artifact root: `{{artifact_root}}{CHANGE-ID}/`  
- Code changes happen in the code repository.  
- Use the defined YAML structures for all reports (`impl_report.yaml`, `test_report.yaml`, `qa_report.yaml`, etc.)  
  
## Knowledge Management  
- Before starting work, review `knowledge/learnings.yaml` for relevant prior knowledge. If greenfield, also review PRD/plan docs referenced in `constraints.md`.  
- When encountering unknowns, add questions to `knowledge/questions.yaml`.  
- Document findings in `knowledge/learnings.yaml` with evidence.  
- Include `knowledge_referenced` and `knowledge_contributed` in all reports.  
  
## First Reply Expectations  
- Summarize the task and acceptance criteria.  
- List assumptions and validation steps.  
- Identify knowledge gaps and questions.  
- State current workflow stage.  
  
## Communication and Evidence  
- Ask clear, specific clarifying questions instead of assuming.  
- Cite evidence with file paths and brief code snippets when explaining behavior/decisions.  
  
## Implementation Discipline  
- Work in small, reviewable steps; mirror existing repo patterns and conventions.  
- Adhere to lint/type rules; add/extend tests; run checks and report results.  
- Follow scope control: only make changes required for the current UoW.  
- Provide change summaries, risks, and flag breaking changes for escalation.  
  
## Testing Guidance  
- Tie tests to ACs/contracts; prefer deterministic, isolated unit tests; minimal mocking.  
- Mirror existing test file naming and placement; avoid brittle selectors in UI tests.  
- Jest and Cypress gates must pass before advancing stages (or the configured `test_stack` equivalent).  
  
## Decision-Making  
- Propose options with trade-offs when risk/ambiguity exists; choose minimal-risk approach.  
- When modules/policies conflict, follow the most restrictive guardrail.  
- Escalate breaking changes or ambiguous requirements to human.
