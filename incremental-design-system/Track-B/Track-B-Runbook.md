# ✅ Track B Runbook

**Incremental Review & QA Addenda (Code Review + QA Enforcement)**

This runbook explains **how to run Track B end‑to‑end** for an incremental change that focuses on **review and QA quality gates**, from story intake through verified completion.

---

## 🧭 What Track B Is (and Is Not)

### Track B *does*

*   Standardize **code review** for incremental changes with scope-drift detection
*   Standardize **QA verification** with regression checklist enforcement
*   Use **Change IDs (`CHG‑YYYYMMDD‑<slug>`) as workstream IDs**
*   Produce **long‑lived, auditable review and QA artifacts**
*   Work with existing orchestration (W‑O/UOWO) without requiring new progress scripts
*   Provide a **standalone track** that can run independently of Track A

### Track B *does not*

*   Introduce new progress tracking scripts (that's Track A)
*   Modify existing W\* scripts or CI/CD pipelines
*   Implement application features
*   Replace greenfield workflows

---

## 🧱 Preconditions (Before You Start)

✅ You have an incremental change (user story, bug, refactor)  
✅ You want **rigorous code review and QA gates**  
✅ You want **story‑level traceability** for review/QA artifacts  
✅ You can use existing W\* progress tracking OR manual updates

If all are true → proceed.

---

## 🪜 End‑to‑End Checklist (IDS‑B‑00 → IDS‑B‑07)

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

### **Step 1 — Triage the Change (IDS‑B‑00)**

Run **IDS‑B‑00: Change Intake & Triage**.

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

### **Step 2 — Analyze Impact (IDS‑B‑01)**

Run **IDS‑B‑01: Impact Analysis**.

This step:

*   Replaces Macro + Meso planning
*   Defines blast radius, risks, regressions
*   Proposes (but does NOT decide) breaking changes
*   Defines rollback + verification needs
*   **Produces the regression checklist** (critical for Track B QA)

Artifacts produced:

*   `../Planning/Changes/<CHG>/02-impact-analysis.md`
*   `{log_root}/{CHG}/meta/impact-analysis.log.md`
*   `{log_root}/{CHG}/meta/02-impact-analysis.md` (copy)

⚠️ **If compatibility is marked BLOCKED**, stop until the human confirms.

✅ **Checkpoint:**  
You have a regression checklist and compatibility classification.

---

### **Step 3 — Create the Incremental Micro Plan (IDS‑B‑02)**

Run **IDS‑B‑02: Incremental Micro Plan**.

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

### **Step 4 — Decompose into Units of Work (IDS‑B‑03)**

Run **IDS‑B‑03: Incremental Work Decomposer**.

This step:

*   Splits the micro plan into atomic UoWs
*   Keeps each unit ≤5 files / ≤400 LOC / ≤10 steps
*   Archives decomposition per-change

Artifacts produced:

*   `../Planning/Changes/<CHG>/04-work-decomposer-output.md`
*   `{log_root}/{CHG}/meta/04-work-decomposer-output.md` (copy)
*   `{log_root}/{CHG}/meta/decomposer.log.md`

✅ **Checkpoint:**  
You now have **assignable, executable Units of Work**.

---

### **Step 5 — Code Review with Addendum (IDS‑B‑04)**

Run **IDS‑B‑04: Incremental Code Review Addendum** for each UoW.

This step:

*   Enforces scope-drift detection against micro plan
*   Verifies compatibility gate adherence
*   Checks regression checklist coverage in tests
*   Validates logging/observability constraints
*   Produces a structured review report

Artifacts produced:

*   `{log_root}/{CHG}/reviews/Review-<Uxx>.md`
*   `../Planning/Changes/<CHG>/artifacts/reviews/Review-<Uxx>.md` (repo copy)

✅ **Checkpoint:**  
Code is reviewed with incremental-specific checks.

---

### **Step 6 — QA Verification with Addendum (IDS‑B‑05)**

Run **IDS‑B‑05: Incremental QA Addendum** for each UoW (or at change level).

This step:

*   Enforces regression checklist execution (from Impact Analysis)
*   Collects evidence for each checklist item
*   Validates acceptance criteria coverage
*   Checks determinism/flake expectations
*   Produces a structured QA report

Artifacts produced:

*   `{log_root}/{CHG}/qa/QA-Report-<Uxx>.md`
*   `../Planning/Changes/<CHG>/artifacts/qa/QA-Report-<Uxx>.md` (repo copy)

✅ **Checkpoint:**  
QA is complete with regression checklist evidence.

---

### **Step 7 — Hotfix Postmortem (IDS‑B‑06) — ONLY if is_hotfix: true**

Run **IDS‑B‑06: Hotfix Postmortem** after production deployment.

This step:

*   Documents what happened, what was deployed, and what broke
*   Captures timeline and root cause
*   Defines follow-up actions
*   Archives for future reference

Artifacts produced:

*   `../Planning/Changes/<CHG>/99-postmortem.md`
*   `{log_root}/{CHG}/postmortem/postmortem.md` (copy)

✅ **Checkpoint:**  
Hotfix is documented for future learning.

---

## ✅ What "Track B Complete" Looks Like

Track B is complete when:

*   All UoWs have **Review-\*.md** and **QA-Report-\*.md** artifacts
*   Regression checklist items have evidence
*   Compatibility gate was respected (or human-confirmed if breaking)
*   If hotfix: postmortem exists
*   Logs exist under:
        {ORCHESTRATED_AGENT_WORK_ROOT}/{CHG}/
*   Repo copies exist under:
        ../Planning/Changes/{CHG}/artifacts/

---

## 🧠 How to Use This Runbook

*   ✅ Give this page to engineers as **"How to review/QA incremental changes"**
*   ✅ Use it when onboarding reviewers or QA
*   ✅ Keep it next to the Track B prompts
*   ✅ Treat deviations as escalation events

---

## 📁 Track B Prompt Index

| ID | Name | Purpose |
|----|------|---------|
| IDS‑B‑00 | Change Intake & Triage | Validate and scope the change |
| IDS‑B‑01 | Impact Analysis | Blast radius, compatibility, regression checklist |
| IDS‑B‑02 | Incremental Micro Plan | Execution-ready plan with UoW boundaries |
| IDS‑B‑03 | Incremental Work Decomposer | Split into atomic UoWs |
| IDS‑B‑04 | Incremental Code Review Addendum | Scope-drift and regression coverage |
| IDS‑B‑05 | Incremental QA Addendum | Regression checklist enforcement |
| IDS‑B‑06 | Hotfix Postmortem | Document hotfix incidents |

---

## 🔗 Related

*   **Track A** — Progress tracking standardization with CHG scripts (Option B)
*   **Greenfield** — Multi-workstream project planning and execution
