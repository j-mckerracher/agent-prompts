# Changelog

All notable changes to the Sequential Task Decomposition with Evaluator-Optimizer Loop workflow are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [Unreleased]

### Removed
- **Test Writing Agents** — Deleted `06-test-writer-agent-prompt.md` (Unit/Component Test Writer), `06b-integration-test-writer-agent-prompt.md` (Integration Test Writer), and `12-test-evaluator-prompt.md` (Test Evaluator). This workflow variant does not include automated test writing.
- **`test_stack` configuration** — Removed from `workflow-config.yaml`, `workflow-config.example.yaml`, orchestrator YAML template, and all agent prompts.
- **`e2e_tests_root` configuration** — Removed from all config files and agent prompt headers.
- **Jest/Cypress gates** — Removed Jest Gate from Implementation Evaluator and Jest/Cypress Full Suite gates from QA Evaluator.
- **Test-related workflow stages** — Removed stages 5b (Unit/Component Testing) and 5c (Integration Testing) from the orchestrator and README.
- **`test_gap` issue type** — Removed from QA Agent, QA Evaluator, and post-QA feedback routing.
- **Test writer log directories** — Removed `logs/unit_test_writer/` and `logs/integration_test_writer/` from directory structure.
- **Test report artifacts** — Removed `unit_test_report.yaml` and `integration_test_report.yaml` from execution output.

### Changed
- Updated all agent prompts to remove test-related references while preserving core functionality.
- Updated README.md: Agent Hierarchy diagram, Agent Invocation table, Workflow Stages, Directory Structure, Agent Prompts Reference table, Model Configuration, Example Workflow Run, and Troubleshooting.
- Updated QUICKSTART.md: Removed test_stack/e2e_tests_root from setup, simplified execution stages.
- Playwright UI QA (`07b-ui-qa-agent-prompt.md`, `14-ui-qa-evaluator-prompt.md`) remains unchanged — it performs visual validation, not test suite execution.

## [2026-02-02]

### Added
- **Quickstart Guide** (`QUICKSTART.md`) - Concise instructions for team onboarding.
- **Configuration System** - `workflow-config.yaml` allows per-user path configuration, replacing hardcoded paths.
- **Human-Readable Logging** - Enforced `work-log.md` creation in `rules-and-guidance.md` to comply with logging policies.

### Changed
- **Token Optimization** - Migrated ALL agent artifacts and communication from JSON to YAML (~30% token reduction).
- **Prompt Architecture** - Updated all prompts to dynamically read paths from `workflow-config.yaml`.
- **Documentation** - Updated `README.md` with setup instructions and YAML examples.

## [2026-01-28]

### Added
- **UI QA Agent** (`07b-ui-qa-agent-prompt.md`) - New conditional agent for validating UI consistency
  - Uses Playwright CLI to compare new/modified UI elements against existing baseline elements
  - Runs `playwright-cli --help` first to discover available commands
  - Detects UI changes via file patterns (`.html`, `.css`, `.scss`, `.tsx`, `.component.ts`, etc.)
  - Produces `ui_qa_report.yaml` with consistency validation, screenshots, and discrepancy findings
  - Only invoked when UI changes are detected in the implementation

- **UI QA Evaluator** (`14-ui-qa-evaluator-prompt.md`) - Evaluator for UI QA reports
  - Validates baseline selection quality
  - Assesses comparison thoroughness
  - Verifies evidence quality (screenshots, Playwright traces)
  - Validates skip justification when UI QA is skipped

- **UI Change Detection Logic** in Orchestrator
  - Analyzes implementation reports for UI file modifications
  - File patterns: `.html`, `.css`, `.scss`, `.tsx`, `.vue`, `.component.ts`, `.component.html`
  - Directory patterns: `components/`, `ui/`, `styles/`, `views/`, `templates/`
  - Logs detection decision in orchestrator logs

- New workflow state: `UIQA` → `EvalUIQA` (conditional, after QA passes)
- New issue type `ui_consistency` for remediation routing
- New directories: `logs/ui_qa/`, `qa/evidence/ui/`
- New artifact: `qa/ui_qa_report.yaml`

### Changed
- Updated workflow state machine to include conditional UI QA stage
- Updated Agent Invocation Summary table to include UI QA Agent
- Updated Evaluators diagram to include UI QA Evaluator
- Updated directory structure documentation
- Updated agent logging table
- Updated remediation routing table with `ui_consistency` issue type
- Updated model configuration to include `ui_qa` agent
- Updated iteration limits to include `ui_qa: 2`

---

## [2026-01-27]

### Added
- Initial workflow implementation for story 4729040 (Person ID hyperlinks in tooltips)
- Knowledge management system with `learnings.yaml` and `questions.yaml`
- Learnings documented:
  - L001: Tooltip implementation using PrimeNG v19's pTooltip directive
  - L002: Cancelled by tooltip generation in test-pill.component.ts
  - L003: Receipted tooltip generation in tests.middleware.ts
  - L004: Notes display with actor (person ID) in note.component.html
  - L005: Data model for person IDs (canceledBy, receiptedBy, actor)
  - L006: Quarterly person lookup URL pattern

---

## Template for Future Entries

```markdown
## [YYYY-MM-DD]

### Added
- New features or capabilities

### Changed
- Changes to existing functionality

### Fixed
- Bug fixes

### Removed
- Removed features or files

### Deprecated
- Features marked for future removal
```
