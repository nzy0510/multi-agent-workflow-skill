# Controller Runtime

This is the short entrypoint for a controller agent. It exists to decide whether coordinated agent work is appropriate, create a bounded plan packet, and avoid sending excessive context to workers.

If this file conflicts with a completed project profile, prefer the stricter rule that better protects user changes, secrets, data, and release state.

## 1. Startup

Run:

```powershell
git status --short --branch
```

Then confirm:

- Whether the user explicitly requested multiple agents, isolated workspaces, parallel work, or a formal workflow.
- Whether the current tree has existing user or agent changes.
- Whether the task touches high risk areas such as identity, authorization, data migration, deployment configuration, secrets, billing, external dependencies, or production data.
- Whether a project profile exists and defines repository-specific validation commands and protected paths.

## 2. Dispatch Decision

Output this decision before execution:

```text
Task level: L0 / L1 / L2 / L3 / High-risk
Dispatch mode: Single agent / Light coordinated / Standard coordinated / Full coordinated
Use multiple agents:
Enabled roles:
Reason full workflow is not enabled:
Human review gates:
Allowed automatic continuation:
```

Default routing:

| Level | Typical task | Default dispatch |
| --- | --- | --- |
| L0 | Small documentation, tiny fix, local explanation, narrow maintenance | Current thread or one agent |
| L1 | Workflow dry run, larger documentation, low risk tooling change | Controller plus one worker, optional combined review |
| L2 | Two or more independent areas, no high risk surface | Architect, workers, integration, review |
| L3 | End-to-end feature across many areas plus docs or release work | Full coordinated workflow |
| High-risk | Secrets, identity, authorization, data, deployment, billing, external source changes | Add mandatory security, testing, integration, and release review |

Do not start a full workflow just because the user mentioned a workflow. Keep small work small.

## 3. Plan Packet Review

For L2, L3, High-risk, or any requested workflow dry run, create a plan packet before creating worker threads or workspaces.

The plan packet must include:

```text
Goal:
Non-goals:
Acceptance criteria:
Task level and dispatch mode:
Role split:
File ownership matrix:
Branch and workspace mapping:
Validation commands:
High risk operations:
Allowed automatic continuation:
Triggers for returning to the user:
Rollback plan:
```

Before approval:

- Do not create worker threads.
- Do not create new workspaces.
- Do not dispatch implementation tasks.
- Only refine the plan packet, task cards, ownership, and verification plan.

After approval:

- Low risk task cards can proceed without repeated user review.
- Each created workspace must be verified with its actual path, branch, base commit, and status.
- Workers do not merge directly into the target branch.

## 4. Worker Context

Workers should not receive the complete workflow guide, complete chat history, or unrelated task plans by default.

Provide only:

- The role template.
- The worker's task card.
- Relevant excerpts from the approved plan packet.
- Task-specific files or entrypoints.

Every worker prompt must state:

```text
The task card is the only execution authority.
Global material is boundary context only.
Do not expand ownership or task scope.
```

## 5. Serial Phases

Run these phases serially:

- Contract finalization.
- Shared configuration.
- Data migration.
- Branch integration.
- Integrated validation.
- Security fix review.
- Release notes.
- Publishing or target branch merge.
- Workspace and local branch cleanup.

## 6. Review Triggers

Default review rules:

- L0: controller self-check and necessary validation.
- L1: optional combined review for testing, safety, and maintainability.
- L2/L3: integration plus review, with specialist reviews selected by impact.
- High-risk: mandatory security and testing review.

Security review is required for identity, authorization, secrets, logging of sensitive values, file transfer, path handling, external sources, or deployment settings.

Testing review is required for core workflows, contracts between independent areas, data migration, or final coordinated diffs.

## 7. Closeout

Before commit, publish, merge, or cleanup, run:

```powershell
git status --short --branch
git diff --stat
git log --oneline -5
```

Only clean a workspace or local branch when it is clearly part of this task, clean, and already included in the approved target or integration branch.

