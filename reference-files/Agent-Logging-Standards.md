# Agent Logging Standards

## Overview

Every agent in the orchestrated workflow system MUST produce a log file each time it is spawned. This provides:
- **Traceability**: Full audit trail of agent actions
- **Debugging**: Ability to diagnose failures and understand agent behavior
- **Metrics**: Data for improving agent efficiency and identifying bottlenecks
- **Reproducibility**: Ability to understand what happened during any execution

## Log Directory Structure

All agent logs are stored under the orchestrated work root, organized by change/workstream:

```
{LOG_ROOT}/{CHANGE_ID}/
├── meta/                          # Planning & orchestration logs
│   ├── triage.log.md
│   ├── impact-analysis.log.md
│   ├── incremental-micro-plan.log.md
│   ├── decomposer.log.md
│   ├── work-assigner.log.md
│   └── orchestrator.log.md
├── planning/                      # Planning artifacts (copies)
│   ├── 01-triage.md
│   ├── 02-impact-analysis.md
│   ├── 03-incremental-micro-plan.md
│   └── 04-work-decomposer-output.md
├── assignments/                   # Work assignments
│   ├── UoW-U01-Assignment.md
│   └── UoW-U02-Assignment.md
├── se/                           # Software Engineer logs & artifacts
│   ├── logs/
│   │   ├── SE-Log-U01.md
│   │   └── SE-Log-U02.md
│   ├── diffs/
│   │   ├── U01.diff
│   │   └── U02.diff
│   ├── test-results-U01.txt
│   ├── test-results-U02.txt
│   ├── build-results-U01.txt
│   └── build-results-U02.txt
├── reviews/                       # Code review reports
│   ├── Review-U01.md
│   ├── Review-U02.md
│   └── reviewer.log.md
├── qa/                           # QA reports
│   ├── QA-Report-U01.md
│   ├── QA-Report-U02.md
│   ├── test-plan-U01.md
│   └── qa.log.md
├── deployments/                   # Deployment reports
│   ├── Deploy-U01.md
│   └── devops.log.md
├── escalations/                   # Escalation reports
│   └── Escalation-U05-20260127.md
├── postmortem/                    # Hotfix postmortems
│   └── hotfix-postmortem.md
├── integration-tests/             # Integration test outputs
│   └── integration-results.md
├── orchestration/                 # Orchestrator state & decisions
│   ├── uowo-U01.log.md
│   ├── uowo-U02.log.md
│   └── workstream-orchestrator.log.md
└── progress/                      # Progress tracking
    ├── project-progress.md
    └── workstream-progress.md
```

### Log Root Resolution

1. Read `log_root` from triage frontmatter if present
2. Else use environment variable `ORCHESTRATED_AGENT_WORK_ROOT` if set
3. Else fallback to: `/Users/mckerracher.joshua/Documents/sbx-rls-iac-josh/Work/Orchestrated-agent-work`
4. Append `/{CHANGE_ID}/` to create the full path

---

## Agent-Specific Log Templates

### 1. Triage Agent Log (IDS-B-00)

**Path:** `{log_root}/meta/triage.log.md`

```markdown
---
tags: [agent-log, triage, ids-b-00]
agent: "Triage Agent"
change_id: "{CHANGE_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "complete|blocked|escalated"
---

# Triage Agent Log — {CHANGE_ID}

## Invocation Context
- **Spawned by:** Human | Orchestrator
- **Input source:** User request | Ticket | PR
- **Session ID:** {SESSION_ID}

## Inputs Received
- Change description: {summary}
- Requested urgency: {urgency}
- Source reference: {ticket_url_or_pr}

## Analysis Performed
- Complexity assessment: {trivial|small|medium|large|xlarge}
- Risk level: {low|medium|high|critical}
- Track assigned: {A|B}
- Compatibility status: {confirmed|pending|blocked}

## Decisions Made
1. {decision_1}
2. {decision_2}

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| Triage document | `{log_root}/planning/01-triage.md` | Written |
| Triage log | `{log_root}/meta/triage.log.md` | Written |

## Notes / Blockers
- {any_notes_or_blockers}

## Handoff
- **Next agent:** Impact Analysis Agent
- **Handoff status:** Ready | Blocked
```

---

### 2. Impact Analysis Agent Log (IDS-B-01)

**Path:** `{log_root}/meta/impact-analysis.log.md`

```markdown
---
tags: [agent-log, impact-analysis, ids-b-01]
agent: "Impact Analysis Agent"
change_id: "{CHANGE_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "complete|blocked|escalated"
---

# Impact Analysis Agent Log — {CHANGE_ID}

## Invocation Context
- **Spawned by:** Orchestrator | Human
- **Triage path:** `{log_root}/planning/01-triage.md`
- **Session ID:** {SESSION_ID}

## Inputs Analyzed
- Triage document: ✓ Read
- Codebase files scanned: {count}
- Dependencies checked: {count}

## Analysis Performed
- Files potentially affected: {count}
- Components impacted: {list}
- Regression risk areas: {list}
- Breaking change potential: {yes|no}

## Regression Checklist Created
| ID | Area | Risk | Verification |
|----|------|------|--------------|
| RC-01 | {area} | {risk} | {verification} |
| RC-02 | {area} | {risk} | {verification} |

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| Impact analysis | `{log_root}/planning/02-impact-analysis.md` | Written |
| Impact log | `{log_root}/meta/impact-analysis.log.md` | Written |

## Notes / Blockers
- {any_notes_or_blockers}

## Handoff
- **Next agent:** Incremental Micro Plan Agent
- **Handoff status:** Ready | Blocked
```

---

### 3. Incremental Micro Plan Agent Log (IDS-B-02)

**Path:** `{log_root}/meta/incremental-micro-plan.log.md`

```markdown
---
tags: [agent-log, micro-plan, ids-b-02]
agent: "Incremental Micro Plan Agent"
change_id: "{CHANGE_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "complete|blocked|escalated"
---

# Micro Plan Agent Log — {CHANGE_ID}

## Invocation Context
- **Spawned by:** Orchestrator | Human
- **Impact analysis path:** `{log_root}/planning/02-impact-analysis.md`
- **Session ID:** {SESSION_ID}

## Inputs Used
- Impact analysis: ✓ Read
- Regression checklist items: {count}
- Code standards: ✓ Applied

## Planning Performed
- UoW boundaries proposed: {count}
- Estimated total LOC: {estimate}
- Critical path identified: {yes|no}

## UoW Boundaries Proposed
| UoW | Title | Est. Files | Est. LOC |
|-----|-------|------------|----------|
| U01 | {title} | {files} | {loc} |
| U02 | {title} | {files} | {loc} |

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| Micro plan | `{log_root}/planning/03-incremental-micro-plan.md` | Written |
| Micro plan log | `{log_root}/meta/incremental-micro-plan.log.md` | Written |

## Notes / Blockers
- {any_notes_or_blockers}

## Handoff
- **Next agent:** Work Decomposer Agent
- **Handoff status:** Ready | Blocked
```

---

### 4. Work Decomposer Agent Log (IDS-B-03)

**Path:** `{log_root}/meta/decomposer.log.md`

```markdown
---
tags: [agent-log, decomposer, ids-b-03]
agent: "Work Decomposer Agent"
change_id: "{CHANGE_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "complete|blocked|escalated"
inputs:
  micro_plan: "{path}"
  impact_analysis: "{path}"
  triage: "{path}"
outputs:
  decomposition: "{path}"
  log_copy: "{path}"
---

# Work Decomposer Agent Log — {CHANGE_ID}

## Invocation Context
- **Spawned by:** Orchestrator | Human
- **Micro plan path:** `{log_root}/planning/03-incremental-micro-plan.md`
- **Session ID:** {SESSION_ID}

## Decomposition Summary
- **UoWs created:** {count}
- **Dependency shape:** DAG (verified)
- **Estimated total LOC:** {estimate}
- **Critical path:** {U01 → U03 → ...}

## UoWs Created
| ID | Title | Dependencies | Est. Files | Est. LOC | Est. Steps |
|----|-------|--------------|------------|----------|------------|
| U01 | {title} | None | {files} | {loc} | {steps} |
| U02 | {title} | U01 | {files} | {loc} | {steps} |

## Constraint Verification
- [ ] All UoWs ≤ 5 files: ✓
- [ ] All UoWs ≤ 400 LOC: ✓
- [ ] All UoWs ≤ 10 steps: ✓
- [ ] No circular dependencies: ✓

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| Decomposition | `{log_root}/planning/04-work-decomposer-output.md` | Written |
| Decomposer log | `{log_root}/meta/decomposer.log.md` | Written |

## Notes / Blockers
- {any_notes_or_blockers}

## Handoff
- **Next agent:** Work Assigner Agent (per UoW)
- **Handoff status:** Ready | Blocked
```

---

### 5. Work Assigner Agent Log

**Path:** `{log_root}/meta/work-assigner.log.md`

```markdown
---
tags: [agent-log, work-assigner, agent-02]
agent: "Work Assigner Agent"
change_id: "{CHANGE_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "complete|blocked|escalated"
---

# Work Assigner Agent Log — {CHANGE_ID}

## Invocation Context
- **Spawned by:** Orchestrator
- **Decomposition path:** `{log_root}/planning/04-work-decomposer-output.md`
- **Session ID:** {SESSION_ID}

## Assignments Created
| UoW ID | Title | Assignment Path | Status |
|--------|-------|-----------------|--------|
| U01 | {title} | `{log_root}/assignments/UoW-U01-Assignment.md` | Created |
| U02 | {title} | `{log_root}/assignments/UoW-U02-Assignment.md` | Created |

## Selection Rationale
- Priority consideration: {notes}
- Dependency satisfaction: {notes}
- Critical path: {notes}

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| Assignment U01 | `{log_root}/assignments/UoW-U01-Assignment.md` | Written |
| Assignment U02 | `{log_root}/assignments/UoW-U02-Assignment.md` | Written |
| Assigner log | `{log_root}/meta/work-assigner.log.md` | Written |

## Notes / Blockers
- {any_notes_or_blockers}

## Handoff
- **Next agent:** Software Engineer Agent (per UoW)
- **Handoff status:** Ready | Blocked
```

---

### 6. Software Engineer Agent Log

**Path:** `{log_root}/se/logs/SE-Log-{UNIT_ID}.md`

```markdown
---
tags: [agent-log, software-engineer, agent-03]
agent: "Software Engineer Agent"
change_id: "{CHANGE_ID}"
unit_id: "{UNIT_ID}"
workstream_id: "{WORKSTREAM_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "ready_for_review|blocked|escalated"
is_rework: false|true
rework_cycle: 0|1|2|3
---

# SE Agent Log — {UNIT_ID}

## Invocation Context
- **Spawned by:** Orchestrator | UOWO
- **Assignment path:** `{log_root}/assignments/UoW-{UNIT_ID}-Assignment.md`
- **Session ID:** {SESSION_ID}
- **Is rework:** {yes|no}
- **Rework cycle:** {0|1|2|3}

## Task Summary
- **Goal:** {goal_from_assignment}
- **Files to read first:** {list}
- **Files to modify:** {list}
- **Success criteria:** {count} items

## Implementation Timeline

### Phase 1: Validation
- **Started:** {timestamp}
- **Assignment complete:** ✓
- **Dependencies satisfied:** ✓
- **Inputs available:** ✓

### Phase 2: Pre-flight
- **Files read:** {list}
- **Patterns identified:** {notes}
- **Plan:** {minimal_change_description}

### Phase 3: Implementation
| Step | Description | Files | LOC | Status |
|------|-------------|-------|-----|--------|
| 1 | {description} | {files} | {loc} | ✓ |
| 2 | {description} | {files} | {loc} | ✓ |

### Phase 4: Validation
- **Lint:** ✓ Pass
- **TypeCheck:** ✓ Pass
- **Tests:** ✓ Pass ({count} tests)
- **Build:** ✓ Pass

## Changes Made
| File | Change Type | LOC Changed | Rationale |
|------|-------------|-------------|-----------|
| {path} | Modified | {loc} | {why} |
| {path} | Created | {loc} | {why} |

## Tests Added/Updated
| Test File | Tests Added | Coverage |
|-----------|-------------|----------|
| {path} | {count} | {coverage} |

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| Diff | `{log_root}/se/diffs/{UNIT_ID}.diff` | Written |
| Test results | `{log_root}/se/test-results-{UNIT_ID}.txt` | Written |
| Build results | `{log_root}/se/build-results-{UNIT_ID}.txt` | Written |
| SE Log | `{log_root}/se/logs/SE-Log-{UNIT_ID}.md` | Written |

## Metrics
- **Total files modified:** {count}
- **Total LOC changed:** {count}
- **Total steps executed:** {count}
- **Constraints satisfied:** ✓ (≤5 files, ≤400 LOC, ≤10 steps)

## Notes / Risks
- {any_notes_or_risks}

## Handoff
- **Next agent:** Code Reviewer Agent
- **Handoff status:** ready_for_review | blocked
```

---

### 7. Code Reviewer Agent Log

**Path:** `{log_root}/reviews/reviewer.log.md`

```markdown
---
tags: [agent-log, code-reviewer, agent-04]
agent: "Code Reviewer Agent"
change_id: "{CHANGE_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "complete"
reviews_performed: [{UNIT_IDS}]
---

# Code Reviewer Agent Log — {CHANGE_ID}

## Session Summary
- **Session ID:** {SESSION_ID}
- **Reviews performed:** {count}
- **Approved:** {count}
- **Rejected:** {count}

## Reviews Performed

### {UNIT_ID} Review
- **Started:** {timestamp}
- **Decision:** Approved | Rejected
- **Duration:** {minutes}
- **Issues found:** {count}
- **Report path:** `{log_root}/reviews/Review-{UNIT_ID}.md`

| Category | Status | Issues |
|----------|--------|--------|
| Correctness | Pass/Fail | {count} |
| Standards | Pass/Fail | {count} |
| Security | Pass/Fail | {count} |
| Tests | Pass/Fail | {count} |
| Maintainability | Pass/Fail | {count} |
| Scope | Pass/Fail | {count} |

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| Review {UNIT_ID} | `{log_root}/reviews/Review-{UNIT_ID}.md` | Written |
| Reviewer log | `{log_root}/reviews/reviewer.log.md` | Written |

## Notes
- {any_notes}
```

---

### 8. QA Engineer Agent Log

**Path:** `{log_root}/qa/qa.log.md`

```markdown
---
tags: [agent-log, qa-engineer, agent-05]
agent: "QA Engineer Agent"
change_id: "{CHANGE_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "complete"
tests_performed: [{UNIT_IDS}]
---

# QA Engineer Agent Log — {CHANGE_ID}

## Session Summary
- **Session ID:** {SESSION_ID}
- **QA cycles performed:** {count}
- **Approved:** {count}
- **Rejected:** {count}

## QA Cycles Performed

### {UNIT_ID} QA
- **Started:** {timestamp}
- **Decision:** Approved | Rejected
- **Duration:** {minutes}
- **Bugs found:** {count}
- **Report path:** `{log_root}/qa/QA-Report-{UNIT_ID}.md`

| Category | Total | Passed | Failed |
|----------|-------|--------|--------|
| Acceptance | {n} | {n} | {n} |
| Edge Cases | {n} | {n} | {n} |
| Integration | {n} | {n} | {n} |
| Regression | {n} | {n} | {n} |

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| QA Report {UNIT_ID} | `{log_root}/qa/QA-Report-{UNIT_ID}.md` | Written |
| Test Plan {UNIT_ID} | `{log_root}/qa/test-plan-{UNIT_ID}.md` | Written |
| QA log | `{log_root}/qa/qa.log.md` | Written |

## Notes
- {any_notes}
```

---

### 9. DevOps Agent Log

**Path:** `{log_root}/deployments/devops.log.md`

```markdown
---
tags: [agent-log, devops, agent-06]
agent: "DevOps Agent"
change_id: "{CHANGE_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "complete"
deployments: [{UNIT_IDS}]
---

# DevOps Agent Log — {CHANGE_ID}

## Session Summary
- **Session ID:** {SESSION_ID}
- **Deployments performed:** {count}
- **Successful:** {count}
- **Failed:** {count}
- **Rolled back:** {count}

## Deployments Performed

### {UNIT_ID} Deployment
- **Started:** {timestamp}
- **Environment:** staging | production
- **Status:** success | failed | rolled_back
- **Version:** {version}
- **Duration:** {minutes}
- **Report path:** `{log_root}/deployments/Deploy-{UNIT_ID}.md`

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| Deploy Report {UNIT_ID} | `{log_root}/deployments/Deploy-{UNIT_ID}.md` | Written |
| DevOps log | `{log_root}/deployments/devops.log.md` | Written |

## Notes
- {any_notes}
```

---

### 10. Orchestrator Agent Log

**Path:** `{log_root}/orchestration/orchestrator.log.md` (for overall orchestration)
**Path:** `{log_root}/orchestration/uowo-{UNIT_ID}.log.md` (per-UoW orchestration)

```markdown
---
tags: [agent-log, orchestrator, agent-00]
agent: "Unit-of-Work Orchestrator Agent"
change_id: "{CHANGE_ID}"
unit_id: "{UNIT_ID}"
spawned_at: "{ISO_TIMESTAMP}"
completed_at: "{ISO_TIMESTAMP}"
status: "success|blocked|failed"
---

# UOWO Agent Log — {UNIT_ID}

## Invocation Context
- **Spawned by:** Workstream Orchestrator
- **Workstream ID:** {WORKSTREAM_ID}
- **Unit ID:** {UNIT_ID}
- **Session ID:** {SESSION_ID}

## State Transitions
| Timestamp | From State | To State | Agent | Notes |
|-----------|------------|----------|-------|-------|
| {ts} | pending | assigned | Work Assigner | Assignment created |
| {ts} | assigned | in_progress | SE | Implementation started |
| {ts} | in_progress | ready_for_review | SE | Implementation complete |
| {ts} | ready_for_review | code_review_approved | Code Reviewer | Approved |
| {ts} | code_review_approved | qa_approved | QA | Approved |
| {ts} | qa_approved | deployed | DevOps | Deployed |
| {ts} | deployed | done | UOWO | Complete |

## Subagent Invocations
| Agent | Spawned | Completed | Duration | Result |
|-------|---------|-----------|----------|--------|
| Work Assigner | {ts} | {ts} | {min} | ready |
| Software Engineer | {ts} | {ts} | {min} | ready_for_review |
| Code Reviewer | {ts} | {ts} | {min} | approved |
| QA Engineer | {ts} | {ts} | {min} | approved |
| DevOps | {ts} | {ts} | {min} | success |

## Rejection Cycles
- **Code Review rejections:** {count}/3
- **QA rejections:** {count}/2

## Final Result
- **Status:** success | blocked | failed
- **Total duration:** {minutes}
- **Artifacts produced:** {count}

## Outputs Produced
| Artifact | Path | Status |
|----------|------|--------|
| UOWO Log | `{log_root}/orchestration/uowo-{UNIT_ID}.log.md` | Written |

## Notes
- {any_notes}
```

---

## Logging Requirements by Agent

| Agent | Log Path | Required | Template Section |
|-------|----------|----------|------------------|
| Triage (IDS-B-00) | `meta/triage.log.md` | ✓ | §1 |
| Impact Analysis (IDS-B-01) | `meta/impact-analysis.log.md` | ✓ | §2 |
| Micro Plan (IDS-B-02) | `meta/incremental-micro-plan.log.md` | ✓ | §3 |
| Work Decomposer (IDS-B-03) | `meta/decomposer.log.md` | ✓ | §4 |
| Work Assigner (02) | `meta/work-assigner.log.md` | ✓ | §5 |
| Software Engineer (03) | `se/logs/SE-Log-{UNIT_ID}.md` | ✓ | §6 |
| Code Reviewer (04) | `reviews/reviewer.log.md` | ✓ | §7 |
| QA Engineer (05) | `qa/qa.log.md` | ✓ | §8 |
| DevOps (06) | `deployments/devops.log.md` | ✓ | §9 |
| UOWO Orchestrator (00) | `orchestration/uowo-{UNIT_ID}.log.md` | ✓ | §10 |
| Workstream Orchestrator | `orchestration/workstream-orchestrator.log.md` | ✓ | — |

---

## Implementation Checklist for Agent Prompts

Each agent prompt MUST include:

1. **Logging section** with:
   - Log path specification
   - Log template reference
   - Required log fields
   
2. **Output requirements** that include:
   - Writing the log file as a mandatory output
   - Log file path in the return contract
   
3. **Log root resolution** logic:
   - Read from triage frontmatter
   - Fallback to environment variable
   - Fallback to default path

---

## Enforcement

- Orchestrator agents MUST verify log files exist before accepting handoffs
- Incomplete logs should trigger escalation
- Log files are NOT optional — they are required artifacts
