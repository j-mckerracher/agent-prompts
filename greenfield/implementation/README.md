<!-- CONFIGURATION -->
<!-- Before running, read 'workflow-config.yaml' at the workflow root to resolve the following paths: -->
<!-- {{knowledge_root}}, {{artifact_root}}, {{obsidian_vault_root}}, {{e2e_tests_root}} -->

# Sequential Task Decomposition with Evaluator-Optimizer Loop

A multi-agent workflow for implementing user stories from acceptance criteria through implementation, testing, and QA.

## Overview

This workflow uses specialized AI agents to:
1. Decompose a user story into tasks (UoWs are supplied by planning docs for greenfield)
2. Implement changes with scope control
3. Write tests based on the configured `test_stack`
4. Validate acceptance criteria with evidence

Each stage includes an **Evaluator-Optimizer loop** that iteratively refines outputs until quality gates pass. For greenfield projects, PRD/plan documents are the source of truth until code exists.

## Setup (Required)

To run this workflow on a new machine or share it with team members:

1.  Copy `workflow-config.example.yaml` to `workflow-config.yaml`.
2.  Edit `workflow-config.yaml` to match your local environment paths.
    *   `knowledge_root`: Path to `agent-reference/knowledge/` in this repo.
    *   `artifact_root`: Where you want output files stored.
    *   `obsidian_vault_root`: Path to your Obsidian Vault (if used).
    *   `e2e_tests_root`: Path to the E2E test project.
    *   `project_type`: Set to `"greenfield"` for new projects without an existing codebase.
    *   `planning_docs_root` or `planning_docs_paths`: Required for greenfield to point to PRD/plan docs.
    *   `test_stack`: Optional override if not using Jest/Cypress.

## Quick Start

### 1. Start the Orchestrator

When you invoke the Orchestrator agent, it will immediately provide you with a copiable YAML template to fill out.

### 2. Fill Out the Configuration

The Orchestrator will give you this template:

```yaml
workflow:
  change_id: ""                    # Required: Unique ID (e.g., "4729040")
  code_repo: ""                    # Required for brownfield; optional for greenfield until scaffolding
  project_type: "brownfield"       # "greenfield" for new projects
  planning_docs_root: ""           # Optional: folder of PRD/plan docs
  planning_docs_paths: []          # Optional: explicit list of PRD/plan files
  test_stack: "jest+cypress"       # Optional: override test tooling

story:
  title: ""                        # Required: Brief title
  description: ""                  # Required: User story statement
  acceptance_criteria:             # Required for brownfield; optional for greenfield if using PRD/plan docs
    - ""
    - ""
  examples:                        # Optional
    - description: ""
      value: ""

models:                            # Optional - defaults to claude-sonnet-4-5 for agents
  software_engineer: "claude-sonnet-4-5"
  test_writer: "claude-sonnet-4-5"
```

### 3. Paste Your Completed YAML

Fill in your story details and paste it back. The Orchestrator will:
1. Validate your configuration
2. Normalize acceptance criteria to `AC1`, `AC2`, etc.
3. Create intake artifacts
4. Begin the workflow

### 3. Monitor Progress

Artifacts are written to:
```
{{artifact_root}}{CHANGE-ID}/
```

Check `intake/config.yaml` for current status and stage.

---

## Agent Hierarchy

The Orchestrator controls the workflow and delegates to specialized agents. **All agents must query the Reference Librarian first** before accessing any knowledge or doing codebase exploration.

```
                                 ┌─────────────────────┐
                                 │    ORCHESTRATOR     │
                                 │  (State Machine)    │
                                 └──────────┬──────────┘
                                            │
            ┌───────────────────────────────┼───────────────────────────────┐
            │                               │                               │
            ▼                               ▼                               ▼
┌───────────────┐              ┌───────────────┐              ┌───────────────┐
│  PLANNING     │              │  EXECUTION    │              │  VALIDATION   │
└───────┬───────┘              └───────┬───────┘              └───────┬───────┘
        │                               │                               │
┌───────┴───────┐              ┌───────┴───────┐                       │
│               │              │               │                       │
▼               ▼              ▼               ▼                       ▼
┌────────┐   ┌───────────┐   ┌──────────┐   ┌───────────┐           ┌──────────┐
│ Task   │   │ Assignm.  │   │ Software │   │   Test    │           │    QA    │
│Generat.│   │  Agent    │   │ Engineer │   │  Writers  │           │  Agent   │
└───┬────┘   └─────┬─────┘   └────┬─────┘   └─────┬─────┘           └────┬─────┘
    │              │              │               │                      │
    │              │              │         ┌─────┴─────┐                │
    │              │              │         │           │                │
    │              │              │         ▼           ▼                │
    │              │              │    ┌────────┐ ┌───────────┐          │
    │              │              │    │  Unit  │ │Integration│          │
    │              │              │    │  Test  │ │   Test    │          │
    │              │              │    │ Writer │ │  Writer   │          │
    │              │              │    └────────┘ └───────────┘          │
    │              │              │                                      │
    │              │              │                              ┌───────┴───────┐
    │              │              │                              │               │
    ▼              ▼              ▼                              ▼               ▼
┌────────┐   ┌──────────┐                   ┌──────────┐   ┌──────────┐
│Assignm.│   │          │                   │    QA    │   │  UI QA   │
│ Agent  │   │          │                   │  Agent   │   │  Agent   │
└────────┘   │          │                   └──────────┘   └──────────┘
             │          │                         │        (conditional)
             ▼          ▼                         ▼
         ┌──────────────────────────────────────────────────────────────────────┐
         │                      REFERENCE LIBRARIAN                             │
         │    (Mandatory first point of contact for ALL knowledge queries)      │
         │   accumulated-knowledge.md │ learnings.yaml │ standing-questions.md  │
         └──────────────────────────────────────────────────────────────────────┘

                                EVALUATORS (one per stage)
         ┌───────────────────────────────────────────────────────────────────────────────────┐
         │  Task Plan    │  Assignment  │  Impl  │  Test  │  QA   │  UI QA    │
         │  Evaluator    │  Evaluator   │  Eval  │  Eval  │ Eval  │  Eval     │
         └───────────────────────────────────────────────────────────────────────────────────┘
                    ▲                          ▲                      ▲
                    └──── Feedback Loop ───────┴───── Revisions ──────┘
```

### Agent Invocation Summary

| Orchestrator Invokes | Must Query Librarian First |
|---------------------|---------------------------|
| Task Generator | ✅ Yes |
| Assignment Agent | ✅ Yes |
| Software Engineer | ✅ Yes |
| Unit/Component Test Writer | ✅ Yes |
| Integration Test Writer | ✅ Yes |
| QA Agent | ✅ Yes |
| UI QA Agent (conditional) | ✅ Yes |
| *All Evaluators* | No (evaluators don't access knowledge) |

---

## Workflow Stages

```
┌─────────┐    ┌──────────┐    ┌────────┐    ┌───────────┐    ┌────┐    ┌───────┐
│ Intake  │───▶│ TaskPlan │───▶│ Assign │───▶│ Execution │───▶│ QA │───▶│ UI QA │
└─────────┘    └──────────┘    └────────┘    └───────────┘    └────┘    └───────┘
                    │              │              │              │           │
                    ▼              ▼              ▼              ▼           ▼
               [Evaluator]    [Evaluator]    [Evaluator]    [Evaluator]  [Evaluator]
                    │              │              │              │      (if UI changes)
                    └───revise─────┴───revise─────┴───revise─────┴──────────┘
```

### Stage 1: Intake
- **Input**: Raw story text from user
- **Output**: `story.yaml`, `config.yaml`, `constraints.md`
- **What happens**: Normalizes acceptance criteria to `AC1..ACn`, extracts examples and constraints
  - For greenfield: ingest PRD/plan docs and record their paths in `constraints.md`

### Stage 2: Task Planning
- **Agent**: Task Generator
- **Evaluator**: Task Plan Evaluator
- **Output**: `planning/tasks.yaml`
- **What happens**: Creates broad implementation tasks covering all ACs

### Stage 3: Assignment
- **Agent**: Assignment Agent
- **Evaluator**: Assignment Evaluator
- **Output**: `planning/assignments.yaml`
- **What happens**: Schedules UoWs respecting dependencies, identifying parallelization opportunities. For greenfield, UoWs come from planning docs (e.g., `Work-Decomposer-Output.md`).

### Stage 5: Execution (per UoW)

**5a. Implementation**
- **Agent**: Software Engineer
- **Evaluator**: Implementation Evaluator
- **Output**: `execution/{UOW-ID}/impl_report.yaml`
- **What happens**: Implements code changes, runs existing tests (or initializes the test stack for greenfield)

**5b. Unit/Component Testing**
- **Agent**: Unit/Component Test Writer
- **Evaluator**: Unit Test Evaluator
- **Output**: `execution/{UOW-ID}/unit_test_report.yaml`
- **What happens**: Writes unit and component tests co-located with code

**5c. Integration Testing**
- **Agent**: Integration Test Writer
- **Evaluator**: Integration Test Evaluator
- **Output**: `execution/{UOW-ID}/integration_test_report.yaml`
- **What happens**: Writes E2E tests in the dedicated E2E app

**Integration Test App Location**:
```
{{e2e_tests_root}}
```
If `test_stack` excludes integration tests or `e2e_tests_root` is empty, the Integration Test Writer should document a skip or scaffold the E2E app as part of setup UoWs.

### Stage 6: QA
- **Agent**: QA Agent
- **Evaluator**: QA Evaluator
- **Output**: `qa/qa_report.yaml`
- **What happens**: Validates all ACs with evidence, assesses regression risk, synthesizes knowledge. For greenfield, regression risk focuses on new baseline behavior.

### Stage 6b: UI QA (Conditional)
- **Agent**: UI QA Agent
- **Evaluator**: UI QA Evaluator
- **Output**: `qa/ui_qa_report.yaml`
- **Condition**: Only runs if UI changes were made (modified `.html`, `.css`, `.scss`, `.tsx`, `.component.ts`, etc.)
- **What happens**: Uses Playwright CLI to compare new/modified UI elements against baseline elements. If no baseline exists (greenfield), establish the initial baseline and validate against design requirements.
- **Key Mandate**: Match existing elements when they exist; otherwise create and document the baseline.

### Stage 7: Post-QA Review
- **Agent**: Orchestrator (awaits user feedback)
- **Output**: `feedback/feedback.yaml` (if issues found)
- **What happens**: User reviews completed work, approves or requests fixes

### Stage 8: Remediation (if needed)
- **Agents**: Routed based on issue type (Software Engineer, Unit Test Writer, Integration Test Writer, UI QA Agent, etc.)
- **Output**: `feedback/remediation_uows.yaml`, updated implementation
- **What happens**: Fixes user-identified issues, runs evaluator loop, returns for re-review

---

## Knowledge Management System

All knowledge flows through the **Reference Librarian Agent**. Agents do NOT access knowledge files directly—the librarian is the mandatory gateway to reduce context bloat.

### How It Works

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Agent needs to  │────▶│ Queries the      │────▶│ Librarian       │
│ know something  │     │ Reference        │     │ responds with   │
└─────────────────┘     │ Librarian FIRST  │     │ answer/hint     │
                        └──────────────────┘     └────────┬────────┘
                                                          │
                               ┌──────────────────────────┴──────────────────────────┐
                               │                                                      │
                               ▼                                                      ▼
                  ┌────────────────────────┐                         ┌────────────────────────┐
                  │ Confidence: FULL       │                         │ Confidence: PARTIAL/   │
                  │ Agent uses answer      │                         │ NONE - Agent explores  │
                  │ directly               │                         │ per librarian hints    │
                  └────────────────────────┘                         └────────────┬───────────┘
                                                                                   │
                                                                                   ▼
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Future agents   │◀────│ Librarian adds   │◀────│ Agent reports   │
│ can query this  │     │ to accumulated-  │     │ findings BACK   │
│ knowledge       │     │ knowledge.md     │     │ to librarian    │
└─────────────────┘     └──────────────────┘     └─────────────────┘
```

### Knowledge Directory

**Location**: `{{knowledge_root}}`

All knowledge is stored centrally in the workflow repository (not per-change-id), allowing learnings to persist across workflow runs. **Agents do not access these files directly—all access goes through the Reference Librarian.**

### Knowledge Files

| File | Purpose | Who Writes |
|------|---------|-----------|
| `accumulated-knowledge.md` | Cumulative knowledge discovered during workflows | Reference Librarian (when agents report findings) |
| `learnings.yaml` | Structured learnings with metadata | Reference Librarian |
| `standing-questions.md` | Questions that could NOT be answered | Reference Librarian (when exploration fails) |
| `questions.yaml` | Active questions being explored | Reference Librarian |
| `rls-system-architecture.md` | System architecture documentation | Manual updates |

### Librarian-Mediated Knowledge Flow

1. **Agent queries librarian**: Agent asks for information FIRST (required)
2. **Librarian responds**: With answer and confidence level
3. **If exploration needed**: Librarian provides hints on what to explore
4. **Agent explores**: Agent investigates the codebase as directed
5. **Agent reports back**: Agent sends findings to librarian
6. **Librarian accumulates**: Librarian adds to `accumulated-knowledge.md`
7. **If not found**: Librarian adds question to `standing-questions.md`

### Agent Knowledge Requirements

Every agent MUST:

1. **Before starting any task:**
   - Query the Reference Librarian FIRST for any information needs
   - DO NOT access knowledge files directly

2. **When encountering unknowns:**
   - Query librarian first
   - If librarian requests exploration, explore the codebase
   - Report findings BACK to librarian (do not add to knowledge files directly)

3. **In their output reports:**
   - Include `librarian_queries` section listing questions asked
   - Include `exploration_reports` section for findings reported back to librarian

---

## Model Configuration

### Default Model

All agents and evaluators default to **`claude-opus-4-5`** for maximum quality reasoning.

The Reference Librarian uses **`gpt-5.2-high-reasoning`** for knowledge queries.

### Available Models

| Model | Best For |
|-------|----------|
| `claude-opus-4-5` | Complex reasoning, high-stakes decisions (default) |
| `claude-sonnet-4-5` | Balanced speed/quality |
| `claude-haiku-4-5` | Fast evaluation, simple tasks |
| `gpt-5.2-high-reasoning` | Knowledge queries (Reference Librarian default) |
| `gpt-5.2`, `gpt-5.1`, `gpt-5` | Alternative options |
| `gpt-5-mini` | Fast/cheap alternative |
| `gemini-3-pro-preview` | Google alternative |

### Recommended Configurations

**Default** (maximum quality):
- All agents: `claude-opus-4-5`
- All evaluators: `claude-opus-4-5`
- Reference Librarian: `gpt-5.2-high-reasoning`

**Speed-optimized** (faster, cheaper):
```yaml
models:
  software_engineer: "claude-sonnet-4-5"
  test_writer: "claude-sonnet-4-5"
  evaluators: "claude-haiku-4-5"
```

**Balanced** (good quality, reasonable speed):
```yaml
models:
  task_generator: "claude-sonnet-4-5"
  software_engineer: "claude-opus-4-5"
  test_writer: "claude-sonnet-4-5"
  evaluators: "claude-sonnet-4-5"
```

---

## Directory Structure

```
{{artifact_root}}{CHANGE-ID}/
│
├── intake/
│   ├── story.yaml          # Normalized story with numbered ACs
│   ├── config.yaml         # Model assignments, run metadata
│   └── constraints.md      # Technical context, examples
│
├── logs/                   # Agent execution logs
│   ├── orchestrator/       # Workflow state transitions
│   ├── reference_librarian/# Knowledge queries and accumulation
│   ├── task_generator/     # Task planning sessions
│   ├── assignment/         # Scheduling sessions
│   ├── software_engineer/  # Implementation sessions
│   ├── unit_test_writer/   # Unit/component test sessions
│   ├── integration_test_writer/ # E2E test sessions
│   ├── qa/                 # QA validation sessions
│   ├── ui_qa/              # UI QA sessions (Playwright-based)
│   └── remediation/        # Remediation session logs
│
├── planning/
│   ├── tasks.yaml          # Broad task plan
│   ├── assignments.yaml    # Execution schedule
│   └── eval_tasks_1.yaml   # Evaluation attempts (if revisions needed)
│
├── execution/
│   ├── UOW-001/
│   │   ├── impl_report.yaml
│   │   ├── unit_test_report.yaml      # From Unit/Component Test Writer
│   │   ├── integration_test_report.yaml # From Integration Test Writer
│   │   └── eval_impl_1.yaml
│   └── UOW-002/
│       └── ...
│
├── feedback/               # Post-QA user feedback
│   ├── feedback.yaml       # User-submitted issues
│   └── remediation_uows.yaml # Remediation work units
│
├── qa/
│   ├── qa_report.yaml
│   ├── ui_qa_report.yaml   # From UI QA Agent (if UI changes)
│   ├── eval_qa_1.yaml
│   ├── eval_ui_qa_1.yaml   # UI QA evaluation (if applicable)
│   └── evidence/
│       ├── screenshots/
│       ├── test_output/
│       └── ui/             # UI QA Playwright evidence
│
├── reviews/
│   └── code_review_findings.yaml
│
└── summary/
    └── final_summary.md

# Knowledge Directory (separate from per-change artifacts):
{{knowledge_root}}
│
├── accumulated-knowledge.md  # Cumulative knowledge (librarian writes)
├── learnings.yaml            # Structured learnings (librarian writes)
├── standing-questions.md     # Unanswered questions (librarian writes)
├── questions.yaml            # Active questions being explored
└── rls-system-architecture.md # System architecture docs
```

---

## Agent Logging

**Every agent produces a log file each time it runs.** Logs are stored in `{CHANGE-ID}/logs/{agent_name}/`.

### Log File Naming

All logs use the format: `{YYYYMMDD_HHMMSS}_{identifier}_session.yaml`

| Agent | Log Location | Example Filename |
|-------|-------------|------------------|
| Orchestrator | `logs/orchestrator/` | `20260127_143000_state_transition.yaml` |
| Reference Librarian | `logs/reference_librarian/` | `20260127_143052_query.yaml` |
| Task Generator | `logs/task_generator/` | `20260127_143500_session.yaml` |
| Assignment | `logs/assignment/` | `20260127_153000_session.yaml` |
| Software Engineer | `logs/software_engineer/` | `20260127_160000_UOW-001_session.yaml` |
| Unit/Component Test Writer | `logs/unit_test_writer/` | `20260127_170000_UOW-001_session.yaml` |
| Integration Test Writer | `logs/integration_test_writer/` | `20260127_173000_UOW-001_session.yaml` |
| QA | `logs/qa/` | `20260127_180000_session.yaml` |
| UI QA (conditional) | `logs/ui_qa/` | `20260127_181500_session.yaml` |

### What Logs Contain

Each log includes:
- **Timestamp and identifiers**: When, which agent, which iteration
- **Input/output artifacts**: What was read, what was written
- **Librarian queries**: Questions asked, confidence received
- **Exploration reports**: Findings reported back to librarian
- **Decisions made**: Key choices with rationale
- **Issues encountered**: Problems and how they were handled

### Using Logs for Debugging

1. **Trace workflow execution**: Follow orchestrator logs to see state transitions
2. **Understand agent decisions**: Each agent logs its reasoning
3. **Debug Reference Librarian**: See what queries got what confidence levels
4. **Track knowledge flow**: See how findings flow back to librarian and into accumulated-knowledge.md

---

## How Agents Communicate

Agents don't call each other directly. The **Orchestrator** mediates all communication:

1. Orchestrator reads the current stage's input artifact
2. Orchestrator dispatches to the appropriate agent with context
3. Agent produces output artifact (yaml) → writes to artifact directory
4. Evaluator reads output → assesses against rubric → writes evaluation
5. Orchestrator reads evaluation:
   - **Pass** → advance to next stage
   - **Revise** → feed evaluation feedback back to agent
   - **Escalate** → pause and notify human

```
┌─────────────┐
│ Orchestrator│
└──────┬──────┘
       │
       ├──dispatch──▶ [Agent] ──writes──▶ artifact.yaml
       │                                       │
       ├──dispatch──▶ [Evaluator] ◀──reads─────┘
       │                   │
       │◀──────────────────┘ (evaluation result)
       │
       └── decide: pass | revise | escalate
```

---

## Evaluator-Optimizer Loop

Each stage runs through this control loop:

1. **Generation**: Agent produces initial artifact
2. **Evaluation**: Evaluator checks against rubric + programmatic checks
3. **Refinement**: If failed, agent revises based on actionable feedback
4. **Iteration**: Repeat until pass or stopping criteria

### Stopping Criteria

| Criteria | Action |
|----------|--------|
| Quality gate pass | Advance to next stage |
| Max iterations (2-3) | Stop and escalate |
| Token budget exceeded | Stop and escalate |
| Similarity plateau | Stop and escalate |

---

## Escalation Handling

The workflow pauses and notifies you when:

- **Ambiguous requirements**: ACs are unclear or contradictory
- **Breaking change detected**: Code changes affect existing contracts
- **Max iterations reached**: Agent can't satisfy evaluator after 2-3 attempts
- **Spec clarification needed**: Implementation requires human decision

When escalated, check:
1. `config.yaml` → `run_metadata.status` shows `escalated`
2. Latest evaluation file shows `escalation_recommendation`
3. Address the issue, then resume the workflow

---

## Agent Prompts Reference

| File | Agent | Purpose |
|------|-------|---------|
| `00-reference-librarian-agent-prompt.md` | Reference Librarian | **Mandatory first contact** for all knowledge queries; accumulates knowledge |
| `01-orchestrator-agent-prompt.md` | Orchestrator | Controls workflow, manages state |
| `02-task-generator-agent-prompt.md` | Task Generator | Creates broad task plan |
| `04-assignment-agent-prompt.md` | Assignment | Schedules execution order |
| `05-software-engineer-agent-prompt.md` | Software Engineer | Implements code changes |
| `06-test-writer-agent-prompt.md` | Unit/Component Test Writer | Writes unit & component tests (co-located) |
| `06b-integration-test-writer-agent-prompt.md` | Integration Test Writer | Writes E2E tests (separate app) |
| `07-qa-agent-prompt.md` | QA | Validates acceptance criteria |
| `07b-ui-qa-agent-prompt.md` | UI QA (conditional) | Validates UI consistency using Playwright CLI |
| `08-task-plan-evaluator-prompt.md` | Task Plan Evaluator | Evaluates task plans |
| `10-assignment-evaluator-prompt.md` | Assignment Evaluator | Evaluates schedules |
| `11-implementation-evaluator-prompt.md` | Implementation Evaluator | Evaluates code changes |
| `12-test-evaluator-prompt.md` | Test Evaluator | Evaluates test coverage |
| `13-qa-evaluator-prompt.md` | QA Evaluator | Evaluates QA reports |
| `14-ui-qa-evaluator-prompt.md` | UI QA Evaluator | Evaluates UI consistency reports |

### Test Writer Agents

Testing is split between two specialized agents:

| Agent | Test Types | Location |
|-------|------------|----------|
| **Unit/Component Test Writer** | Unit tests, component tests | Co-located with code (`src/components/__tests__/`, `src/services/__tests__/`) |
| **Integration Test Writer** | E2E integration tests | Separate app: `{{e2e_tests_root}}` |

---

## Reference Librarian Agent

The **Reference Librarian** is the **mandatory first point of contact** for all knowledge queries. It holds all accumulated knowledge and manages the knowledge lifecycle. All agents **must query it first** before doing any codebase exploration. It runs on `gpt-5.2-high-reasoning` for intelligent query handling.

### Why?
- **Reduces context bloat**: Agents only receive relevant answers, not entire files
- **Mandatory gateway**: ALL knowledge access goes through the librarian
- **Knowledge accumulation**: Librarian adds agent findings to persistent knowledge
- **Confidence levels**: Tells agents when to explore vs. use answers directly
- **Standing questions**: Tracks unanswered questions for future resolution

### Query-First Workflow

1. Agent has a question
2. Agent queries the Reference Librarian (REQUIRED FIRST STEP)
3. Librarian responds with `confidence: full | partial | none`
4. **If `full`**: Agent uses the answer directly
5. **If `partial` or `none`**: Agent explores based on librarian hints, then **reports back**
6. **Agent reports findings**: Librarian adds to `accumulated-knowledge.md`
7. **If answer not found**: Librarian adds query to `standing-questions.md`

### Example Query - Has Answer

```
Query: "What tooltip patterns exist in this codebase?"
```

Response:

```yaml
answer: "Tooltips use PrimeNG v19's pTooltip directive with tooltipPosition attribute..."
  source_files: ["accumulated-knowledge.md"]
  confidence: "full"
  requires_exploration: false
```

### Example Query - Needs Exploration

```
Query: "Where is the PersonService repository pattern implemented?"
```

Response:

```yaml
answer: null
  confidence: "none"
  requires_exploration: true
  exploration_request: {
    action: "explore_and_report"
    hint: "Search for PersonService in the codebase and trace its database calls"
    report_format: "Provide: file paths, code patterns, and a summary of findings"
```

### What It Knows

All files in `agent-reference/knowledge/`:
- `accumulated-knowledge.md` - Cumulative discovered knowledge
- `learnings.yaml` - Structured learnings with metadata
- `standing-questions.md` - Unanswered questions needing attention
- `questions.yaml` - Active questions being explored
- `rls-system-architecture.md` - System architecture documentation

---

## Post-QA Feedback and Remediation

After QA passes, the workflow enters **PostQAReview** state and waits for your review.

### Providing Feedback

When you review completed work and find issues, provide feedback using this template:

```yaml
feedback:
  issues:
    - description: "Tooltip link opens in same tab instead of new tab"
      type: "bug"               # bug|missing_feature|test_gap|ux_issue|spec_clarification
      severity: "high"          # critical|high|medium|low
      affected_acs: ["AC-003"]
      reproduction_steps: "1. Hover tooltip 2. Click person ID 3. Opens in same tab"
      expected_behavior: "Should open in new tab"
      
  general_notes: "Everything else looks good"
```

### Issue Types

| Type | Routed To | What It Means |
|------|-----------|---------------|
| `bug` | Software Engineer | Code doesn't work as expected |
| `missing_feature` | Task Generator (new UoW) | Functionality not implemented |
| `test_gap` | Test Writer | Missing test coverage |
| `ux_issue` | Software Engineer | UI/UX problem |
| `spec_clarification` | You (escalated) | Requirements unclear |

### Remediation Flow

```
You submit feedback
       ↓
Orchestrator creates remediation UoWs
       ↓
Routes each issue to appropriate agent
       ↓
Agent fixes + Evaluator validates
       ↓
You verify fixes
       ↓
Approve or submit more feedback
       ↓
Repeat until approved → Complete
```

### Approving Completion

When satisfied with the work, reply with:
- "approved"
- "lgtm"
- "looks good"

The workflow will transition to **Complete** state.

### Resuming After Feedback

If the workflow was stopped and you need to resume at the feedback stage:

```yaml
resume:
  enabled: true
  resume_at: "POST_QA_REVIEW"
  feedback_file: "feedback/feedback.yaml"
```

---

## Troubleshooting

### Workflow stuck in a loop
Check the evaluation files (`eval_*_k.yaml`) for:
- Repeated issues that aren't being addressed
- Conflicting feedback
- Missing context the agent needs

### Tests failing repeatedly
1. Check `execution/{UOW-ID}/logs/` for Jest/Cypress output
2. Review `eval_impl_*.yaml` for implementation evaluator feedback
3. Consider if the AC is actually testable as written

### Agent producing invalid YAML
The orchestrator will re-prompt for schema-compliant output. If persistent:
1. Check if the story input has unusual formatting
2. Simplify complex ACs into smaller pieces
3. Try a different model for that agent

---

## Example Workflow Run

```bash
# 1. Start workflow
--change-id 4729040 \
--story-input ./story-4729040.txt \
--code-repo ~/projects/my-app

# 2. Orchestrator creates:
#    - intake/story.yaml (6 ACs normalized)
#    - intake/config.yaml
#    - intake/constraints.md

# 3. Task Generator produces tasks.yaml (4 tasks)
#    - Task Plan Evaluator: PASS

# 4. Assignment produces assignments.yaml (using Work-Decomposer-Output.md)
#    - Assignment Evaluator: PASS

# 5. Execution loop (for each UoW):
#    - Software Engineer implements → Jest passes
#    - Implementation Evaluator: PASS
#    - Test Writer adds tests → Cypress passes
#    - Test Evaluator: PASS

# 7. QA validates all ACs
#    - QA Evaluator: PASS
#    - Final summary written to summary/final_summary.md

# Done! All artifacts in Orchestrated-agent-work/4729040/
```
