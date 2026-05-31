# OpenState

**An open state layer for long-running agents.**

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
