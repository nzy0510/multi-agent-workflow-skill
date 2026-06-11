# Multi-Agent Workflow Skill

[中文](README.zh-CN.md) | English

A reusable Codex skill and documentation package for coordinating multi-agent software work with task levels, Plan Packet Review, file ownership, isolated workspaces, worker task cards, integration, and review gates.

This repository is intentionally project-neutral. It does not include application code, private configuration, deployment secrets, or repository-specific conventions. Use the included project profile template to adapt the workflow to a specific codebase.

## What It Provides

- A Codex skill: `skill/multi-agent-workflow`
- A short controller runtime: `docs/controller-runtime.md`
- A full workflow reference: `docs/workflow.md`
- Prompt and run-record templates: `docs/templates/`
- A project profile template: `docs/project-profile-template.md`

## When To Use It

Use this workflow when a task benefits from coordination:

- multiple independent work areas
- parallel agent work
- worktree or branch isolation
- cross-area integration
- security, testing, maintainability, or release review
- explicit user approval before high risk operations

Do not force it onto small single-file edits or simple read-only explanations. The workflow explicitly supports keeping small work small.

## Core Concepts

### Task Levels

| Level | Typical task | Default dispatch |
| --- | --- | --- |
| L0 | Small documentation, tiny fix, local explanation, narrow maintenance | Current thread or one agent |
| L1 | Workflow dry run, larger documentation, low risk tooling change | Controller plus one worker, optional combined review |
| L2 | Two or more independent areas, no high risk surface | Architect, workers, integration, review |
| L3 | End-to-end feature across many areas plus docs or release work | Full coordinated workflow |
| High-risk | Secrets, identity, authorization, data, deployment, billing, external source changes | Mandatory security, testing, integration, and release review |

### Plan Packet Review

For L2, L3, High-risk, or explicit workflow dry runs, the controller creates a plan packet before creating worker threads or workspaces. The plan packet defines:

- goal and non-goals
- acceptance criteria
- role split
- file ownership
- branch and workspace mapping
- validation commands
- high risk operations
- triggers for returning to the user
- rollback plan

## Installation

### Install From This Repository

Clone the repository:

```powershell
git clone https://github.com/nzy0510/multi-agent-workflow-skill.git
cd multi-agent-workflow-skill
```

Copy the skill directory into your Codex skills folder:

```powershell
Copy-Item -Recurse .\skill\multi-agent-workflow "$env:USERPROFILE\.codex\skills\multi-agent-workflow"
```

Restart Codex after installing so the skill is picked up.

### Update an Existing Installation

If the skill already exists, remove or rename the old local copy first:

```powershell
Remove-Item -Recurse "$env:USERPROFILE\.codex\skills\multi-agent-workflow"
Copy-Item -Recurse .\skill\multi-agent-workflow "$env:USERPROFILE\.codex\skills\multi-agent-workflow"
```

Restart Codex after updating.

### Verify the Skill Files

The installable skill lives at:

```text
skill/multi-agent-workflow/
```

It should contain:

```text
SKILL.md
agents/openai.yaml
references/controller-runtime.md
references/workflow.md
references/templates.md
references/project-profile-template.md
```

## Configure a Project for Long-Term Use

For a project that will use this workflow repeatedly, do these three setup steps once.

### 1. Add a Project Profile

Copy the template into the target project:

```powershell
New-Item -ItemType Directory -Force .\docs\agents
Copy-Item .\docs\project-profile-template.md .\docs\agents\project-profile.md
```

Then fill in `docs/agents/project-profile.md` with project-specific rules:

- project name, default language, repository root, target branch, and remote policy
- required startup checks
- major areas and default owners
- validation commands
- protected paths and data
- review triggers
- release and cleanup policy

The profile is the project-specific layer. The skill provides the generic workflow; the profile tells it how that workflow maps to the current repository.

### 2. Add an Entry in `AGENTS.md`

Add a short entrypoint to the target project's `AGENTS.md` or equivalent agent instruction file:

```md
When the user explicitly mentions "multi-agent", "parallel work", "worktree isolation", "workflow", or "Plan Packet Review", use the global `$multi-agent-workflow` skill as the generic coordination workflow and read `docs/agents/project-profile.md` as this project's adaptation layer.
```

If the project already has a local agent workflow, keep it as the project-specific detail layer and make the relationship explicit:

```md
Use `$multi-agent-workflow` for task-level dispatch and Plan Packet Review. Use `docs/agents/project-profile.md` for project paths, validation commands, protected files, and release rules. Read deeper local workflow documents only when role details or templates are needed.
```

### 3. Invoke the Skill Explicitly

In the target project, start a coordinated task with:

```text
Use $multi-agent-workflow to plan this task using the project profile. First output task level, dispatch mode, ownership, workspace mapping, validation plan, and review gates.
```

The expected first response should include:

```text
Task level:
Dispatch mode:
Use multiple agents:
Enabled roles:
Reason full workflow is not enabled:
Human review gates:
Allowed automatic continuation:
```

For L2, L3, High-risk, or explicit workflow dry runs, the next artifact should be a Plan Packet Review. Worker threads or workspaces should not be created before that plan packet is approved.

## Notes

- This package is documentation and skill metadata only.
- It does not install dependencies or require credentials.
- Repository-specific checks belong in the consuming project's profile.

## License

MIT
