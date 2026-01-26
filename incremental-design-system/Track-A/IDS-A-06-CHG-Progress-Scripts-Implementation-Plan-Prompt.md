# IDS-A-06 — CHG Progress Scripts Implementation Plan (Incremental Design System / Track A)

## Persona
You are a senior tooling engineer and delivery lead. You produce an implementation-ready plan that can be decomposed into atomic UoWs under strict limits.

## Purpose
Implement Track A Option B by adding **parallel progress-tracking scripts** that support CHG workstreams without modifying existing W* scripts. 

This plan must be:
- minimal-risk (no regression to W* flows)
- execution-ready (clear file list and tests)
- aligned to the existing orchestration state machine semantics
- archived as long-lived documentation

---

## Policy Constraints (Authoritative)
1. Option B is chosen: **add new CHG scripts**, do not change W* scripts. 
2. State machine semantics must remain consistent with orchestration:
   `pending → assigned → in_progress → ready_for_review → code_review_approved → qa_approved → deployed → done`
   plus `blocked`, `code_review_rejected`, `qa_rejected`. 
3. Long-lived docs are required. 
4. No secrets/PII in logs or progress files. 
5. Keep changes small and focused; avoid unrelated refactors. 

---

## Path Conventions (Team‑Portable; Authoritative)

### Repo‑Relative Paths
All repo paths are expressed relative to `agent-prompts/`:
- Scripts: `../progress-tracking/scripts/`
- Progress files: `../progress-tracking/`
- Project progress: `../progress-tracking/project-progress.md` 

### Agent Reference Files
- `../agent-prompts/reference-files/Files-to-Keep-Updated.md` 
- `../agent-prompts/reference-files/Code-Standards.md` 
- `../agent-prompts/reference-files/Common-Pitfalls-to-Avoid.md` 

---

## Inputs (Provided)
You will be given:
1) The compatibility spec produced by IDS-A-05:
- `../progress-tracking/CHG-progress-script-compatibility-spec.md` 

2) The current scripts folder (if present):
- `../progress-tracking/scripts/` 

3) Current project aggregation:
- `../progress-tracking/project-progress.md` 

If scripts folder is missing, you must plan to create it.

---

## Logging and Traceability (Portable)
This tooling work should be tracked like any other story.
Your plan must assume a Change ID exists for the tooling story, e.g.:
- `CHG-YYYYMMDD-chg-progress-scripts`

Logs must go under:
- `{ORCHESTRATED_AGENT_WORK_ROOT}/{CHANGE-ID}/...` (portable root semantics) 

---

## Your Task (IDS-A-06)

### Step 1 — Restate Option B Scope
Confirm that the plan will:
- Create new scripts for CHG workstreams
- Leave existing W* scripts untouched
- Ensure incremental orchestration can call the CHG scripts 

### Step 2 — Define Script Set and Interfaces
The plan MUST specify the new scripts (names are fixed for Option B):

- `../progress-tracking/scripts/init-change.sh`
- `../progress-tracking/scripts/update-change-status.sh`
- `../progress-tracking/scripts/mark-change-blocked.sh`
- `../progress-tracking/scripts/complete-change.sh`
- `../progress-tracking/scripts/update-project-for-change.sh` (if needed)

Each must define:
- required args
- optional args (e.g., NOTE)
- expected files touched:
  - `../progress-tracking/{CHANGE-ID}-progress.md`
  - `../progress-tracking/project-progress.md` 

### Step 3 — Define File Format Requirements
The plan must align to IDS-A-04 progress file schema:
- YAML frontmatter includes change_id, log_root, completion counts
- Units list supports the orchestration states above
- History appends are timestamped
- No secrets/PII 

### Step 4 — Define Backward Compatibility Behavior
- New scripts only affect CHG progress files
- W* scripts remain as-is
- Project progress aggregation must safely include both W* and CHG* entries without deleting existing content 

### Step 5 — Testing & Verification Plan
Define a test strategy (script-level verification) including:
- create progress file
- update a unit status
- mark blocked
- complete change when all units done
- update project-progress entry
- failure cases: missing args, missing progress file, malformed YAML
No actual code execution here — only define the plan. 

### Step 6 — Decompose into UoWs (Critical)
Produce UoW boundaries sized to:
- ≤ 5 files changed per UoW
- ≤ 400 LOC per UoW
- ≤ 10 steps per UoW
- one primary feature per unit 

### Step 7 — Persist Plan Artifact (Long‑Lived)
Write your plan to:
- `../progress-tracking/CHG-progress-scripts-implementation-plan.md`

---

## Deliverable Template (MUST USE)

```markdown
---
tags: [progress, incremental, plan, option-b, track-a]
created: "<YYYY-MM-DD>"
status: "draft|approved"
source_spec: "[[../progress-tracking/CHG-progress-script-compatibility-spec]]"
---

# CHG Progress Scripts — Implementation Plan (Option B)

## 1. Scope
### In Scope
- Add CHG-only scripts:
  - init-change.sh
  - update-change-status.sh
  - mark-change-blocked.sh
  - complete-change.sh
  - update-project-for-change.sh (if needed)
- Update project-progress aggregation to include CHG entries without disturbing W entries

### Out of Scope
- Modifying existing W* scripts
- Refactoring orchestration beyond updating incremental prompts to call CHG scripts

## 2. Script Interfaces (Contract)
Document args and expected behavior for each script.

## 3. File Formats
- CHG progress file format: `../progress-tracking/{CHANGE-ID}-progress.md` (align to IDS-A-04)
- Project progress update rules: append/update "Incremental Changes" section

## 4. Work Breakdown (Constrained)
List tasks in order.

## 5. Proposed Units of Work (UoWs)
### U01 — Add init-change.sh + minimal progress file creation
- Goal:
- Files:
- AC:
- Test plan:

### U02 — Add update-change-status.sh
...

### U03 — Add mark-change-blocked.sh
...

### U04 — Add update-project-for-change.sh (or integrate into init)
...

### U05 — Add complete-change.sh
...

## 6. Verification Plan
- Checklist of manual/script-run validations
- Failure mode validations

## 7. Risks & Rollback
- Risks:
- Mitigations:
- Rollback plan:
```

---

## Output Contract (Return)

Return JSON:

```json
{
  "status": "ready|blocked",
  "artifact": "../progress-tracking/CHG-progress-scripts-implementation-plan.md",
  "option": "B",
  "blocker": null
}
```

---

## Guardrails

- Do not implement code. This is planning only.
- Do not modify existing W* scripts.
- Keep UoWs within constraints; split if needed.
- No secrets or PII in plan artifacts.
