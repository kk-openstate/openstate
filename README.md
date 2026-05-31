# OpenState

**An open state layer for long-running agents.**

[English](#openstate) | [简体中文](#简体中文)

OpenState is an early-stage open-source project exploring a simple idea:

> Long-running agents need more than larger context windows and better memory.
> They need a continuously updated, auditable state layer.

An agent working for minutes can often rely on its current context. An agent
working for days or months cannot. It needs a compact representation of its
current situation: goals, progress, active constraints, unresolved questions,
risks, evidence freshness, budgets, approvals, and the next things that deserve
attention.

OpenState is currently in the **design and validation phase**. The first MVP is
deliberately simple: a structured Markdown state document maintained alongside
an agent's work.

## The Problem

A long-running agent produces an ever-growing stream of conversations, tool
calls, observations, logs, and memories. Keeping all of this in context is
expensive and unreliable. Retrieving relevant memories helps, but memory alone
does not answer:

- What is the agent trying to achieve right now?
- Which facts are verified, and which are still hypotheses?
- What changed since the last checkpoint?
- Which evidence is stale after the environment changed?
- What is blocked or waiting for approval?
- Which actions are prohibited or too risky?
- What should receive attention next?

OpenState aims to make the agent's current operational situation explicit,
small, inspectable, and recoverable.

## State Is Not Memory

OpenState distinguishes several kinds of agent context:

| Layer | Primary question | Typical content |
| --- | --- | --- |
| Context | What can the model see during this inference? | Prompt, selected files, tool output |
| Memory | What happened before that may be relevant now? | Experiences, decisions, historical facts, reusable procedures |
| Instructions | How should the agent work in this project? | `AGENTS.md`, `CLAUDE.md`, project rules, stable workflows |
| Identity | Who is this agent? | `SOUL.md`, values, personality, long-lived behavioral boundaries |
| **State** | **What is the current situation?** | **Goal, phase, active work, risks, evidence, approvals, next attention** |

State should not duplicate source code, logs, conversations, or a complete
memory store. It should reference evidence and summarize only information that
can change the agent's next action.

## The SAM Architecture

OpenState proposes a **State-Attention-Memory** loop:

```text
State
  -> drives Attention
  -> recalls relevant Memory
  -> informs action and observation
  -> updates State
```

### State

A compact and continuously updated representation of the agent's current
situation.

### Attention

A projection of state into the questions, risks, evidence, and working set that
matter now. Attention prevents the agent from treating all historical
information as equally important.

### Memory

A store of past observations, decisions, experiences, and procedures. Memory is
recalled according to the current attention signal, then used to update state
after new evidence arrives.

SAM is not intended to replace memory systems. It provides a control loop around
them.

## Markdown MVP

The first OpenState experiment is a human-readable Markdown document:

```text
OPENSTATE.md
```

It acts as a structured, dynamic handoff document. The agent reads it when a
session begins, updates it at meaningful checkpoints, and leaves the next
session with a reliable recovery point.

An initial shape may look like:

```md
# OpenState

## User Contract
- Goal:
- Acceptance criteria:
- Non-goals:
- Constraints:

## Current State
- Phase:
- Active task:
- Blockers:
- Next actions:

## Facts And Hypotheses
### Verified facts
- Claim:
  - Evidence:

### Hypotheses
- Claim:
  - Confidence:
  - Next verification:

## Validation
- Check:
  - Status:
  - Verified at revision:
  - Freshness:

## Risks And Approvals
- Risk:
- Pending approval:
- Prohibited action:

## Checkpoint
- Updated at:
- Environment revision:
- Handoff note:
```

The MVP follows a hybrid update model:

```text
current snapshot:       update OPENSTATE.md in place
important transitions: append a lightweight timeline entry
recovery points:        save occasional immutable checkpoints
```

The document must remain concise. Full history belongs in memory, logs, version
control, or external systems.

## Update Principles

The agent should update state only when new information can change future
action, risk assessment, or recovery.

Examples:

- a goal or constraint changes
- a phase is completed
- a hypothesis is confirmed or rejected
- a meaningful test, build, or CI result arrives
- previously valid evidence becomes stale
- a blocker, risk, or approval requirement appears
- the session ends or context compaction is imminent

Not every field has the same owner:

| State type | Agent update policy |
| --- | --- |
| User goals, acceptance criteria, prohibitions | Require user confirmation |
| Objective evidence such as Git revision or test result | Update with evidence |
| Progress, blockers, next actions | Agent may update |
| Hypotheses and risk estimates | Agent may update, clearly marked as uncertain |
| High-risk decisions | Agent may propose; a human or policy must approve |
| Checkpoint timeline | Append-only |

## Initial Profiles

OpenState is intended to be domain-independent, with profiles for different
long-running agents.

### Coding Agents

Maintain task objectives, active files, verified facts, hypotheses, test
freshness, blockers, risky operations, and handoff checkpoints.

### Digital Life

Maintain current goals, commitments, relationships, active concerns, and
evolving self-understanding while keeping identity and historical memory
separate.

### Controlled-Environment Agriculture

Maintain crop phase, environmental conditions, resource budgets, equipment
health, sensor confidence, and active risks. In physical systems, OpenState
should remain a supervisory layer above deterministic real-time safety
controllers.

## Why Explore a Separate State Layer?

Existing research supports the problem, while the OpenState solution remains a
hypothesis to validate:

- Long-context models do not always use information reliably when it appears in
  the middle of a large context window:
  [Lost in the Middle](https://arxiv.org/abs/2307.03172).
- Long-horizon coding tasks remain substantially harder than isolated issue
  resolution:
  [SWE-EVO](https://arxiv.org/abs/2512.18470).
- LLMs struggle with state tracking, action effects, and longer action
  sequences:
  [ACTIONREASONINGBENCH](https://arxiv.org/abs/2406.04046).
- Tool-using agents remain inconsistent when they must follow domain policies:
  [$\tau$-bench](https://arxiv.org/abs/2406.12045).
- No single memory method works consistently across agent settings:
  [EvoMemBench](https://arxiv.org/abs/2605.18421).

OpenState asks:

> Given the same model, tools, and memory system, can an explicit operational
> state layer improve long-horizon success, recovery, efficiency, and safety?

## Evaluation Direction

The first experiments should compare:

| Group | Setup |
| --- | --- |
| A | Raw context only |
| B | Long context with running summaries |
| C | Memory retrieval |
| D | Structured `OPENSTATE.md` |
| E | Memory plus `OPENSTATE.md`, forming a SAM loop |

Useful metrics include:

- task completion rate
- recovery time after interruption
- token and tool-call cost
- repeated attempts
- duration of incorrect assumptions
- stale evidence usage
- policy violations
- human intervention count

## Relationship To Agent Memory Systems

Projects such as [agentmemory](https://github.com/rohitg00/agentmemory) provide
persistent memory infrastructure for coding agents: capturing observations,
indexing experience, recalling relevant history, and supporting cross-session
continuity.

OpenState explores a complementary layer:

```text
agentmemory: What happened before that may be relevant?
OpenState:   What is the current situation, and what should govern the next step?
```

A future integration could use OpenState attention to drive memory recall, then
use recalled experience and new evidence to update the current state.

## Roadmap

### Phase 0: Design

- [x] Define the SAM hypothesis
- [x] Distinguish state from context, memory, instructions, and identity
- [ ] Refine the minimal `OPENSTATE.md` schema
- [ ] Define update ownership and checkpoint rules
- [ ] Design the first coding-agent evaluation

### Phase 1: Markdown MVP

- [ ] Test `OPENSTATE.md` in real long-running coding tasks
- [ ] Compare baseline and SAM-assisted trajectories
- [ ] Collect failure cases and state update patterns

### Phase 2: Local State Engine

- [ ] Extract stable fields from the Markdown experiments
- [ ] Add revision history, audit events, and evidence freshness
- [ ] Integrate with coding agents through MCP, hooks, or plugins
- [ ] Add optional memory adapters

### Phase 3: Multi-Agent And Domain Profiles

- [ ] Add shared state ownership and concurrency rules
- [ ] Add policy checks, approvals, and budgets
- [ ] Explore digital-life and controlled-environment profiles
- [ ] Evaluate learned state estimation as an advisory layer

## Design Process

OpenState is being designed in public. The current collaboration principles are
recorded in [`DESIGN.md`](DESIGN.md).

## Status

OpenState is a research and design-stage project. The interfaces, schemas, and
architecture are expected to evolve as the Markdown MVP is tested.

---

## 简体中文

**OpenState：面向长周期 Agent 的开放状态层。**

OpenState 是一个处于早期设计阶段的开源项目，探索一个简单的想法：

> 长周期 agent 不仅需要更长的上下文窗口和更好的记忆系统，还需要一个持续更新、
> 可审计、可恢复的 state layer。

运行几分钟的 agent 通常可以依赖当前上下文。持续运行数天或数月的 agent
则需要一个紧凑的当前局面表示：目标、进度、约束、未解决问题、风险、证据新鲜度、
预算、审批状态，以及下一步真正值得关注的事情。

OpenState 当前处于**设计与验证阶段**。第一个 MVP 刻意保持简单：在 agent
工作目录中维护一份结构化 Markdown 状态文档。

### 为什么需要 State Layer？

长周期 agent 会不断产生对话、工具调用、观察结果、日志和记忆。将所有历史信息
放入上下文既昂贵，也不可靠。检索相关记忆可以提供帮助，但 memory 本身无法回答：

- Agent 当前究竟要完成什么？
- 哪些结论已经验证，哪些仍然只是假设？
- 环境发生变化后，哪些证据已经过期？
- 哪些事情被阻塞，或正在等待审批？
- 哪些操作被禁止，或风险过高？
- 下一步应该关注什么？

OpenState 的目标是让 agent 的当前运行态变得明确、紧凑、可检查、可恢复。

### State 不是 Memory

| 层级 | 核心问题 | 典型内容 |
| --- | --- | --- |
| Context | 本次推理可以看到什么？ | Prompt、选中的文件、工具输出 |
| Memory | 过去发生过什么，哪些历史可能与现在有关？ | 经历、决策、历史事实、可复用流程 |
| Instructions | 在这个项目中应当如何工作？ | `AGENTS.md`、`CLAUDE.md`、项目规则 |
| Identity | Agent 是谁？ | `SOUL.md`、价值观、人格、长期行为边界 |
| **State** | **当前处于什么局面？** | **目标、阶段、风险、证据、审批、下一步 attention** |

State 不应复制完整代码、日志、对话或 memory store。它只保存会改变 agent
下一步行动的信息，并引用相应证据。

### SAM 架构

OpenState 提出 **State-Attention-Memory** 循环：

```text
State
  -> 驱动 Attention
  -> 召回相关 Memory
  -> 影响行动与观察
  -> 更新 State
```

- **State**：持续更新的当前局面表示。
- **Attention**：从 state 中提取此刻真正重要的问题、风险、证据和工作集。
- **Memory**：保存过去的观察、决策、经验和流程，并根据 attention 按需召回。

SAM 不替代 memory system，而是在 memory system 之外增加一个控制循环。

### Markdown MVP

第一个实验是一份人类可读的 Markdown 文件：

```text
OPENSTATE.md
```

它是一份结构化、动态更新的交接文档。Agent 在 session 开始时读取它，在重要
节点更新它，并为下一次 session 留下可靠的恢复点。

MVP 采用混合更新方式：

```text
当前快照：      直接更新 OPENSTATE.md
关键状态迁移：  追加轻量 timeline
恢复节点：      偶尔保存不可变 checkpoint
```

文档必须保持紧凑。完整历史应进入 memory、日志、版本控制或外部系统。

### 更新原则

只有当新信息会改变后续行动、风险判断或恢复起点时，才应该更新 state。

常见触发条件：

- 用户修改目标或约束
- 一个阶段完成
- 假设被证实或推翻
- 获得重要测试、构建或 CI 结果
- 原本有效的证据变得过期
- 出现阻塞、风险或审批需求
- Session 结束或上下文即将压缩

不同字段具有不同的所有权：

| State 类型 | Agent 更新策略 |
| --- | --- |
| 用户目标、验收标准、禁止事项 | 必须由用户确认 |
| Git revision、测试结果等客观证据 | 附带证据后更新 |
| 进度、阻塞、下一步行动 | Agent 可以更新 |
| 假设、风险估计 | Agent 可以更新，但必须标明不确定性 |
| 高风险决策 | Agent 只能提出建议，由人类或策略批准 |
| Checkpoint timeline | 只追加，不改写 |

### 初始应用场景

- **Coding agent**：维护目标、活跃文件、事实、假设、测试新鲜度、阻塞、危险操作
  和交接 checkpoint。
- **数字生命**：维护当前目标、承诺、关系、活跃关注点和不断演化的自我理解，同时
  将 identity 与历史 memory 分开。
- **设施农业自动控制 agent**：维护作物阶段、环境条件、资源预算、设备健康、
  传感器可信度和风险。对于物理系统，OpenState 只能作为监督层，不能替代确定性
  实时安全控制器。

### 为什么值得探索？

已有研究支持问题的存在，但 OpenState 的解法仍然是需要验证的假设：

- 长上下文模型不总能可靠使用位于上下文中间的信息：
  [Lost in the Middle](https://arxiv.org/abs/2307.03172)。
- 长周期 coding 任务明显比单个 issue 修复更困难：
  [SWE-EVO](https://arxiv.org/abs/2512.18470)。
- LLM 在状态追踪、动作影响和较长动作序列上存在困难：
  [ACTIONREASONINGBENCH](https://arxiv.org/abs/2406.04046)。
- 工具型 agent 在遵守领域策略时仍然不稳定：
  [$\tau$-bench](https://arxiv.org/abs/2406.12045)。
- 没有一种 memory 方法在所有 agent 场景中始终有效：
  [EvoMemBench](https://arxiv.org/abs/2605.18421)。

OpenState 希望验证：

> 在模型、工具和 memory system 保持一致时，显式 operational state layer
> 是否可以提高长周期任务的成功率、恢复能力、效率和安全性？

### 与 Agent Memory 系统的关系

[agentmemory](https://github.com/rohitg00/agentmemory) 等项目为 coding agent
提供持久化 memory 基础设施：捕获 observation、索引经验、召回历史，并支持跨
session 连续性。

OpenState 探索的是互补层：

```text
agentmemory：过去发生过什么，哪些历史可能与现在有关？
OpenState：  当前是什么局面，下一步应当由什么约束和驱动？
```

未来可以由 OpenState 生成 attention，驱动 memory recall，再使用召回的经验
和新证据更新 state。

### 路线图

1. **设计阶段**：定义 SAM 假设，收敛最小 `OPENSTATE.md` schema 和更新规则。
2. **Markdown MVP**：在真实长周期 coding 任务中测试结构化 state 文档。
3. **本地 State Engine**：增加 revision、审计事件、证据新鲜度、MCP 和插件接入。
4. **多 Agent 与领域 Profile**：增加共享 state、权限、审批、预算和领域适配。

### 当前状态

OpenState 仍是一个研究与设计阶段的项目。接口、schema 和架构将随着 Markdown
MVP 的真实使用经验持续演化。
