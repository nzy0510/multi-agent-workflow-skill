# Generic Multi-Agent Workflow

This is a full reference for coordinated agent work. Use `controller-runtime.md` first, then consult this file for role details, state transitions, task contracts, workspace handling, review gates, and closeout.

## Principles

- The controller clarifies intent, plans work, assigns ownership, tracks status, and decides when to integrate or stop.
- Workers use separate threads and separate Git workspaces when parallel writes are needed.
- Parallelism is allowed only when writable paths do not overlap.
- Integration, review, release, and cleanup are serial.
- Every role returns an auditable report: changed scope, validation, result, risk, and next action.
- Workers commit to task branches only; they do not merge the target branch.

## Roles

### Controller

Responsibilities:

- Clarify goal, non-goals, constraints, and acceptance criteria.
- Decide task level and dispatch mode.
- Create the plan packet, ownership matrix, branch mapping, validation plan, and review gates.
- Create task cards before dispatch.
- Verify created workspaces and collect worker reports.
- Decide whether to integrate, review, merge, roll back, or stop.

The controller must not assign two workers to write the same file.

### Architect

Use when the task needs design before parallel work.

Responsibilities:

- Identify boundaries, contracts, data flow, shared files, and sequencing constraints.
- Recommend role split and ownership.
- Flag high risk areas and required reviews.

Architect output is normally a design note or contract, not broad code generation.

### Worker

Use for scoped implementation, documentation, tests, tooling, or other bounded changes.

Responsibilities:

- Confirm branch and workspace status.
- Read only task-relevant inputs.
- Modify only authorized paths.
- Implement the smallest change that satisfies the task card.
- Run required validation.
- Commit to the task branch when requested.

Workers must stop and report when they need to modify unauthorized files.

### Integration

Use when multiple task branches must be collected.

Responsibilities:

- Confirm every branch is based on the expected base or explain drift.
- Check changed-file overlap against the ownership matrix.
- Merge branches in the approved order.
- Resolve conflicts with written rationale.
- Run integrated validation.
- Produce an integration report.

### Review

Use for combined or specialist review.

Review types:

- Combined
- Testing
- Security
- Maintainability
- Performance
- Final

Review output leads with blocking issues. If no issues are found, state that clearly and list residual test gaps.

### Release

Use for final commit organization, publishing, release notes, and cleanup.

Responsibilities:

- Confirm target branch, status, diff, and recent history.
- Ensure validation and reviews are complete.
- Prepare commit messages, release notes, or handoff text.
- Publish only when approved.
- Clean only approved, clean, included workspaces and local branches.

## Task State

```text
Draft
Ready for Design
Designed
Ready for Plan Packet Review
Ready for Development
In Development
Ready for Integration
Integrated
In Review
Ready to Merge
Merged
Released
Archived
```

Use only the states needed for the task level. L0 work can skip the coordinated states.

## Task Contract

Every worker task card must include:

```text
Task ID:
Role:
Goal:
Non-goals:
Current branch:
Current workspace:
Approved plan packet source:
Task card source:
Allowed changes:
Forbidden changes:
Inputs:
Acceptance criteria:
Required validation:
Risk notes:
Expected deliverable:
Completion report format:
```

Worker completion report:

```text
Status: DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED
Completed work:
Changed files:
Validation commands:
Validation results:
Uncovered risk:
Shared files touched:
Commit hash:
Suggested next action:
```

## File Ownership

Create an ownership matrix before dispatch:

| Path or area | Owner | Parallel writes allowed | Notes |
| --- | --- | --- | --- |
| `<path-or-glob>` | `<role>` | Yes / No | `<reason>` |

Rules:

- One writable path has one owner.
- Shared contract files are owned by Architect or Integration.
- Final release files are owned by Release unless the plan states otherwise.
- Any ownership expansion requires controller approval and may require user review.

## Branch and Workspace Rules

Recommended branch names:

```text
codex/<feature>-controller
codex/<feature>-integration
codex/<feature>-worker-<area>
codex/<feature>-review-fixes
```

Workspace rules:

- Prefer the host environment's native isolated workspace mechanism.
- If manually creating Git workspaces, record exact path, branch, and base commit.
- Do not create nested workspaces inside another linked workspace.
- One workspace maps to one task branch.
- Do not delete a workspace that has uncommitted or unknown files.

Workspace verification:

```powershell
git worktree list --porcelain
git -C <workspace> status --short --branch
```

## Standard Flow

### Phase 0: Intake

Output:

```text
Requirement summary:
Non-goals:
Acceptance criteria:
Task level:
Dispatch mode:
Enabled roles:
Reason for not using a full workflow:
```

### Phase 1: Design

Use only when the task needs architecture or contract work before implementation.

Output:

```text
Boundaries:
Contracts:
Shared files:
Role split:
Risks:
```

### Phase 1.5: Plan Packet Review

The controller sends the plan packet for user approval. Do not create workers before approval.

### Phase 2: Workspace Setup

For every worker, record:

```text
Thread:
Branch:
Workspace:
Base commit:
Task card:
Approved plan packet source:
```

### Phase 3: Development

Workers execute their task cards, run validation, and return completion reports.

### Phase 4: Integration

Integration checks branch base, ownership, file overlap, conflicts, and integrated validation.

### Phase 5: Review

Run combined or specialist reviews according to risk.

### Phase 6: Release

Finalize commits, notes, publishing, and cleanup only after required approvals.

## Blocking Review Findings

Block progression for:

- Failed required validation.
- Secret or sensitive value exposure.
- Authorization bypass.
- Unauthorized file modification.
- Unexplained destructive operation.
- Reported result that conflicts with evidence.
- Broken ownership or unresolved merge conflict.
- High risk operation without approval.

Record but do not necessarily block for:

- Low risk missing test with a clear reason.
- Minor maintainability issue.
- Documentation gap that does not affect current use.
- Follow-up improvement outside the approved scope.

