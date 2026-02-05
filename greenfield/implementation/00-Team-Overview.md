<!-- CONFIGURATION -->
<!-- Before running, read 'workflow-config.yaml' at the workflow root to resolve the following paths: -->
<!-- {{knowledge_root}}, {{artifact_root}}, {{obsidian_vault_root}}, {{e2e_tests_root}} -->

# AI Development Team — Overview and Integration Guide

## Team Composition

### Knowledge Management
| Agent | Primary Role | Execution Mode | Managed By | Model |
|------|---------------|----------------|------------|-------|
| **Reference Librarian** | Knowledge gateway, query response, accumulation | Standalone | Orchestrator | Configured per run |

### Planning/Decomposition Agents
| Agent | Primary Role | Execution Mode | Managed By | Model |
|------|---------------|----------------|------------|-------|
| **WS Micro-Level Planner** | Workstream-level implementation specs | Standalone | Human | — |
| **Work Decomposer** | Break workstream plan into atomic UoWs | Standalone | Human | — |

### Execution & Validation Agents
| Agent | Primary Role | Execution Mode | Managed By | Model |
|------|---------------|----------------|------------|-------|
| **Workstream Orchestrator (W-O)** | Workstream coordination, spawn UOWO | Autonomous | Human | Sonnet |
| **Unit-of-Work Orchestrator (UOWO)** | Single UoW lifecycle management | Subagent | W-O | Sonnet |
| **Work Assigner** | Create UoW assignment | Subagent | UOWO | Haiku |
| **Software Engineer** | Implement UoW | Subagent | UOWO | Haiku |
| **Implementation Evaluator** | Validate implementation against DoD + gates | Subagent | UOWO | Haiku |
| **Unit/Component Test Writer** | Write unit/component tests | Subagent | UOWO | Haiku |
| **Test Evaluator** | Validate test coverage & gates | Subagent | UOWO | Haiku |
| **Integration Test Writer** | Write end-to-end integration tests | Subagent | UOWO | Haiku |
| **Code Reviewer** | Review implementation quality | Subagent | UOWO | Haiku |
| **QA Engineer** | User-focused validation | Subagent | UOWO | Haiku |
| **QA Evaluator** | Evaluate QA report + gates | Subagent | UOWO | Haiku |
| **UI QA Agent** | UI consistency validation (conditional) | Subagent | UOWO | Haiku |
| **UI QA Evaluator** | Evaluate UI QA report | Subagent | UOWO | Haiku |
| **DevOps** | Deploy approved code | Subagent | UOWO | Haiku |

---

## Core Workflow (Execution Phase)

**High-level sequence per UoW:**
```
Assignment → Implementation → Eval(Impl) → Unit Tests → Eval(Tests) → Integration Tests → Eval(Tests)
  → Code Review → QA → Eval(QA) → UI QA (conditional) → Eval(UI QA) → Deployment
```

### Evaluator–Optimizer Loop
Each stage with an evaluator uses this loop:
1. Generation (agent produces artifact)
2. Evaluation (evaluator checks rubric + programmatic gates)
3. Refinement (agent revises)
4. Repeat until pass or stop condition

**Stopping criteria:** pass, max iterations, token/compute budget, similarity plateau, or escalation.

### UI QA Conditional Trigger
UI QA runs only if UI changes detected:
- File types: `.html`, `.css`, `.scss`, `.tsx`, `.vue`, `.component.ts`, `.component.html`
- Directories: `components/`, `ui/`, `styles/`, `views/`, `templates/`

---

## Configuration & Artifact Roots
All agents must resolve paths from `workflow-config.yaml`:
- `{{knowledge_root}}`
- `{{artifact_root}}`
- `{{obsidian_vault_root}}`
- `{{e2e_tests_root}}`

Artifacts live under:
```
{{artifact_root}}{CHANGE-ID}/
```

### Standard Artifact Structure
```
{{artifact_root}}{CHANGE-ID}/
  intake/
    story.yaml
    config.yaml
    constraints.md
  planning/
    tasks.yaml
    uow_plan.yaml
    assignments.yaml
  execution/
    UOW-001/
      assignment_report.yaml
      impl_report.yaml
      unit_test_report.yaml
      integration_test_report.yaml
      code_review_report.yaml
  qa/
    qa_report.yaml
    ui_qa_report.yaml
    evidence/
      ui/
  logs/
    reference_librarian/
    work_assigner/
    software_engineer/
    implementation_evaluator/
    unit_test_writer/
    integration_test_writer/
    test_evaluator/
    code_reviewer/
    qa/
    qa_evaluator/
    ui_qa/
    ui_qa_evaluator/
    devops/
    orchestrator/
  summary/
    final_summary.md
  work-log.md
```

---

## Knowledge Flow (Mandatory)
- All agents query the **Reference Librarian** first for knowledge.
- Agents **do not** access knowledge files directly.
- Exploration findings are reported back to the librarian.
- Librarian updates `accumulated-knowledge.md` and `learnings.yaml`.

---

## Logging Requirements (Dual Logging)
1. **Machine logs**: Structured YAML logs in `{CHANGE-ID}/logs/<agent>/`.
2. **Human log**: Append major actions to `{CHANGE-ID}/work-log.md` with timestamp, command, result, and rationale.

---

## Guardrails & Kill Switches
- **Forbidden actions**: external network calls, secrets access, env files, lock files, generated directories.
- **Kill switches**: diff > 500 lines, repeated identical edits, forbidden file touch, network egress.
- **Escalation**: If blocked or plateau detected, stop and escalate with context.

---

## Consistency Rules
- Keep prompts project-agnostic.
- Avoid references to any specific test framework in prompts unless required by the workflow stage.
- Follow existing repo conventions for naming, structure, and logs.
