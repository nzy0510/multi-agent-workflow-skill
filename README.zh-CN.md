# Multi-Agent Workflow Skill

中文 | [English](README.md)

这是一个可复用的 Codex skill 和通用工作流文档包，用来协调多 Agent 软件工程任务。它围绕任务分级、Plan Packet Review、文件 ownership、隔离 worktree、子 Agent 任务卡、集成和审查门禁来组织复杂工作。

本仓库保持项目中立：不包含业务代码、私有配置、部署密钥或特定仓库规则。要在具体项目中使用，请先根据 `docs/project-profile-template.md` 补充项目适配信息。

## 提供内容

- Codex skill：`skill/multi-agent-workflow`
- 主控短入口：`docs/controller-runtime.md`
- 完整工作流参考：`docs/workflow.md`
- Prompt 和运行记录模板：`docs/templates/`
- 项目适配模板：`docs/project-profile-template.md`

## 适用场景

当任务需要协作和边界控制时，适合使用这套工作流：

- 涉及多个相互独立的工作区域
- 需要并行 Agent 执行
- 需要 worktree 或分支隔离
- 需要跨区域集成
- 需要安全、测试、可维护性或发布审查
- 高风险操作前需要用户明确审批

不要把它强行套到很小的单文件修改或简单只读说明上。这套流程明确支持“小任务小处理”。

## 核心概念

### 任务分级

| 等级 | 典型任务 | 默认调度 |
| --- | --- | --- |
| L0 | 小文档、极小修复、本地说明、窄范围维护 | 当前线程或单 Agent |
| L1 | 工作流试运行、较大文档、低风险工具变更 | Controller + 一个 Worker，可选 Combined Review |
| L2 | 两个以上独立区域，不触及高风险面 | Architect + Workers + Integration + Review |
| L3 | 跨多个区域并包含文档或发布工作的完整功能 | 完整协调流程 |
| High-risk | 密钥、身份、授权、数据、部署、计费、外部来源变更 | 强制 Security、Testing、Integration、Release Review |

### Plan Packet Review

对 L2、L3、High-risk，或用户明确要求的工作流试运行，Controller 必须先生成 Plan Packet，再创建 worker 线程或 worktree。Plan Packet 至少定义：

- 目标和非目标
- 验收标准
- 角色拆分
- 文件 ownership
- 分支和 worktree 映射
- 验证命令
- 高风险操作
- 必须回到用户确认的触发条件
- 回滚方案

## 安装

### 从本仓库安装

先克隆仓库：

```powershell
git clone https://github.com/nzy0510/multi-agent-workflow-skill.git
cd multi-agent-workflow-skill
```

把 skill 目录复制到 Codex skills 目录：

```powershell
Copy-Item -Recurse .\skill\multi-agent-workflow "$env:USERPROFILE\.codex\skills\multi-agent-workflow"
```

安装后重启 Codex，让新 skill 生效。

### 更新已有安装

如果本机已经安装过同名 skill，先删除或改名旧目录：

```powershell
Remove-Item -Recurse "$env:USERPROFILE\.codex\skills\multi-agent-workflow"
Copy-Item -Recurse .\skill\multi-agent-workflow "$env:USERPROFILE\.codex\skills\multi-agent-workflow"
```

更新后同样需要重启 Codex。

### 检查 skill 文件

可安装的 skill 位于：

```text
skill/multi-agent-workflow/
```

目录中应包含：

```text
SKILL.md
agents/openai.yaml
references/controller-runtime.md
references/workflow.md
references/templates.md
references/project-profile-template.md
```

## 为项目长期接入

如果某个项目会长期使用这套工作流，建议一次性完成下面三步。

### 1. 添加项目 Profile

把模板复制到目标项目：

```powershell
New-Item -ItemType Directory -Force .\docs\agents
Copy-Item .\docs\project-profile-template.md .\docs\agents\project-profile.md
```

然后填写 `docs/agents/project-profile.md` 中的项目专属规则：

- 项目名称、默认语言、仓库根目录、目标分支和远端策略
- 启动时必须执行的检查
- 主要模块和默认 owner
- 验证命令
- 受保护路径和敏感数据
- review 触发条件
- 发布和清理策略

profile 是项目适配层。skill 提供通用流程；profile 负责说明这套流程在当前仓库里如何落地。

### 2. 在 `AGENTS.md` 中加入入口

在目标项目的 `AGENTS.md` 或同类 agent 指令文件中加入简短入口：

```md
当用户明确提到“多 Agent”、“并行开发”、“worktree 隔离”、“工作流”或 “Plan Packet Review” 时，使用全局 `$multi-agent-workflow` skill 作为通用调度流程，并读取 `docs/agents/project-profile.md` 作为本项目适配层。
```

如果项目已经有本地工作流文档，可以保留它们作为项目细则层，并明确关系：

```md
使用 `$multi-agent-workflow` 负责任务分级、调度判断和 Plan Packet Review。使用 `docs/agents/project-profile.md` 负责项目路径、验证命令、受保护文件和发布规则。只有需要角色细则或模板时，再读取更深层的本地 workflow 文档。
```

### 3. 显式调用 Skill

在目标项目中，用下面的提示启动一次协调任务：

```text
使用 $multi-agent-workflow 基于项目 profile 为这个任务制定执行方案。先输出任务等级、调度模式、ownership、worktree 映射、验证计划和审查关卡。
```

第一次输出应包含：

```text
Task level:
Dispatch mode:
Use multiple agents:
Enabled roles:
Reason full workflow is not enabled:
Human review gates:
Allowed automatic continuation:
```

对 L2、L3、High-risk，或明确要求的工作流试运行，下一步应先生成 Plan Packet Review。在该总执行包获批前，不应创建 worker 线程或 worktree。

## 使用方式

在仓库中显式调用：

```text
使用 $multi-agent-workflow 为这个任务制定多 Agent 执行方案。
```

也可以使用更短的提示：

```text
使用 $multi-agent-workflow 协调这个多模块功能。
```

```text
使用 $multi-agent-workflow 判断这个任务是否需要多 Agent。
```

为了得到更好的计划，建议同时提供：

- 任务目标
- 非目标
- 验收标准
- 是否允许创建 worker 线程或 worktree
- 是否允许 commit、push 或 release
- 已填写的项目 profile，或当前仓库的 agent 规则入口

skill 会先输出调度判断：

```text
Task level:
Dispatch mode:
Use multiple agents:
Enabled roles:
Reason full workflow is not enabled:
Human review gates:
Allowed automatic continuation:
```

对 L2、L3、High-risk，或明确要求的工作流试运行，它应先生成 Plan Packet Review，再创建任何 worker 线程或 worktree。

## 说明

- 本仓库只包含文档和 Codex skill 元数据。
- 它不安装依赖，也不需要凭据。
- 具体项目的检查命令应写在使用方项目的 profile 中。

## 许可证

MIT
