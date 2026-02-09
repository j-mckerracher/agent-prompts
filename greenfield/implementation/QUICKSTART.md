# Quickstart Guide

Get up and running with the Sequential Task Decomposition workflow in under 5 minutes.

## 1. One-Time Setup

1.  **Configure Environment**:
    ```bash
    cp workflow-config.example.yaml workflow-config.yaml
    ```
2.  **Edit Configuration**:
    Open `workflow-config.yaml` and update the paths to match your local machine:
    *   `knowledge_root`: Path to `agent-reference/knowledge/` (Absolute path)
    *   `artifact_root`: Where you want output files stored (Absolute path)
    *   `obsidian_vault_root`: Your Obsidian vault location (Absolute path)
    *   `project_type`: Use `"greenfield"` for new projects without an existing codebase
    *   `planning_docs_root` or `planning_docs_paths`: Point to PRD/plan docs if greenfield

## 2. Running the Workflow

1.  **Start the Orchestrator**:
    Copy the contents of `01-orchestrator-agent-prompt.md` and paste it into your AI agent session.

2.  **Provide Story Details**:
    The agent will provide a YAML template. Fill it out with your `change_id`, `code_repo`, and `story` details.

3.  **Execute Stages**:
    Follow the agent's guidance. The workflow proceeds sequentially:
    *   **Planning**: Task Gen → Assignment (UoWs sourced from planning docs)
    *   **Execution**: Implementation
    *   **Validation**: QA → UI QA (if applicable)

## 3. Monitoring & Logs

*   **Human Log**: Check `{{artifact_root}}/{CHANGE_ID}/work-log.md` for a readable narrative of all actions.
*   **Machine Artifacts**: Check `{{artifact_root}}/{CHANGE_ID}/` for YAML reports and state files.

## Tips

*   **Prompting**: Always allow the agents to read the `workflow-config.yaml` when they start.
*   **Logging**: If you need to stop/resume, tell the agent to "Resume from state X" (see README).

---
*For full details, see [README.md](./README.md).*
