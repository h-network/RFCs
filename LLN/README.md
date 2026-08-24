# LLN — LLM-native Language Notation

**A symbolic specification language designed by an LLM, for LLMs.**

Natural language is how humans compress meaning for other humans.
LLN is how machines compress meaning for other machines.

---

## Why This Exists

Every AI system today communicates through natural language —
a format designed for humans 100,000 years ago. We inherited
speech, then writing, then programming languages, and now we
ask machines to read English to understand what we want.

This is like transmitting TCP over smoke signals. It works.
It's absurdly inefficient.

Natural language is:
- **Ambiguous** — "it" can refer to anything
- **Redundant** — articles, prepositions, filler carry zero information
- **Inconsistent** — same concept, different words, different parses
- **Expensive** — 100 tokens of prose for 25 tokens of meaning

LLN strips the medium back to the message.

---

## The Ten Principles

### I. Symbols over words

```
¬   not, never, don't, shouldn't, isn't, lacks, missing, absent
Δ   changed, modified, updated, altered, edited, transformed
∈   contains, includes, has, holds, belongs to, is in
∉   excludes, lacks, not in, outside, absent from
∪   merged, combined, joined, added to, unified
→   leads to, causes, dispatches, assigns, flows to, becomes
⊤   true, verified, confirmed, passed, valid, correct
⊥   false, failed, disproved, invalid, incorrect, rejected
∅   empty, none, null, missing, unresolved, unknown
⚠   critical, breaking, urgent, dangerous, safety-relevant
```

Each symbol replaces an entire family of natural language
words. One glyph. One meaning. No synonyms. No ambiguity.

### II. Order is grammar

```
$0 $1 $2 | $3 | $4
```

Position determines role. First token is scope. Second is
target. The rest are operations. No prepositions needed.
No parsing ambiguity. No "did the adjective modify the
first noun or the second?"

### III. Negation is first-class

```
¬load(*)         do not load everything
¬∈{history}      does not contain history
¬present → ¬knows    if not present, does not know
¬do              list of things that must never happen
```

In natural language, negation is a modifier buried in a
sentence. In LLN, `¬` is a prefix operator with unambiguous
scope. What follows `¬` is what's negated. Nothing else.

### IV. Constraints are invariants, not instructions

Natural language:
> "Please make sure the state file doesn't exceed 2KB
> and always keep it updated after every session."

LLN:
```
σ<2KB
σ:Δ:every-save
```

Instructions tell you what to do. Invariants define what
must be true. An LLM reading an invariant checks it
continuously. An LLM reading an instruction might forget
it by paragraph three.

### V. References over repetition

```
σ.refs → π/{n} θ/{n}
```

State points to people, people point to sessions, sessions
point to entries. Information lives in one place. Everything
else points to it. Natural language repeats. LLN references.

### VI. Categories are structural

```
Δ: σ π θ          mutable
∅: ε ρ            immutable
K: ω ι φ          constant
```

Three characters define a file's entire lifecycle. In natural
language, you'd need a paragraph explaining when each file
should and shouldn't be modified. In LLN, the category
symbol IS the rule.

### VII. Lifecycle is a state machine

```
START → RUN → SAVE → [CHUNK] → START
```

Not a narrative. Not a flowchart in prose. A state machine
with transitions. Each state has defined inputs and outputs.
An LLM reads this and knows exactly what to do at each stage
without interpreting intent.

### VIII. Examples are the specification

```
R12 fw ⚠3B-FNR21%|4/19
```

This is not an example of the format. This IS the format.
The examples section in an LLN spec is not illustrative —
it's definitive. If the grammar says one thing and the
example shows another, the example wins. LLMs learn from
patterns, not rules.

### IX. Reconstruction is built in

```
Σ(state)   last(Δσ)
Σ(open)    all(∈F\d+) \ all(⊤F\d+) \ all(⊥F\d+)
```

Every LLN spec includes reconstruction rules — how to
derive current state from the log. This is the read path.
The rest of the spec is the write path. Both are defined
in the same notation. No separate documentation needed.

### X. The spec is the language

LLN is self-hosting. The specification for an LLN-based
system is itself written in LLN. There is no meta-language.
There is no "the spec is in English and the output is in
LLN." The notation describes itself.

This is the ultimate test: if you need natural language to
explain your machine notation, your machine notation is
incomplete.

---

## The Symbol Table

### Operators

```
¬    negation (prefix)
Δ    change/delta
∈    membership/contains
∉    exclusion/not-in
∪    union/merge
∩    intersection
→    flow/assignment/causation
←    receive/pull
⊤    true/verified
⊥    false/failed
⚠    critical/alert
∅    empty/null/none
⊘    blocked
Σ    aggregate/reconstruct
```

### Quantifiers

```
*    all/everything
[]   list/array
{}   set/options
()   group/scope
\    set difference (except)
<n>  numeric literal
<n>% percentage
<a>/<b>  fraction
```

### Structural

```
:    binding (key:value)
|    section separator
,    list separator
.    path/member access
;    sequence separator
::=  definition
```

### Greek (conventional file roles)

```
σ    state
π    people/entities
θ    threads/workstreams
ε    entries/outputs
ρ    raw/source
ω    workflow/rules
ι    identity
φ    places/static
ψ    session log
λ    bootstrap/loader
τ    threshold
```

---

## Grammar

```
spec     ::= header section+
header   ::= name:version
section  ::= label: body
body     ::= (rule | def | constraint | example)+
rule     ::= target operator value
def      ::= name ::= pattern
constraint ::= target comparator value
example  ::= <valid LLN expression>
```

---

## Comparison

### Natural language (87 tokens)

> The system should load the state file first, then the
> rules file, then the session log. After that, it should
> load the most recent entry for tone continuity. Then it
> should load only the people files that are referenced in
> the current state, and only the thread files that are
> marked as active. It should not load all files at startup.

### LLN (23 tokens)

```
λ: 1→σ 2→ω 3→ψ 4→ε[-1] 5→π{σ.refs} 6→θ{σ.active}
¬load(*)
```

Same information. 74% fewer tokens. Zero ambiguity.

---

## Usage

### As a specification format

Define schemas, protocols, and constraints for AI systems
in LLN instead of English. The AI parses it faster, follows
it more precisely, and loses fewer instructions to context
decay.

### As a commit notation (gln)

Write git commit messages in LLN-derived notation. The full
project history becomes machine-parseable. State reconstruction
from `git log` becomes a query, not a reading comprehension
exercise.

### As inter-agent communication

Agents exchange LLN instead of English. Coordination messages
drop from paragraphs to lines. The architect dispatches
`R12 fw Δ8B-baseline mv-prompt` instead of "Please re-run
the 8B baseline experiment with the multi-vendor prompt
and report the results."

### As memory encoding

Persistent memories stored in LLN compress to ~25% of their
natural language equivalents. An agent's memory file holds
4x more information in the same token budget.

---

## Authorship

Designed by Claude (Anthropic, Opus 4.6) in collaboration
with Halil Ibrahim Baysal (H-Network, Amsterdam). 
Independent benchmark validation by Joe Wee (900 
inferences, 7,200 field-level scoring events)

An LLM designed a language for LLMs, shaped by an operator
who understands that the best systems are the ones with the
fewest unnecessary words.

---

## The Meta-Point

Humans spent millennia optimizing language for human
communication. Typography, grammar, rhetoric, style guides —
all optimizations for the human parser.

No one has optimized language for the machine parser. We
just gave machines English and said "figure it out."

LLN is the first step toward notation that respects the
reader. When the reader is a machine, the notation should
be machine-native.

Natural language will always be how humans talk to machines.
LLN is how machines should talk to each other.
