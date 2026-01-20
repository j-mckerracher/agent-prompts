# AI Development Team — Overview and Integration Guide

## Team Composition

| Agent | Primary Role | Execution Mode | Managed By | Model |
|-------|-------------|----------------|------------|-------|
| **Work Decomposer** | Break micro-plan into atomic UoWs | Standalone | Human | — |
| **Workstream Orchestrator (W-O)** | Coordinate entire workstream, spawn UOWOs | Autonomous | Human | Sonnet |
| **Unit-of-Work Orchestrator (UOWO)** | Execute single UoW lifecycle, manage subagents | Subagent | W-O | Sonnet |
| **Work Assigner** | Select and assign UoWs to SE | Subagent | UOWO | Haiku |
| **Software Engineer** | Implement assigned UoW | Subagent | UOWO | Haiku |
| **Code Reviewer** | Review implementation quality | Subagent | UOWO | Haiku |
| **QA Engineer** | Validate functionality and write tests | Subagent | UOWO | Haiku |
| **DevOps** | Deploy, monitor, maintain CI/CD | Subagent | UOWO | Haiku |

## Workflow Diagram

```
┌─────────────────┐
│  Micro-Level    │
│     Plan        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Work Decomposer │ ──── Produces: UoW Decomposition JSON
└────────┬────────┘
         │
         ▼
┌───────────────────────────────────────────────────────────┐
│           WORKSTREAM ORCHESTRATOR (W-O)                    │
│                     [Sonnet]                               │
│                                                            │
│  • Manages entire workstream (all UoWs)                   │
│  • Builds dependency graph, identifies parallelization    │
│  • Spawns UOWO agents for each ready UoW                  │
│  • Tracks workstream-level progress                       │
│  • Escalates when workstream is blocked                   │
└────────┬──────────────────────────────────────────────────┘
         │
         │ For each ready UoW, spawns a UOWO:
         │
         ▼
┌───────────────────────────────────────────────────────────┐
│         UNIT-OF-WORK ORCHESTRATOR (UOWO)                   │
│                     [Sonnet]                               │
│                                                            │
│  • Manages single UoW lifecycle                           │
│  • Spawns subagents in sequence                           │
│  • Handles rejection loops                                │
│  • Returns completion status to W-O                       │
└────────┬──────────────────────────────────────────────────┘
         │
         │ Spawns subagents with [Haiku] model:
         │
         ▼
┌─────────────────┐
│  Work Assigner  │ ──── Returns: Assignment file path + status
│    [Haiku]      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    Software     │ ──── Returns: Diff, work log, test results + status
│    Engineer     │
│    [Haiku]      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Code Reviewer  │ ──── Returns: Review report, decision (approve/reject)
│    [Haiku]      │      │
└────────┬────────┘      │ If rejected: UOWO loops back to SE
         │                │
         ▼                ▼
┌─────────────────┐  [Rejection Loop: max 3 cycles]
│   QA Engineer   │ ──── Returns: QA report, decision (approve/reject)
│    [Haiku]      │      │
└────────┬────────┘      │ If rejected: UOWO loops back to SE
         │                │
         ▼                ▼
┌─────────────────┐  [Rejection Loop: max 2 cycles]
│     DevOps      │ ──── Returns: Deployment report, status
│    [Haiku]      │
└────────┬────────┘
         │
         ▼
    Production
```

## Shared Conventions

### Artifact Naming
- UoW IDs: `U01`, `U02`, ... `Unn`
- Assignment files: `UoW-<UNIT_ID>-Assignment.md`
- SE Work Logs: `SE-Log-<UNIT_ID>.md`
- Review Reports: `Review-<UNIT_ID>.md`
- QA Reports: `QA-Report-<UNIT_ID>.md`

### Status Values
All agents use consistent status tracking:
- `pending` — Not yet started
- `in_progress` — Currently being worked
- `blocked` — Awaiting resolution
- `ready` — Ready for handoff
- `approved` — Passed review/QA
- `rejected` — Failed review/QA, needs rework
- `done` — Completed and verified

### Shared Constraints
All implementation agents adhere to:
- **File limit:** ≤5 files per UoW
- **LOC limit:** ≤400 lines of code per UoW
- **Step limit:** ≤10 implementation steps per UoW
- **No secrets:** Use placeholders like `{{API_KEY}}`
- **No commits:** Unless explicitly authorized by orchestrator

### Escalation Chain
```
Subagent (02-06) → UOWO → W-O → Human Orchestrator
                    ↓      ↓
                    ↓    Decomposer (01) (if UoW split needed)
                    ↓
              Handles rejection loops
```

**UOWO handles:**
- Rejection loops (automatic retry with limits)
- Individual UoW blockers

**W-O handles:**
- Workstream-level coordination
- Parallelization decisions
- Blocker escalation to human when all paths blocked
- UoW splitting decisions (escalate to Decomposer or human)

### Shared Resources
All agents reference:
- `04-Agent-Reference-Files/Code-Standards.md`
- `04-Agent-Reference-Files/Common-Pitfalls-to-Avoid.md`
- Project-specific micro-level plan

## Progress Tracking Protocol

All agents maintain two levels of progress tracking to provide visibility into implementation status:

### Shared Progress Files

1. **Project Progress**: `progress-tracking/project-progress.md`
   - Tracks all 10 workstreams (W1-W10)
   - Updated by: Work Decomposer (initialization), QA Engineer (completion counts)
   - Contains: Overall completion %, active workstreams, blockers

2. **Workstream Progress**: `progress-tracking/W{N}-progress.md`
   - Tracks all UoWs in active workstream
   - Updated by: All agents at key transition points
   - Contains: UoW statuses, dependencies, detailed history

### Status Lifecycle

```
pending → assigned → in_progress → ready_for_review →
code_review_approved → qa_approved → done ✓

         ↓ (temporary states)
    blocked ──> (returns to in_progress when resolved)
         ↓
code_review_rejected/qa_rejected ──> (returns to in_progress for rework)
```

### Update Responsibilities

| Agent | Update Trigger | Status Transition | Helper Script |
|-------|---------------|-------------------|---------------|
| Work Decomposer | After decomposition | null → pending | `init-workstream.sh` |
| Work Assigner | Creating assignment | pending → assigned | `update-status.sh` |
| Software Engineer | Start work | assigned → in_progress | `update-status.sh` |
| Software Engineer | Hit blocker | in_progress → blocked | `mark-blocked.sh` |
| Software Engineer | Complete impl | in_progress → ready_for_review | `update-status.sh` |
| Code Reviewer | Approve | ready_for_review → code_review_approved | `update-status.sh` |
| Code Reviewer | Reject | ready_for_review → code_review_rejected | `update-status.sh` |
| QA Engineer | Approve | code_review_approved → done | `update-status.sh` |
| QA Engineer | Reject | code_review_approved → qa_rejected | `update-status.sh` |

### Progress Update Commands

All agents use helper scripts located in `progress-tracking/scripts/`:

**Update UoW Status**:
```bash
./progress-tracking/scripts/update-status.sh <WORKSTREAM_ID> <UNIT_ID> <NEW_STATUS> [NOTE]
```

**Mark as Blocked**:
```bash
./progress-tracking/scripts/mark-blocked.sh <WORKSTREAM_ID> <UNIT_ID> "<DESCRIPTION>" "<RESOLUTION>"
```

**Check All Blockers** (for human orchestrator):
```bash
./progress-tracking/scripts/check-blockers.sh
```

### Blocker Tracking Format

When a UoW is blocked, the blocker must include:
- **Description**: What is blocking progress
- **Blocked Since**: ISO timestamp
- **Resolution Required**: What needs to happen to unblock
- **Escalated To**: Who can unblock (usually "Human Orchestrator")

### QA Engineer Responsibilities

QA Engineer has the additional responsibility of updating project-level progress:
- When marking UoWs as `done`, workstream completion percentage is automatically updated
- When all UoWs in a workstream are done, run:
  ```bash
  ./progress-tracking/scripts/complete-workstream.sh <WORKSTREAM_ID>
  ```

## Handoff Protocols

**Note:** The workflow has two orchestration tiers:
- **W-O** manages workstream-level coordination (spawning UOWOs, tracking progress)
- **UOWO** manages individual UoW lifecycles (spawning agents 02-06, handling rejection loops)

### Work Decomposer → W-O (Workstream Orchestrator)
- **Artifact:** `Work-Decomposer-Output.md` (JSON structure)
- **Required fields:** `unit_id`, `dependencies`, `files_to_edit_or_create`, `success_criteria`, `est_impl_tokens`
- **W-O action:** Load decomposition, build dependency graph, identify ready UoWs

### W-O → UOWO (Unit-of-Work Orchestrator)
- **Input:** `{unit_id, workstream_id, decomposition_path}`
- **Output:** `{status, unit_id, final_state, rejection_cycles, blocker, artifacts}`
- **W-O action:** Track completion, identify newly unblocked UoWs, continue or escalate

### UOWO → Work Assigner
- **Input:** `{decomposition_json_path, unit_id, workstream_id}`
- **Output:** `{status, assignment_file_path, unit_id, blocker}`
- **UOWO action:** If status=ready, spawn SE; if blocked, return to W-O

### UOWO → Software Engineer
- **Input:** `{assignment_file_path, workstream_id, unit_id, context}`
- **Output:** `{status, diff_path, work_log_path, test_results_path, build_results_path, blocker}`
- **UOWO action:** If status=ready_for_review, spawn Code Reviewer; if blocked, return to W-O

### UOWO → Code Reviewer
- **Input:** `{assignment_file_path, diff_path, work_log_path, test_results_path, workstream_id, unit_id}`
- **Output:** `{status, decision, review_report_path, issues}`
- **UOWO action:** If approved, spawn QA; if rejected, re-spawn SE with issues (max 3 cycles)

### UOWO → QA Engineer
- **Input:** `{assignment_file_path, review_report_path, diff_path, work_log_path, workstream_id, unit_id}`
- **Output:** `{status, decision, qa_report_path, bugs}`
- **UOWO action:** If approved, spawn DevOps; if rejected, re-spawn SE with bugs (max 2 cycles)

### UOWO → DevOps
- **Input:** `{qa_report_path, unit_id, environment, auto_deploy}`
- **Output:** `{status, deployment_report_path, deployed_version, error}`
- **UOWO action:** If success, mark UoW as done and return success to W-O; if failed, return failure to W-O

## Communication Protocol

### W-O → UOWO Communication

**W-O → UOWO:**
- Spawns UOWO with `model: "sonnet"` via Task tool
- Provides unit_id, workstream_id, and decomposition path
- May spawn multiple UOWOs in parallel for independent UoWs

**UOWO → W-O:**
- Returns structured JSON with status, final_state, rejection_cycles, artifacts
- Returns blocker information if unable to complete UoW
- W-O processes result and identifies next actions

### UOWO → Subagent Communication

**UOWO → Subagent (02-06):**
- Spawns subagent with `model: "haiku"` via Task tool
- Provides file paths to required artifacts
- Includes context for rework (if applicable)

**Subagent → UOWO:**
- Returns structured JSON output with status, artifact paths, and results
- Returns blocker information if stuck
- Does NOT directly communicate with other subagents or W-O

### Subagent-Artifact Communication

Subagents communicate via structured Markdown artifacts saved to the file system:
1. **Reads** artifacts specified in input parameters
2. **Produces** artifacts (assignments, diffs, reports) saved to designated paths
3. **Returns** artifact paths to Orchestrator
4. **Logs** all decisions and actions in work logs

### Escalation Format (Subagents → UOWO → W-O)

Subagents return structured blocker information to UOWO:
```json
{
  "status": "blocked",
  "blocker": {
    "description": "1-2 sentence summary",
    "resolution_required": "What needs to happen",
    "affected_fields": ["list", "of", "fields"],
    "options": ["A) option 1", "B) option 2"],
    "recommendation": "Option A/B with rationale"
  }
}
```

UOWO propagates blockers to W-O in its return:
```json
{
  "status": "blocked",
  "unit_id": "U05",
  "final_state": "blocked",
  "blocker": {
    "description": "...",
    "resolution_required": "...",
    "agent": "Software Engineer"
  }
}
```

W-O creates formal escalation reports for human review when workstream progress is blocked.

## Agent Prompt Files

| Agent | Prompt Location | Execution Mode | Model |
|-------|----------------|----------------|-------|
| Workstream Orchestrator (W-O) | `agent-prompts/00-Workstream-Orchestrator-Agent.md` | Autonomous coordinator | Sonnet |
| Unit-of-Work Orchestrator (UOWO) | `agent-prompts/00-Orchestrator-Agent.md` | Subagent (spawned by W-O) | Sonnet |
| Work Decomposer | `agent-prompts/01-Work-Decomposer-Agent.md` | Standalone (run once per workstream) | — |
| Work Assigner | `agent-prompts/02-Work-Assigner-Agent.md` | Subagent (spawned by UOWO) | Haiku |
| Software Engineer | `agent-prompts/03-Software-Engineer-Agent.md` | Subagent (spawned by UOWO) | Haiku |
| Code Reviewer | `agent-prompts/04-Code-Reviewer-Agent.md` | Subagent (spawned by UOWO) | Haiku |
| QA Engineer | `agent-prompts/05-QA-Engineer-Agent.md` | Subagent (spawned by UOWO) | Haiku |
| DevOps | `agent-prompts/06-DevOps-Agent.md` | Subagent (spawned by UOWO) | Haiku |

### Usage Pattern

1. **Human** runs **Work Decomposer (01)** with micro-level plan → produces decomposition JSON
2. **Human** starts **Workstream Orchestrator (W-O)** with workstream ID → autonomous execution begins
3. **W-O** spawns **UOWO** agents for each ready UoW (respecting dependencies)
4. **UOWO** spawns subagents **02-06** in sequence for the assigned UoW
5. **UOWO** returns completion status to **W-O**
6. **W-O** continues until all UoWs complete or workstream is blocked
7. **W-O** escalates to **Human** only when all progress paths are blocked
