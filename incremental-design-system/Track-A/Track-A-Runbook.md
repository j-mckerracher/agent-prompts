# ✅ Track A Runbook

**Incremental Progress Tracking (CHG Workstreams, Option B)**

This runbook explains **how to run Track A end‑to‑end** for an incremental change that introduces **CHG‑based progress tracking**, from story intake through execution kickoff.

---

## 🧭 What Track A Is (and Is Not)

### Track A *does*

*   Standardize progress tracking for **incremental changes**
*   Use **Change IDs (`CHG‑YYYYMMDD‑<slug>`) as workstream IDs**
*   Introduce **parallel CHG progress scripts** (Option B)
*   Preserve all existing **W\*** workflows untouched
*   Produce **long‑lived, auditable artifacts**

### Track A *does not*

*   Modify existing W\* scripts
*   Change CI/CD pipelines
*   Implement application features
*   Replace greenfield workflows

---

## 🧱 Preconditions (Before You Start)

✅ You have chosen **Option B** (parallel CHG scripts)  
✅ You want **story‑level traceability**  
✅ You want to leave **existing W\* flows untouched**

If all three are true → proceed.

---

## 🪜 End‑to‑End Checklist (IDS‑A‑00 → IDS‑A‑09)

### **Step 0 — Create the Change Folder**

Create a new Change ID and folder:

    ../Planning/Changes/CHG-YYYYMMDD-<slug>/

Inside it, create:

    00-change-intent.md

This file must:

*   Declare `is_hotfix: true|false`
*   Include acceptance criteria
*   Declare scope and constraints

📌 *This is the human entry point.*

---

### **Step 1 — Triage the Change (IDS‑A‑00)**

Run **IDS‑A‑00: Change Intake & Triage**.

This step:

*   Validates the change intent
*   Confirms **one story = one workstream**
*   Classifies risk
*   Decides if the **hotfix path** applies
*   Creates the **per‑change log root**

Artifacts produced:

*   `../Planning/Changes/<CHG>/01-triage.md`
*   `{log_root}/{CHG}/meta/triage.log.md`
*   `{log_root}/{CHG}/meta/01-triage.md` (copy)

✅ **Checkpoint:**  
You now have a bounded change and a log root.

---

### **Step 2 — Analyze Impact (IDS‑A‑01)**

Run **IDS‑A‑01: Impact Analysis**.

This step:

*   Replaces Macro + Meso planning
*   Defines blast radius, risks, regressions
*   Proposes (but does NOT decide) breaking changes
*   Defines rollback + verification needs

Artifacts produced:

*   `../Planning/Changes/<CHG>/02-impact-analysis.md`
*   `{log_root}/{CHG}/meta/impact-analysis.log.md`
*   `{log_root}/{CHG}/meta/02-impact-analysis.md` (copy)

⚠️ **If compatibility is marked BLOCKED**, stop until the human confirms.

✅ **Checkpoint:**  
You understand what will change, what could break, and how to test it.

---

### **Step 3 — Create the Incremental Micro Plan (IDS‑A‑02)**

Run **IDS‑A‑02: Incremental Micro Plan**.

This step:

*   Turns impact analysis into an execution‑ready plan
*   Proposes **UoW‑sized boundaries**
*   Defines test strategy and regression checks
*   Remains tightly scoped (no architecture rework)

Artifacts produced:

*   `../Planning/Changes/<CHG>/03-incremental-micro-plan.md`
*   `{log_root}/{CHG}/meta/incremental-micro-plan.log.md`
*   `{log_root}/{CHG}/meta/03-incremental-micro-plan.md` (copy)

✅ **Checkpoint:**  
You now have something that *can be decomposed and executed*.

---

### **Step 4 — Initialize Progress Tracking (IDS‑A‑04)**

Run **IDS‑A‑04: Progress Tracking Initializer**.

This step:

*   Creates the per‑change progress file
*   Wires it to planning artifacts
*   Prepares the file for orchestration updates

Artifacts produced:

*   `../progress-tracking/<CHG>-progress.md`
*   `{log_root}/{CHG}/meta/progress-tracking-init.log.md`
*   `{log_root}/{CHG}/meta/progress-file.md` (copy)
*   Optional update to:
    *   `../progress-tracking/project-progress.md`

✅ **Checkpoint:**  
There is now a **single source of truth** for progress on this change.

---

### **Step 5 — Write Progress Script Compatibility Spec (IDS‑A‑05)**

Run **IDS‑A‑05: Progress Script Compatibility Spec**.

This step:

*   Documents the mismatch between W\* and CHG workflows
*   Specifies **Option B** (parallel CHG scripts)
*   Defines acceptance criteria for tooling behavior
*   Produces a human‑reviewable spec

Artifact produced:

*   `../progress-tracking/CHG-progress-script-compatibility-spec.md`

✅ **Checkpoint:**  
You have a clear, reviewed contract for how CHG tooling must behave.

---

### **Step 6 — Plan Tooling Implementation (IDS‑A‑06)**

Run **IDS‑A‑06: CHG Progress Scripts Implementation Plan**.

This step:

*   Converts the compatibility spec into an executable plan
*   Defines script interfaces and behaviors
*   Splits work into UoW‑friendly tasks

Artifact produced:

*   `../progress-tracking/CHG-progress-scripts-implementation-plan.md`

✅ **Checkpoint:**  
You now have a tooling story ready to be decomposed.

---

### **Step 7 — Decompose Tooling Plan into UoWs (IDS‑A‑08)**

Run **IDS‑A‑08: Work Decomposer for CHG Progress Scripts**.

This step:

*   Splits the tooling plan into atomic UoWs
*   Keeps each unit ≤5 files / ≤400 LOC
*   Preserves Option B constraints (no W\* changes)

Artifacts produced:

*   `../Planning/Changes/<CHG>/04-work-decomposer-output.md`
*   `{log_root}/{CHG}/meta/04-work-decomposer-output.md`
*   `{log_root}/{CHG}/meta/decomposer.log.md`

✅ **Checkpoint:**  
You now have **assignable, executable Units of Work**.

---

### **Step 8 — Start Execution (IDS‑A‑09)**

Run **IDS‑A‑09: CHG Progress Scripts Execution Orchestrator Starter**.

This step:

*   Launches the Workstream Orchestrator for the CHG ID
*   Ensures **CHG progress scripts** are used
*   Enforces log‑rooted artifacts
*   Starts UOWOs according to dependencies

Artifacts produced:

*   `{log_root}/{CHG}/orchestration/workstream.log.md`
*   `{log_root}/{CHG}/orchestration/uowo-*.log.md`

✅ **Final Checkpoint:**  
Track A execution is live and progressing via CHG workflows.

---

## ✅ What "Track A Complete" Looks Like

Track A is complete when:

*   All UoWs show `done` in:
        ../progress-tracking/<CHG>-progress.md
*   `project-progress.md` reflects final status
*   Logs exist under:
        {ORCHESTRATED_AGENT_WORK_ROOT}/{CHG}/
*   No W\* scripts were modified
*   CHG scripts are ready for reuse by future incremental stories

---

## 🧠 How to Use This Runbook

*   ✅ Give this page to engineers as **"How to run incremental changes"**
*   ✅ Use it when onboarding reviewers or QA
*   ✅ Keep it next to the incremental prompts
*   ✅ Treat deviations as escalation events

---

## 📁 Track A Prompt Index

| ID | Name | Purpose |
|----|------|---------|
| IDS‑A‑00 | Change Intake & Triage | Validate and scope the change |
| IDS‑A‑01 | Impact Analysis | Blast radius, compatibility, regression checklist |
| IDS‑A‑02 | Incremental Micro Plan | Execution-ready plan with UoW boundaries |
| IDS‑A‑03 | Hotfix Postmortem | Document hotfix incidents (if is_hotfix: true) |
| IDS‑A‑04 | Progress Tracking Initializer | Create per-change progress file |
| IDS‑A‑05 | Progress Script Compatibility Spec | Define Option B script requirements |
| IDS‑A‑06 | CHG Progress Scripts Implementation Plan | Plan for creating CHG scripts |
| IDS‑A‑07 | Incremental Orchestration Progress Commands Addendum | Wire prompts to use CHG scripts |
| IDS‑A‑08 | Work Decomposer for CHG Progress Scripts | Decompose tooling plan into UoWs |
| IDS‑A‑09 | CHG Progress Scripts Execution Orchestrator Starter | Kickoff execution |

---

## 🔗 Related

*   **Track B** — Review & QA addenda for incremental changes
*   **Greenfield** — Multi-workstream project planning and execution
