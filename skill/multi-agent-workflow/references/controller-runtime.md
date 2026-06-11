# Controller Runtime Reference

Run `git status --short --branch`, then decide task level and dispatch mode before implementation.

## Levels

| Level | Typical task | Default dispatch |
| --- | --- | --- |
| L0 | Small documentation, tiny fix, local explanation, narrow maintenance | Current thread or one agent |
| L1 | Workflow dry run, larger documentation, low risk tooling change | Controller plus one worker, optional combined review |
| L2 | Two or more independent areas, no high risk surface | Architect, workers, integration, review |
| L3 | End-to-end feature across many areas plus docs or release work | Full coordinated workflow |
| High-risk | Secrets, identity, authorization, data, deployment, billing, external source changes | Add mandatory security, testing, integration, and release review |

## Plan Packet

Required fields:

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

Before approval, do not create worker threads, new workspaces, or implementation tasks. After approval, verify every created workspace with actual path, branch, base commit, and status.

