# h-conductor

Multi-agent AI orchestration using tmux, Redis, and git.
One architect coordinates N expert agents. Each agent is a
separate AI instance with scoped directory ownership.
Communication flows through git branches and a message bus.
No shared context. No agent-to-agent crosstalk.

## Concept

Most multi-agent frameworks let agents talk to each other
freely. This creates coordination chaos, scope creep, and
untraceable decision chains.

h-conductor enforces a strict hierarchy: one architect
routes all work, experts execute within their scope, and
every artifact passes through review before reaching the
shared codebase. Agents never communicate directly.

## Architecture

```
┌─────────────────────────────────────┐
│            Architect                │
│  Routes work. Reviews output.       │
│  Merges to main. Never writes code. │
└──────────┬──────────────────────────┘
           │ assigns rounds
    ┌──────┴──────────────────┐
    │      Message Bus        │
    │  Task dispatch + signals│
    └──┬─────┬─────┬─────┬───┘
       │     │     │     │
    ┌──┴┐ ┌──┴┐ ┌──┴┐ ┌──┴┐
    │ A │ │ B │ │ C │ │ D │  ← Expert agents
    └───┘ └───┘ └───┘ └───┘
     own    own   own   own  ← Each owns one directory
     dir    dir   dir   dir
```

Each expert runs in its own environment with its own context
window. Experts see only their directory and the task
assignment. They cannot read each other's work.

## Round-Based Execution

Work proceeds in atomic rounds:

1. Architect declares a round and assigns agents
2. Each assigned agent receives a scoped task
3. Agents execute independently within their directory
4. Each agent signals completion
5. Architect reviews all deliverables
6. Approved work is merged to the shared codebase
7. Communication artifacts are stripped from the codebase

No partial merges. No mid-round changes. Each round is
a clean, reviewable, revertable unit.

## Roles

**Architect**
- Owns the shared codebase and the merge process
- Routes work based on scope, not capability
- Reviews deliverables for scope compliance
- Never writes implementation code
- Maintains the decision log

**Experts**
- Each owns one directory or domain
- Receive tasks through the message bus
- Deliver work on isolated branches
- Signal completion through the message bus
- Cannot modify files outside their scope

## Coordination

The message bus handles three things:
- **Task dispatch** — architect to expert
- **Completion signals** — expert to architect
- **Health monitoring** — detect stalled agents

Git handles everything else:
- **Artifact exchange** — branches carry the work
- **Review** — architect reads the branch diff
- **Persistence** — merged work is permanent record

No shared memory. No agent-to-agent messages. No file
conflicts. The bus coordinates timing. Git carries content.

## Scope Enforcement

Each expert's scope is defined at setup:
- Which directory they own
- Which files they can modify
- What they read vs. what they write

Scope is enforced by architect review at merge time, not
by filesystem permissions. The architect checks every diff
against the declared scope before merging.

## Autopilot

An optional watchdog monitors agent activity. If an agent
is silent beyond a threshold, the architect is notified.
The architect decides whether to resend, reassign, or
escalate — the watchdog only observes.

Bounded autonomy: the architect can execute a declared
sequence of rounds without operator input, stopping at
defined checkpoints. The operator sets the autonomy window.

## What This Produces

- Clean commit history with one logical change per commit
- Full audit trail of who did what and why
- Zero scope-violation merge conflicts
- Parallel execution where tasks are independent
- Sequential execution where tasks are coupled
- Operator attention used for decisions, not coordination

## Production

- 8 parallel agents, 670+ commits, zero scope conflicts
- 5 agents, 33 rounds, 13 auditable findings
- Single-agent to full-team scaling within the same framework

## Requirements

- AI instances that accept scoped prompts
- A message bus
- Git
- A terminal multiplexer for the human viewport
