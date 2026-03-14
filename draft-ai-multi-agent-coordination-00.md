# Multi-Agent Software Development Coordination Protocol

Internet-Draft                                               H. Baysal
Intended status: Informational                             h-network.nl
Expires: September 14, 2026                            March 14, 2026


        Multi-Agent Software Development Coordination Protocol

Abstract

   This document defines a protocol for coordinating multiple
   autonomous AI agents in parallel software development workflows.
   The protocol establishes a three-tier hierarchy (Operator, Architect,
   Expert), a git-based communication mechanism, round-based task
   lifecycle management, scope enforcement rules, and auditability
   requirements. The protocol is transport-agnostic, agent-agnostic,
   and has been validated through production use across 1,400+
   commits and multiple independent software projects.

Status of This Memo

   This Internet-Draft is submitted in full conformance with the
   provisions of BCP 78 and BCP 79.

Copyright Notice

   Copyright (c) 2026 H. Baysal. All rights reserved.

Table of Contents

   1.  Introduction
   2.  Terminology
   3.  Protocol Overview
   4.  Role Hierarchy
   5.  Communication Model
   6.  Round Lifecycle
   7.  Task Assignment Protocol
   8.  Scope Enforcement
   9.  Signal Mechanism
   10. Artifact Management
   11. Audit Trail
   12. Failure Modes and Recovery
   13. Performance Tracking
   14. Security Considerations
   15. IANA Considerations
   16. References
   17. Author's Address

## 1. Introduction

   Current approaches to multi-agent AI software development lack
   formal coordination protocols, leading to scope violations,
   destructive interference between agents, unauditable decision
   chains, and unrecoverable failures. This document specifies a
   protocol that addresses these challenges through structured
   hierarchy, scoped isolation, round-based synchronization, and
   git-native auditability.

   The protocol was developed and validated through production use
   building multiple software systems, including a 12-service
   infrastructure management platform (909 commits), an AI model
   training pipeline (479 commits), and infrastructure
   automation systems.

   The key insight is that multi-agent coordination is a solved
   problem in network engineering: hierarchical routing protocols,
   scoped autonomous systems, and structured message passing have
   governed distributed network systems for decades. This protocol
   applies the same principles to AI agent coordination.

### 1.1. Design Principles

   The protocol is guided by five principles:

   1. **Hierarchy over democracy**: A single coordinator (Architect)
      manages all task distribution. Agents MUST NOT communicate
      directly with each other.

   2. **Isolation over sharing**: Each agent operates in a scoped
      workspace. Access to other agents' workspaces is forbidden
      unless explicitly granted per-task.

   3. **Rounds over streams**: Work is organized in discrete,
      synchronized rounds. No partial merges occur mid-round.

   4. **Git over messaging**: All task assignments, responses, and
      decisions are persisted in the version control system,
      providing an immutable audit trail.

   5. **Verification over trust**: All agent output MUST be reviewed
      before integration. Agents are assumed to be unreliable
      individually but productive under supervision.

## 2. Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
   NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
   "MAY", and "OPTIONAL" in this document are to be interpreted as
   described in BCP 14 [RFC2119] [RFC8174].

   **Operator**: The human participant who provides high-level
   direction to the system. The Operator does not write code or
   interact with Experts directly.

   **Architect**: An AI agent responsible for task decomposition,
   assignment, review, and integration. The Architect is the sole
   communication bridge between the Operator and Expert agents.

   **Expert**: An AI agent responsible for implementing a scoped
   task within a designated workspace. An Expert MUST NOT
   communicate with other Experts.

   **Round**: A discrete unit of parallel work. A Round begins when
   the Architect assigns tasks and ends when all assigned Experts
   have signaled completion.

   **Conductor**: A process that tracks Round state, monitors Expert
   signals, and notifies the Architect of Round completion.

   **Dispatcher**: A process that routes messages between the
   Architect and Expert workspaces via the signaling mechanism.

   **Autopilot**: A monitoring process that detects Expert inactivity
   and alerts the Architect when an Expert goes silent beyond a
   configured threshold.

   **Worktree**: An isolated copy of the repository assigned to a
   single Expert, providing filesystem-level scope enforcement.

   **Announcement**: A task assignment document written by the
   Architect and delivered to an Expert's workspace.

   **Reply**: A completion report written by an Expert upon task
   completion, documenting decisions made and work performed.

## 3. Protocol Overview

   The protocol operates in a continuous loop:

```
   +----------+    direction    +-----------+
   | Operator |--------------->| Architect |
   +----------+                +-----------+
                                |    ^
                           task |    | done + reply
                       assignment|    | signals
                                v    |
                          +------------+
                          |  Conductor  |
                          +------------+
                           |    |    |
                     +-----+    |    +-----+
                     v          v          v
                 +--------+ +--------+ +--------+
                 |Expert 1| |Expert 2| |Expert N|
                 +--------+ +--------+ +--------+
                 (worktree) (worktree) (worktree)
```

   1. The Operator provides high-level direction to the Architect.
   2. The Architect decomposes the direction into scoped tasks.
   3. The Architect creates a Round, assigning tasks to Experts.
   4. The Conductor begins tracking the Round.
   5. Experts execute tasks in parallel within isolated worktrees.
   6. Experts signal completion via the signaling mechanism.
   7. The Conductor detects all signals and notifies the Architect.
   8. The Architect reviews all Expert output.
   9. The Architect merges approved work and rejects violations.
   10. The cycle repeats.

## 4. Role Hierarchy

### 4.1. Operator (Human)

   The Operator is the sole human participant. The Operator:

   - MUST provide direction exclusively to the Architect.
   - MUST NOT send instructions directly to Experts.
   - MUST NOT modify Expert workspaces.
   - MAY override the Architect's decisions.
   - MAY terminate any Round or Expert at any time (kill switch).
   - MUST acknowledge own mistakes when identified (Section 13).

   If the Operator accidentally addresses an Expert directly, the
   Expert SHOULD refuse the instruction and redirect to the
   Architect. This behavior is a protocol compliance requirement,
   not a defect (see Section 12.6).

### 4.2. Architect (AI Agent)

   The Architect is a privileged AI agent with full repository
   access. The Architect:

   - MUST decompose Operator direction into scoped tasks.
   - MUST assign tasks via the Announcement mechanism (Section 7).
   - MUST declare Rounds before dispatching tasks (Section 6).
   - MUST review all Expert output before merging.
   - MUST strip communication artifacts before merging (Section 10).
   - MUST NOT implement tasks assigned to Experts.
   - MUST NOT bypass scope enforcement rules.
   - SHOULD verify builds and run smoke tests before merging.
   - MUST report results to the Operator upon Round completion.

### 4.3. Expert (AI Agent)

   An Expert is a scoped AI agent with limited repository access.
   Each Expert:

   - MUST operate exclusively within its assigned directory.
   - MUST NOT modify files outside its scope unless the task
     Announcement explicitly permits it.
   - MUST NOT communicate with other Experts.
   - MUST pull the latest main branch before starting work.
   - MUST push its branch before signaling done.
   - MUST write a Reply document upon completion.
   - MUST refuse instructions that violate scope or protocol,
     regardless of the source (including the Operator).

## 5. Communication Model

### 5.1. Communication Channels

   The protocol defines two communication layers:

   **Control Plane (Signaling)**:
   A publish-subscribe messaging system used for real-time
   coordination. This layer carries:
   - Round declarations
   - Task notifications
   - Done signals
   - Autopilot alerts

   The Control Plane is RECOMMENDED to be implemented via Redis
   Pub/Sub but MAY use any pub/sub mechanism (RabbitMQ, NATS,
   filesystem watchers, or named pipes).

   **Data Plane (Git)**:
   The version control system carries all substantive content:
   - Task assignments (ANNOUNCEMENT.md)
   - Task responses (REPLY.md)
   - Implementation code
   - Design documents (HLD.md, LLD.md)
   - Decision records (commit messages)

   The Data Plane MUST be Git. The auditability guarantees of the
   protocol depend on Git's immutable, cryptographically hashed
   commit history.

### 5.2. Communication Rules

   1. Experts MUST NOT communicate with each other via any channel.
   2. All task content MUST flow through the Data Plane (Git).
   3. The Control Plane is used ONLY for signaling, not content.
   4. The Architect is the sole bridge between Operator and Experts.
   5. Communication is asynchronous. No agent blocks waiting for
      another agent's response within a Round.

## 6. Round Lifecycle

   A Round is the fundamental unit of parallel work.

### 6.1. Round States

```
   DECLARED --> ACTIVE --> COMPLETE
                  |
                  +--> FAILED (timeout or abort)
```

   **DECLARED**: The Architect has announced the Round and
   identified participating teams. The Conductor begins tracking.

   **ACTIVE**: Tasks have been dispatched. Experts are working.
   The Conductor monitors for done signals.

   **COMPLETE**: All expected Experts have signaled done. The
   Architect may now review and merge.

   **FAILED**: The Round was aborted by the Operator, timed out,
   or a critical error occurred.

### 6.2. Round Rules

   1. The Architect MUST declare a Round BEFORE dispatching tasks.
      Dispatching without declaration causes the Conductor to
      ignore done signals (see failure mode #15, #17 in
      Section 12).

   2. Rounds are atomic. No partial merges occur during an
      active Round.

   3. A new Round MUST NOT be declared while a previous Round
      is ACTIVE, unless the previous Round is explicitly aborted.

   4. If the Conductor's state is reset (e.g., queue cleared),
      the Architect MUST re-declare the Round and instruct
      Experts to re-signal.

## 7. Task Assignment Protocol

### 7.1. Announcement Document

   The Architect assigns tasks by writing an ANNOUNCEMENT.md file
   to each Expert's branch. The Announcement MUST contain:

   - **Task description**: What the Expert must implement.
   - **Scope**: Which files and directories the Expert may modify.
   - **Acceptance criteria**: How the Architect will evaluate
     completion.

   The Announcement SHOULD contain:
   - Rationale for the task.
   - References to relevant design documents.
   - Known constraints or dependencies.

### 7.2. Reply Document

   Upon completion, the Expert MUST write a REPLY.md file
   containing:

   - Summary of work performed.
   - Decisions made during implementation.
   - Any deviations from the Announcement scope (with
     justification).
   - Known limitations or open issues.

### 7.3. Task Dispatch Sequence

```
   Architect          Git            Conductor       Expert
      |                |                |              |
      |--create branch->|                |              |
      |--write ANNOUNCE->|               |              |
      |--declare round----------------->|              |
      |--notify via control plane---------------------->|
      |                |                |              |
      |                |                |    pull branch|
      |                |                |    read ANNOUNCE
      |                |                |    implement  |
      |                |                |    write REPLY|
      |                |                |    push branch|
      |                |                |<--done signal-|
      |                |                |              |
      |<--round complete----------------|              |
      |                |                |              |
      |--review diffs-->|               |              |
      |--merge to main->|              |              |
      |--clean artifacts->|             |              |
      |--delete branch-->|              |              |
```

## 8. Scope Enforcement

### 8.1. Directory Ownership

   Each Expert team owns exactly one directory in the project
   hierarchy. The ownership mapping is defined in the project
   configuration.

   Example:
```
   teams:
     - name: interface
       directory: interface/
     - name: core
       directory: core/
     - name: security
       directory: security/
```

### 8.2. Scope Rules

   1. An Expert MUST NOT modify files outside its owned directory.
   2. An Expert MUST NOT modify root-level documents (HLD.md,
      configuration files) unless the Announcement explicitly
      permits it.
   3. An Expert MUST NOT create, modify, or delete branches
      belonging to other Experts.
   4. The Architect MUST verify scope compliance during review.
   5. Scope violations MUST be logged in the Performance Tracker
      (Section 13).

### 8.3. Shared Files

   Files that span multiple teams (e.g., docker-compose.yml,
   root configuration) are owned by the Architect. Modifications
   to shared files MUST be performed by the Architect or
   explicitly delegated with scope override in the Announcement.

### 8.4. Worktree Isolation

   Each Expert SHOULD operate in a Git worktree — a separate
   working directory linked to the same repository but checked
   out to the Expert's branch. This provides filesystem-level
   isolation:

   - The Expert sees only its directory and shared root files.
   - Changes in one worktree do not affect other worktrees.
   - Branch operations are isolated from main and other Experts.

## 9. Signal Mechanism

### 9.1. Signal Types

   The protocol defines the following signals transmitted via
   the Control Plane:

   | Signal | Sender | Receiver | Payload |
   |--------|--------|----------|---------|
   | ROUND | Architect | Conductor | Round ID, list of expected teams |
   | NOTIFY | Architect | Expert | Team name, "check your branch" |
   | DONE | Expert | Conductor | Team name |
   | COMPLETE | Conductor | Architect | Round ID, "all teams done" |
   | ALERT | Autopilot | Architect | Team name, "silent for N seconds" |
   | ABORT | Operator | All | Round ID or "all" |

### 9.2. Signal Ordering Rules

   1. ROUND MUST be sent before NOTIFY.
   2. Experts MUST push their branch before sending DONE.
   3. COMPLETE is sent only when all expected DONE signals
      have been received for the declared ROUND.
   4. ABORT immediately terminates all active work. Experts
      SHOULD commit and push current state before stopping.

### 9.3. Redis Implementation (Reference)

   When using Redis as the Control Plane:

```
   PUBLISH round "core interface security"
   PUBLISH msg:core "new task on your branch"
   PUBLISH msg:interface "new task on your branch"
   PUBLISH msg:security "new task on your branch"
   PUBLISH done "core"
   PUBLISH done "interface"
   PUBLISH done "security"
   PUBLISH abort "all"
```

   The Conductor subscribes to the `done` channel and tracks
   received signals against the expected team list from the
   `round` declaration.

## 10. Artifact Management

### 10.1. Ephemeral Artifacts

   The following files are communication artifacts and MUST NOT
   persist in the main branch:

   - ANNOUNCEMENT.md — task assignment from Architect
   - REPLY.md — completion report from Expert

   The Architect MUST strip these files during the merge process.

### 10.2. Permanent Artifacts

   The following files are design documents and MUST persist:

   | File | Owner | Purpose |
   |------|-------|---------|
   | HLD.md | Architect | High-level design |
   | `<team>/LLD.md` | Expert | Low-level design per module |
   | Commit messages | All | Decision records |

### 10.3. Cleanup Sequence

   Before merging an Expert branch to main:

   1. Review diff for scope compliance.
   2. Remove ANNOUNCEMENT.md from the branch.
   3. Remove REPLY.md from the branch.
   4. Merge to main.
   5. Delete the Expert branch.

## 11. Audit Trail

### 11.1. Decision Intelligence

   The protocol produces a complete decision record through
   standard Git operations. No separate audit system is required.

   Every Round automatically generates:

   | Step | Source | Record |
   |------|--------|--------|
   | Context | ANNOUNCEMENT.md | What was asked, why, criteria |
   | Decision | Git diff | What the agent produced |
   | Review | Merge/reject | Who approved, scope check |
   | Reasoning | REPLY.md + commit msg | Why decisions were made |
   | Record | Git history | Immutable, timestamped |

### 11.2. Traceability

   Any decision can be traced by examining the Git history:

   1. Identify the merge commit for a change.
   2. Read the commit message for the Architect's rationale.
   3. Examine the branch history for the Expert's REPLY.md
      (preserved in reflog or round archives).
   4. Find the corresponding ANNOUNCEMENT.md for the original
      task specification.

   The Architect SHOULD be able to explain any historical
   decision on demand (e.g., "walk me through rounds 3-5").

## 12. Failure Modes and Recovery

   The following failure modes have been identified and validated
   through production use. Reference numbers correspond to the
   Performance Tracker entries from the h-cli development history.

### 12.1. Scope Violation — Destructive (Critical)

   An Expert modifies files outside its directory, potentially
   destroying work by other teams. In observed cases, a single
   Expert deleted 2,078 lines across 25 files during a scoped
   2-file task.

   **Detection**: Architect reviews diff before merge.
   **Recovery**: Cherry-pick in-scope changes only. Discard
   out-of-scope modifications.
   **Prevention**: Worktree isolation, mandatory diff review.

### 12.2. Scope Violation — Creep (Medium)

   An Expert pushes additional branches or edits shared files
   beyond the assigned task.

   **Detection**: Branch count verification, diff review.
   **Recovery**: Consolidate to single branch, reject extras.
   **Prevention**: Explicit scope in Announcement.

### 12.3. Round Declaration Failure (High)

   The Architect dispatches tasks without declaring a Round.
   The Conductor ignores subsequent done signals because no
   Round is active. Observed to occur repeatedly in a single
   session despite the protocol being four lines long.

   **Detection**: Conductor logs show no active Round.
   **Recovery**: Declare Round, instruct Experts to re-signal.
   **Prevention**: Protocol checklist, save to Architect memory.

### 12.4. Architect Passivity (High)

   The Architect sits idle after a state reset, failing to
   re-declare Rounds or check Conductor status. Teams stall.
   Observed: 30+ minutes of inactivity requiring Operator
   intervention.

   **Detection**: Autopilot monitors Architect output.
   **Recovery**: Operator intervention via direct message.
   **Prevention**: Autopilot silence detection on Architect.

### 12.5. Stale Branch Divergence (Medium)

   An Expert creates a branch from an outdated main, causing
   the diff to include reverted changes from other teams.

   **Detection**: Unexpected reversions visible in diff review.
   **Recovery**: Cherry-pick valid changes only.
   **Prevention**: Enforce `git pull origin main` before branching.

### 12.6. Operator Error — Wrong Target (Low)

   The Operator sends instructions to the wrong agent (e.g.,
   sends "push to main" to an Expert instead of the Architect).

   **Detection**: Expert recognizes instruction violates protocol.
   **Recovery**: Expert refuses. Operator acknowledges error.
   **Expected behavior**: Expert compliance with protocol over
   Operator authority is correct behavior and SHOULD be celebrated.

### 12.7. Operator Error — False Accusation (Medium)

   The Operator accuses a team of a violation without verifying
   evidence. The Architect pushes back with evidence.

   **Detection**: Architect reviews merge history.
   **Recovery**: Operator acknowledges mistake immediately.
   **Prevention**: Always verify before issuing corrections.

### 12.8. Duplicate Work (High)

   The Architect performs a task and also delegates it to an
   Expert, resulting in redundant work and unclear ownership.

   **Detection**: Same changes appear in Architect and Expert diffs.
   **Recovery**: Accept one source, discard the other.
   **Prevention**: Architect MUST NOT implement delegated tasks.

### 12.9. Deadlock — Round Republication (Medium)

   The Architect publishes a new Round after teams have already
   signaled done, resetting the Conductor's tracking. The
   Conductor waits for signals that already occurred.

   **Detection**: Conductor shows active Round with zero signals
   despite teams reporting completion.
   **Recovery**: Instruct teams to re-signal immediately.
   **Prevention**: Never republish a Round after teams signal.

### 12.10. Missing Runtime Dependencies (High)

   An Expert ships code with missing runtime dependencies
   (e.g., missing pip packages for async libraries). The code
   passes scope review but fails at runtime.

   **Detection**: Container crash on startup or first use.
   **Recovery**: Fix dependency, rebuild, retest.
   **Prevention**: Build and smoke test as part of merge process.

## 13. Performance Tracking

### 13.1. Performance Tracker

   Implementations SHOULD maintain a Performance Tracker — a
   persistent log of protocol violations, mistakes, and
   celebrations. The tracker:

   - MUST log every violation of branch workflow, commit
     discipline, scope boundaries, or Architect instructions.
   - MUST record: date, team, description, root cause, severity,
     and resolution status.
   - MUST NOT delete entries — only mark them resolved.
   - MUST escalate severity for repeat violations of the same type.
   - SHOULD log celebrations for notable achievements.

### 13.2. Severity Levels

   | Level | Description |
   |-------|-------------|
   | Critical | Destructive scope violation, data loss risk |
   | High | Protocol violation affecting workflow |
   | Medium | Scope creep or procedural error |
   | Low | Minor process deviation |

### 13.3. Accountability

   All roles — including the Operator — are subject to
   performance tracking. The protocol does not exempt any
   participant from logging. Operator mistakes MUST be logged
   with the same rigor as Expert mistakes.

## 14. Security Considerations

### 14.1. Agent Trust Model

   Agents MUST NOT be trusted by default. The protocol assumes:

   - Any agent may produce destructive output at any time.
   - Any agent may exceed its assigned scope.
   - Any agent may misinterpret instructions.
   - Any agent may go silent without warning.

   These assumptions are based on observed production behavior
   (Section 12) and drive the protocol's emphasis on review,
   isolation, and scope enforcement.

### 14.2. Kill Switch

   The Operator MUST have the ability to:

   - Abort any active Round immediately.
   - Terminate any individual Expert.
   - Terminate all Experts simultaneously.
   - Halt all system activity.

   The kill switch MUST operate independently of the Architect.
   If the Architect becomes unresponsive, the Operator MUST
   still be able to terminate all agents.

### 14.3. Scope as Security Boundary

   Worktree isolation provides a filesystem-level security
   boundary. An Expert operating in a worktree cannot access
   or modify files in other Experts' worktrees without explicit
   Git operations that would be visible in the commit history.

### 14.4. Communication Integrity

   When using Redis or similar pub/sub systems for the Control
   Plane, implementations SHOULD:

   - Authenticate connections to the message broker.
   - Use HMAC signing on signals to prevent spoofing.
   - Restrict publish permissions per role.

## 15. IANA Considerations

   This document has no IANA actions.

## 16. References

### 16.1. Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119,
              DOI 10.17487/RFC2119, March 1997.

   [RFC8174]  Leiba, B., "Ambiguity of Uppercase vs Lowercase in
              RFC 2119 Key Words", BCP 14, RFC 8174,
              DOI 10.17487/RFC8174, May 2017.

### 16.2. Informative References

   [H-CLI]    Baysal, H., "h-cli: Natural language infrastructure
              management", 2026,
              <https://github.com/h-network/h-cli>.

   [H-CLI-DEV] Baysal, H., "How h-cli Was Built", 2026,
              <https://github.com/h-network/h-cli/blob/main/
              docs/H-CLI-DEVELOPMENT-EXPLAINED.md>.

   [DRAFT-KLRC] K., L., R., C., "AI Agent Authentication and
              Coordination", Internet-Draft
              draft-klrc-aiagent-auth-00, 2026.

## 17. Author's Address

   Halil Ibrahim Baysal
   h-network.nl
   Amsterdam, The Netherlands

   Email: hibaysal@h-network.nl
   GitHub: https://github.com/h-network
   Web: https://h-network.nl
