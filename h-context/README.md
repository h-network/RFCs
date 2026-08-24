# h-context

Structured persistent context for AI sessions using files and git.
No vector database. No embeddings. No framework. No dependencies.

## The Problem

Every AI session starts from zero. Vector databases, embedding
stores, and knowledge graphs add infrastructure to solve a problem
that has a simpler answer: organized files.

## The Solution

A directory convention, a bootstrap file, and git.

```
project/
├── CLAUDE.md          # bootstrap — tells the AI what to load
├── rules.md           # session workflow, extraction rules
├── state.md           # current state — slim, points to detail files
├── identity.md        # who the AI is in this context
├── session-log.md     # what happened each session (append-only)
├── people/            # one file per person — knows, doesn't know
├── places/            # locations, environments, infrastructure
├── threads/           # active workstreams, issues, storylines
├── entries/           # extracted outputs — reports, chapters, logs
└── raw/               # preserved source material
```

### Three categories

| Category | Contents | Update rule |
|---|---|---|
| **Mutable state** | `state.md`, `people/`, `threads/` | Updated every session |
| **Immutable output** | `entries/`, `raw/` | Append-only |
| **Static reference** | `rules.md`, `identity.md`, `places/` | Rarely changed |

Mixing these categories causes drift. Separating them prevents it.

## Architecture

### Bootstrap

`CLAUDE.md` is small — under 1KB. It does not contain context. It
tells the AI where to find context and in what order:

```markdown
On session start:
1. Read state.md
2. Read rules.md
3. Read session-log.md
4. Read the last entry in entries/
5. Load relevant people/ and threads/ based on active state
```

The always-loaded payload stays minimal. Detail files load on demand.

### State

`state.md` stays under 2KB. It's an index, not a dump:
- Current situation
- Timeline position
- Active thread index (pointers to `threads/` files)
- Key metrics if relevant

Full descriptions live in their own files. State points to them.

### Knowledge matrix

Each file in `people/` tracks what that person knows and doesn't:

```markdown
# Jan

## Role
Site engineer

## Knows
- Project X is scheduled for April
- Firewall was replaced in January

## Doesn't Know
- Switch failover happened
- Network engineer did it before he started

## Last Interaction
May 6
```

If they weren't in the room, they don't know. If information was
shared, it's tracked: who told whom, when, in which session.

### Threads

Each active workstream gets its own file tracking status, timeline,
unresolved questions, and references to related entries. The AI
checks the thread file before responding to anything that touches
that workstream.

### Entries

Outputs are extracted, not drafted. The AI converts raw interaction
into structured output (reports, chapters, summaries). Each entry
gets its own commit. `raw/` preserves the original interaction as
source material.

### Session management

- **Start**: load files per bootstrap order, resume from `state.md`
- **Save**: update state, write changed people/threads, extract
  entries, commit to git
- **Chunking**: when session size exceeds threshold, dump history
  to `raw/`, rotate session ID, load recent chunks into next
  session prompt

Save workflow is defined in `rules.md`, not in code. The AI follows
it because it's in the prompt. Discipline over tooling.

## Git Discipline

- **One commit per logical change.** Session save, entry extraction,
  thread update — each a separate commit.
- **Commit messages describe what changed and why.** The git log
  is the session history.
- **Entries committed separately from state.** Each output is
  traceable to the session that produced it.
- **No manual edits without a commit.** Gaps in the log are gaps
  in the record.

## Why Files

**File organization is retrieval.** Directories organized by meaning
don't need semantic search. The structure is the index.

**Git is versioned memory.** Every state change is a commit. Rollback
is `git checkout`. `git blame` tells you when a fact entered context.

**Zero dependencies.** No database, no embedding model, no vector
store. `cat state.md` is the debug tool.

**Human-inspectable.** Every file is readable and editable. A new
participant reads the files and has full context in minutes.

## Applications

| Domain | people/ | threads/ | entries/ |
|---|---|---|---|
| Engineering | Colleagues, vendors | Projects, incidents | Reports, post-mortems |
| Fiction | Characters + knowledge | Storylines, mysteries | Chapters, scenes |
| Project management | Stakeholders, team | Workstreams, blockers | Status reports |
| Research | Collaborators | Hypotheses, experiments | Papers, findings |
| Education | Students, figures | Lessons, topics | Essays, assignments |

## Requirements

- An LLM with large context window (50K+ tokens recommended)
- Git
- A text editor

### Optional

- `claude -p --session-id` / `--resume` for persistent CLI sessions
- Any frontend (Telegram, web, terminal) — the interface is swappable

## What This Is Not

- **Not a vector DB replacement.** This is for structured,
  bounded, human-scale context — not millions of unstructured
  documents.
- **Not an AI framework.** No agents, no chains. A file convention
  with git discipline. It can optionally be combined with h-orchestrator for agents per task
- **Not model-dependent.** Any LLM that reads a system prompt works.

## Production

- **Collaborative fiction**: 51 sessions, 15 tracked entities with
  independent knowledge states, 7 concurrent threads. Zero
  continuity violations.
- **Multi-agent automation**: 8 parallel agents, 670+ commits,
  directory-scoped ownership, zero scope conflicts.
- **Security audit**: 33 sequential rounds, 5 agents, 13 findings
  across 3 domains. 255 commits. Blinded methodology.

Same convention. Same discipline. Same result.
