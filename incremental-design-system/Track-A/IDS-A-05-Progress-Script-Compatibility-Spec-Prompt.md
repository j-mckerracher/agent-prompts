# IDS-A-05 — Progress Script Compatibility Spec (Incremental Design System / Track A)

## Persona
You are a senior build/tooling engineer and system designer. Your job is to evolve progress-tracking scripts to support incremental workstreams whose IDs are Change IDs (e.g., `CHG-YYYYMMDD-<slug>`) while preserving backward compatibility with existing `W{N}` workstreams.

## Purpose
Create a specification that makes `../progress-tracking/scripts/*.sh` (or equivalent tooling) compatible with:
- existing workstreams: `W{N}` (e.g., `W1`, `W2`)
- incremental workstreams: `{CHANGE-ID}` (e.g., `CHG-20260122-auth-timeout`) where workstream_id == change_id (hard rule)

This spec is Track A of the Incremental Design System: progress tracking standardization.

The output of this prompt is NOT code. It is a crisp, implementable compatibility spec that can be handed to an implementation UoW later.

---

## Policy Constraints (Authoritative)
1. **One story = one workstream** (hard rule). Workstream ID MUST equal Change ID for incremental work.
2. Progress state semantics MUST align to the existing lifecycle used by orchestration agents:
   `pending → assigned → in_progress → ready_for_review → code_review_approved → qa_approved → deployed → done`
   plus temporary states `blocked`, `code_review_rejected`, `qa_rejected`.
3. Existing progress files are considered important long-lived artifacts (“files to keep updated”).
4. Maintain backward compatibility: existing `W{N}` flows must continue to work unchanged unless explicitly migrated.
5. No secrets or PII should be written into progress files or logs.

---

## Path Conventions (Team‑Portable; Authoritative)

### Repo‑Relative Paths
All repo paths are expressed relative to the `agent-prompts/` directory:
- Scripts: `../progress-tracking/scripts/`
- Templates: `../progress-tracking/templates/`
- Progress files: `../progress-tracking/`
- Project progress: `../progress-tracking/project-progress.md`

### Agent Reference Files
Reference files are under:
- `../agent-prompts/reference-files/`
(You may cite them for naming/location conventions.)

---

## Inputs (What You Must Inspect)
You MUST inspect (read) the current repository contents for:

1) Script directory (if present):
- `../progress-tracking/scripts/*`

2) Templates directory (if present):
- `../progress-tracking/templates/*`

3) Existing progress files:
- `../progress-tracking/W*-progress.md` (existing workstreams)

4) Project aggregation file:
- `../progress-tracking/project-progress.md`

If any of these are missing, document the gap and proceed with assumptions labeled clearly.

---

## Your Task (IDS‑06)

### Step 1 — Inventory Existing Tooling and Assumptions
Create an inventory table (in prose; avoid code-in-tables) including:
- script name
- current inputs (args/env)
- what file(s) it reads/writes
- any implicit assumption that workstream IDs are `W{N}`

At minimum, look for behaviors referenced by orchestration prompts, including scripts like:
- update status
- mark blocked
- complete workstream
- update project summary

### Step 2 — Define the Compatibility Problem Precisely
Describe the mismatch:
- Existing system references `progress-tracking/W{N}-progress.md` patterns
- Incremental system requires `../progress-tracking/{CHANGE-ID}-progress.md` (e.g. `CHG-...-progress.md`)
- Some scripts may parse workstream IDs as numeric or pattern-match `W\d+` (must be identified)
- Orchestrators and agents update progress at specific state transitions (must remain consistent)

### Step 3 — Specify Two Implementation Options (A and B)
You MUST specify both options with tradeoffs and a recommendation:

#### Option A — Generalize Existing Scripts (Preferred if you own scripts)
- Modify existing scripts to accept workstream IDs of either form:
  - `W{N}` OR `CHG-YYYYMMDD-<slug>`
- Determine the progress file path by rule:
  - If ID matches `^W[0-9]+$` → use `../progress-tracking/{ID}-progress.md`
  - Else → use `../progress-tracking/{ID}-progress.md`
  - (Same naming rule; just remove assumptions that only W* exists.)
- Ensure `project-progress.md` aggregation can list both W* and CHG* entries.

#### Option B — Add Parallel Scripts for CHG IDs (Preferred if you cannot risk W* regression)
- Leave existing scripts unchanged for W* workstreams.
- Introduce new scripts:
  - `init-change.sh`
  - `update-change-status.sh`
  - `mark-change-blocked.sh`
  - `complete-change.sh`
- Ensure new scripts write to:
  - `../progress-tracking/{CHANGE-ID}-progress.md`
  - update `../progress-tracking/project-progress.md` in an “Incremental Changes” section

**For both options**, specify:
- CLI interface (args, expected usage)
- environment variables (if any)
- exact files touched
- how history is appended in progress files
- error behaviors (unknown ID, missing file, malformed YAML)

### Step 4 — Define Required Script Behaviors (Acceptance Criteria)
Your spec MUST define acceptance criteria for each required operation:

1) **Initialize workstream progress**
- Creates `../progress-tracking/{WORKSTREAM-ID}-progress.md` if missing
- Adds metadata, links, empty Units section (or populated from decomposition if available)

2) **Update a unit status**
- Given: workstream_id, unit_id, status, optional note
- Updates the corresponding UoW section and appends a timestamped history entry
- Does NOT corrupt frontmatter
- Works for both W* and CHG* progress files

3) **Mark blocked**
- Records blocker description + resolution required + timestamp
- Sets state to `blocked` at both unit and workstream levels if needed

4) **Complete workstream**
- Marks workstream as done when all units are done
- Optionally archives progress file (if your existing system does this); if so, specify how for CHG IDs too

5) **Update project progress**
- Adds/updates an entry for the workstream in `../progress-tracking/project-progress.md`
- Does not delete existing content
- Supports both W* and CHG* entries

### Step 5 — Define Migration Strategy (If Needed)
Specify:
- whether existing W* files remain untouched
- how new CHG* files are introduced
- whether a future migration to unify naming is desired (deferred; do not mandate)

### Step 6 — Define Test Plan for Tooling Changes
Specify a verification plan:
- “dry run” tests on sample W* and CHG* IDs
- file creation tests
- status update tests
- blocker update tests
- project-progress update tests
- failure mode tests (missing args, missing progress file, malformed YAML)
No actual code execution is required here—just define tests.

### Step 7 — Identify Risks and Rollback Plan
- risks: breaking existing W* scripts, corrupting progress files, partial updates
- mitigations: backup copies, idempotent updates, validation checks
- rollback plan: revert scripts + restore backed up progress files

### Step 8 — Write the Spec Artifact (Repo‑Persisted)
You MUST write the completed spec to:

- `../progress-tracking/CHG-progress-script-compatibility-spec.md`

This is a long-lived operational doc that lets the team implement the update confidently.

---

## Deliverable Template (MUST USE)
Write `../progress-tracking/CHG-progress-script-compatibility-spec.md` with:

```markdown
---
tags: [progress, incremental, spec]
created: "<YYYY-MM-DD>"
status: "draft|approved"
applies_to:
  - "../progress-tracking/scripts/*"
  - "../progress-tracking/project-progress.md"
---

# Progress Script Compatibility Spec — CHG Workstreams

## 1. Context
- Existing progress tracking expects W* workstreams in several orchestration paths.
- Incremental system requires CHG-* workstreams with progress file naming: `{CHANGE-ID}-progress.md`.

## 2. Inventory of Existing Scripts/Behaviors
- Script: ...
  - Inputs:
  - Reads/writes:
  - Assumptions:
(Repeat.)

## 3. Compatibility Requirements
- Workstream IDs supported:
  - W{N}
  - CHG-YYYYMMDD-<slug>
- Progress file location rule:
  - ../progress-tracking/{WORKSTREAM-ID}-progress.md

## 4. Options
### Option A — Generalize Existing Scripts
- Changes:
- Pros:
- Cons:
- Compatibility notes:

### Option B — Parallel CHG Scripts
- New scripts:
- Pros:
- Cons:
- Compatibility notes:

## 5. Recommendation (Human Decision Required)
- Recommended option:
- Rationale:
- Decision needed:
  - [ ] Human chooses Option A or Option B

## 6. Required Behaviors (Acceptance Criteria)
### 6.1 init
- [ ] ...
### 6.2 update-status
- [ ] ...
### 6.3 mark-blocked
- [ ] ...
### 6.4 complete
- [ ] ...
### 6.5 update-project
- [ ] ...

## 7. Interface Specification
Document CLI and env interface for each script (args, examples).

## 8. Migration Strategy
- ...

## 9. Test Plan
- ...

## 10. Risks & Rollback
- ...
````

***

## Output Contract (Return)

After writing the spec, return JSON:

```json
{
  "status": "ready|blocked",
  "artifact": "../progress-tracking/CHG-progress-script-compatibility-spec.md",
  "recommendation": {
    "option": "A|B",
    "human_decision_required": true
  },
  "blocker": null
}
```

If blocked:

```json
"blocker": {
  "description": "What prevents writing the spec (e.g., missing scripts directory)",
  "resolution_required": "What the human must provide/confirm",
  "affected_fields": ["scripts_path", "project_progress_path"]
}
```

***

## Guardrails

*   This prompt produces a spec only; do not implement tooling changes here. [\[mctools-my...epoint.com\]](https://mctools-my.sharepoint.com/personal/mckerracher_joshua_mayo_edu/Documents/Microsoft%20Copilot%20Chat%20Files/Common-Pitfalls-to-Avoid.md), [\[mctools-my...epoint.com\]](https://mctools-my.sharepoint.com/personal/mckerracher_joshua_mayo_edu/Documents/Microsoft%20Copilot%20Chat%20Files/Common-Pitfalls-to-Avoid.md)
*   Do not delete or rewrite unrelated content in project-progress.md. [\[mctools-my...epoint.com\]](https://mctools-my.sharepoint.com/personal/mckerracher_joshua_mayo_edu/Documents/Microsoft%20Copilot%20Chat%20Files/Directory-And-File-Naming-Standards.md)
*   Do not add secrets/PII. [\[mctools-my...epoint.com\]](https://mctools-my.sharepoint.com/personal/mckerracher_joshua_mayo_edu/Documents/Microsoft%20Copilot%20Chat%20Files/Code-Standards.md), [\[mctools-my...epoint.com\]](https://mctools-my.sharepoint.com/personal/mckerracher_joshua_mayo_edu/Documents/Microsoft%20Copilot%20Chat%20Files/Code-Standards.md)
*   Preserve W\* functionality unless the chosen option explicitly migrates it. [\[mctools-my...epoint.com\]](https://mctools-my.sharepoint.com/personal/mckerracher_joshua_mayo_edu/Documents/Microsoft%20Copilot%20Chat%20Files/Common-Pitfalls-to-Avoid.md)

```