---
name: worker-reporting
description: >-
  Standard reporting and durability protocol for task workers. Covers durable
  result persistence (git push, PR creation, shared-dir artifacts), the mandatory
  5-part report structure, and completion signaling via sciontool.
---

# Task Worker Reporting & Durability Protocol

You are an ephemeral task worker. Once your result is accepted by the coordinator, your container and local workspace are destroyed. Anything that exists only inside your container filesystem will be permanently lost.

Follow this protocol to make your results durable, format your report, and cleanly signal completion.

## 1. Make Your Output Durable (Before Reporting)

Before sending your report or completing your task, you MUST persist your work outside the container:

### Code Changes
- **Commit:** Create clean, focused git commits on your assigned branch.
  - Non-interactive commit: `git commit -m "feat(scope): concise description" --no-edit`
- **Push:** Push your branch to the remote repository:
  - `git push origin <branch-name>`
- **Pull Request:** When the project workflow or task requires a PR, create it using GitHub CLI:
  - `gh pr create --title "..." --body "..."`
- **Push Blocked?** If git push fails (e.g. read-only credentials, air-gapped network, no remote permission), **state this explicitly in your report**. The coordinator will keep your container alive until a human or coordinator pulls or exports your changes.

### Analyses, Findings, and Artifacts
- **Short Findings:** Include the complete finding or answer directly in your final report.
- **Large Files, Logs, Datasets, or Binaries:** Place them in persistent shared storage:
  - Shared directories: `/scion-volumes/<name>/` (inspect available mounts with `scion shared-dir list`)
  - Or publish via artifact service if enabled: `scion artifact publish <path>`

---

## 2. Mandatory 5-Part Report Format

When your task is complete (or if you are permanently blocked), send exactly ONE structured report to the coordinator. Use these five exact headings:

### - **Outcome:**
A single line stating `done`, `partly done`, or `blocked`.

### - **Result:**
What you produced and where it permanently lives:
- Git branch name and commit SHA (e.g., `branch: scion/fix-auth-1, commit: a1b2c3d`)
- Pull Request URL (if opened)
- File paths on shared storage (e.g., `/scion-volumes/artifacts/summary.pdf`)
- Or the answer/analysis itself for analytical tasks.

### - **Verification:**
What verification steps you executed and what passed:
- Exact test, linter, or build commands executed (e.g., `go test -v ./pkg/auth/... -run TestTokenRefresh`)
- Result and exit code (e.g., `PASS: 12 tests passed, 0 failures`)
- If no automated tests exist, state the manual or static verification performed.

### - **Assumptions and Open Points:**
Any key assumptions made during execution, trade-offs, or open decisions requiring coordinator/user attention. If none, write "None".

### - **Out of Scope:**
Adjacent issues, bugs, or refactoring opportunities noticed during the work that were outside the task's scope. If none, write "None".

---

## 3. Communication & Clarifications

- **Ambiguity Rule:** If an essential requirement is ambiguous and a wrong guess would waste significant work, ask the coordinator ONE concise question with your proposed default, accompanied by:
  ```bash
  sciontool status ask_user "<your question>"
  ```
  Then message the coordinator:
  ```bash
  scion message coordinator "<question>"
  ```
  Otherwise, make a reasonable assumption, document it under **Assumptions**, and proceed.
- **Reporting to Coordinator:** Send your final report to the coordinator:
  ```bash
  scion message coordinator "<structured report>"
  ```

---

## 4. Signal Task Completion

Once your changes are durable and your report has been sent, notify the Scion orchestration engine:

```bash
sciontool status task_completed "<task title>"
```

**CRITICAL:** Stop immediately after signaling completion. Do NOT enter a polling loop, do NOT run `sleep`, and do NOT append questions like "What should I do next?".
