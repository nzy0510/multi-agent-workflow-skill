---
name: multi-agent-workflow
description: Use when the user wants to plan, run, review, or explain a reusable coordinated agent workflow with Plan Packet Review, file ownership, isolated workspaces, worker task cards, integration, and review gates.
---

# Multi-Agent Workflow

Use this skill to coordinate agentic work without binding the process to one repository. Start with a short dispatch decision, then load only the reference needed for the task.

## Startup

1. Run `git status --short --branch`.
2. Check for a project profile or ask for one only if repository inspection cannot provide the missing constraints.
3. Decide the task level: `L0`, `L1`, `L2`, `L3`, or `High-risk`.
4. Decide dispatch mode: single agent, light coordinated, standard coordinated, or full coordinated.

## Required Output Before Dispatch

```text
Task level:
Dispatch mode:
Use multiple agents:
Enabled roles:
Reason full workflow is not enabled:
Human review gates:
Allowed automatic continuation:
```

## References

- Read `references/controller-runtime.md` for the short operating rules.
- Read `references/workflow.md` for role details, states, ownership, integration, review, and cleanup.
- Read `references/templates.md` when creating prompts, task cards, or run records.
- Read `references/project-profile-template.md` when a repository needs a project profile.

## Guardrails

- Do not create worker threads or new workspaces before Plan Packet Review when the task is L2, L3, High-risk, or an explicit workflow dry run.
- Do not assign two workers to write the same file.
- Give workers only their role template, task card, relevant plan excerpts, and task entry files.
- Return to the user for scope expansion, high risk changes, failed required validation, publishing, or destructive cleanup.
- Keep small tasks small.

