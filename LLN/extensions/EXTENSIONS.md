# LLN Extensions - Production-Discovered Additions

**Author:** Joe Wee (Tyga.Cloud Ltd)
**Date:** 2026-05-11
**Base:** LLN v1.0 (37 symbols, 74% compression on flat key-value state)
**Validation:** Three independent production systems, 3 unrelated domains, Ollama gemma4 (self-hosted)

---

## Summary

We applied LLN in production across three unrelated domains:

- **Project ATLAS** - multi-jurisdiction regulatory advisory (9 domains × 17 jurisdictions, 54 knowledge entries)
- **Project PRISM** - AI source code generation (23 categories × 2 technology stacks, 6 orchestrated agents)
- **Project ORACLE** - AI NPC engine for gaming (real-time dialogue, persistent behaviour, multi-agent)

The grammar required **zero modifications**. All extensions work within existing rules. We propose 11 additions to the spec based on patterns that emerged from production use.

| # | Extension | Source | Compression | Novel? |
|---|---|---|---|---|
| 1 | Multi-Agent Conductor Protocol | PRISM | 70% on agent messages | New message type |
| 2 | Hierarchical Dot-Notation Namespaces | PRISM | 75% on constraint data | Grammar extension |
| 3 | Chained Negation + Zero-LLM Execution | PRISM | N/A (enables regex) | New capability |
| 4 | Cross-Stack Polyglot Proof | PRISM | 379K chars, 361 sections | Validation |
| 5 | ASCII-Safe Notation Mode | ATLAS | 52.1% (same grammar) | Portability |
| 6 | Pre-Encoding Compression Layer | ATLAS | +10-20% on prose | New technique |
| 7 | Temporal Primitives | ATLAS | Part of 52.1% overall | New vocabulary |
| 8 | New Symbol: κ (kappa) | Infra | 94-97% on diagnostics | New Greek letter |
| 9 | Temporal Decay Encoding | ORACLE | Part of 66% psychology | New capability |
| 10 | Concurrent State Entries | ORACLE | Part of 66% psychology | Pattern documentation |
| 11 | Trigger Pattern Lists | ORACLE | Part of 66% psychology | New capability |

**Combined benchmark:** 52.1% (ATLAS), 64.6% (PRISM), 62% (ORACLE). Fidelity: 0.94+ on Ollama gemma4 (self-hosted).

---

## LLN v1.0 Baseline (what already exists)

For reference - the original spec by Halil Ibrahim Baysal defines:

| Category | Symbols | Examples |
|---|---|---|
| **Core operators** (14) | Negation, flow, membership, truth | `¬ → ← ∈ ∉ ∪ ∩ ⊤ ⊥ ⚠ ∅ ⊘ Δ Σ` |
| **Structural** (6) | Binding, separators, paths | `: \| , . ; ::=` |
| **Collections** (5) | Lists, sets, groups | `[] {} () * \` |
| **Greek letters** (11) | Role-based state categories | `σ θ ε ρ ω ι φ ψ λ τ π` |
| **Grammar** | Positional rules | `spec ::= header section+` |
| **Categories** | Mutability model | `Δ` mutable, `∅` immutable, `K` constant |
| **Bootstrap** | Load sequence | `λ: 1→σ 2→ω 3→ψ` |
| **Reconstruction** | Self-recovery | `Σ(state) last(Δσ)` |
| **Benchmark** | Flat key-value | 74% compression vs natural language |

**What v1.0 does NOT cover:**
- Inter-agent messaging
- Hierarchical namespaces (only flat prefixes)
- Machine-executable notation (LLM-only)
- Temporal lifecycle (effective/expires/decay)
- Multi-valued concurrent state
- Activation/trigger conditions
- ASCII portability mode
- Pre-encoding compression techniques
- Domain knowledge symbol (κ)
- Model compatibility / fidelity data

**Everything below is proposed as v1.1 additions.**

---

## Executive Summary (what we're adding)

| # | Extension | Type | Confidence | One-liner |
|---|---|---|---|---|
| 1 | **TASK/SIG/GATE** | New message types | High | Wire protocol for multi-agent orchestration. 70% compression. |
| 2 | **σ:design.*** | Grammar extension | High | Dot-notation hierarchical namespaces within σ:. 75% on constraints. |
| 3 | **Chained ¬** | New capability | High | Space-separated negations as machine-executable ban lists. Zero LLM cost. |
| 4 | **Cross-Stack Polyglot** | Validation evidence | High | Same grammar encodes 2 tech stacks (379K chars, 46 blueprints). |
| 5 | **ASCII-Safe Mode** | Portability (OPTIONAL) | Medium | Grammar works without Unicode. For edge/embedded models. |
| 6 | **Pre-Encoding Strip** | Implementation guide | Medium | Remove filler words before encoding. +10-20% on prose. Companion doc, not core spec. |
| 7 | **Temporal Primitives** | New vocabulary | High | EFF:/EXP:/NOW/UPCOMING/SUNSET for versioned data. |
| 8 | **κ (kappa)** | New Greek letter | High | Synthesised domain knowledge context. 94-97% on diagnostics. |
| 9 | **DECAY:/HEAL:** | New capability | High | Values fade over time. Temporal state without re-querying. |
| 10 | **Concurrent Entries** | Pattern documentation | Medium | Multiple lines with same prefix coexist. No \| overloading. |
| 11 | **TRIGGERS:[...]** | New capability | High | Activation conditions. Complement to ¬ exclusions. Zero-LLM match. |

**Classification key:**
- **New message types / vocabulary / capability** = Hard spec additions (add to symbol table)
- **Grammar extension** = Changes how existing symbols combine
- **Recommended practice** = Implementation guidance, not grammar
- **Validation evidence** = Proves something about the spec without changing it
- **Pattern documentation** = Formalises how existing grammar is already used

---

## Detailed Extensions

---

## Extension 1: Multi-Agent Conductor Protocol

**Problem:** Multi-agent systems need a wire protocol for task dispatch, completion signals, and verification gates. LLN has the grammar but no defined message types.

**Proposed core symbols (add to spec):**

| Symbol | Meaning | Category |
|---|---|---|
| `TASK` | Task dispatch (orchestrator → agent) | conductor |
| `SIG` | Signal/result (agent → orchestrator) | conductor |
| `GATE` | Verification checkpoint | conductor |
| `R:` | Round ID | conductor |
| `A:` | Agent ID | conductor |
| `T:` | Task type | conductor |
| `st:` | Status (ok/error/timeout) | conductor |
| `res:` | Gate result (pass/fail/skip) | conductor |

**Optional domain-specific fields (implementations MAY add):**

| Symbol | Meaning | Notes |
|---|---|---|
| `S:` | Step number | Useful for ordered pipelines |
| `G:` | Gate type | Useful when multiple gate types exist |
| `f:` | Output count | Application-specific (files, records, etc.) |
| `ms:` | Elapsed milliseconds | Performance tracking |
| `msg:` | Message/reason | Human-readable context |

**Examples:**

```
TASK|R:R5|A:codewriter|T:generate|S:3       dispatch: round 5, agent codewriter, task generate, step 3
SIG|R:R5|A:codewriter|st:ok|f:3|ms:4200     result: completed, 3 files, 4.2s
GATE|R:R5|G:validation|res:pass              gate: validation passed
GATE|R:R5|G:pre-push|res:fail|msg:"conflict" gate: failed with reason
```

**Comparison:**

Natural language (47 tokens):
> "The codewriter agent completed round 5 successfully, generating 3 files in 4.2 seconds. The design verification gate passed."

LLN (14 tokens):
```
SIG|R:R5|A:codewriter|st:ok|f:3|ms:4200
GATE|R:R5|G:design|res:pass
```

**70% compression. Zero ambiguity.**

**Production evidence:** 14-message conductor round (5 tasks + 5 signals + 4 gates) compresses from 1,433 JSON chars to 558 LLN chars (61.1%).

---

## Extension 2: Hierarchical Dot-Notation Namespaces

**Problem:** The spec uses flat Greek prefixes (`σ:state`, `θ:threads`). Complex domains need sub-namespacing without new top-level symbols.

**Proposal:** Allow `.` (member access, already in spec as path operator) within σ: bindings to create hierarchies.

**Pattern:** `σ:domain.subdomain value|value|value`

**Examples:**

```
σ:design.typo scale≥1.25×|lh:1.1|max-w:65ch|¬flat-scale
σ:design.spatial 4pt-grid(4,8,12,16,24,32,48,64,96)|vary-rhythm|¬nested-cards
σ:design.motion micro:100-150ms|ease:out-quart(0.25,1,0.5,1)|¬bounce
σ:design.color tinted-neutrals(chroma:0.005-0.015)|60-30-10|¬pure-#000
σ:design.interaction 8-states|:focus-visible|skeleton>spinner
σ:design.ban ¬gradient-text ¬glassmorphism ¬side-stripe
```

**Benchmark:** 75.0% compression across 12 sections × 2 design registers. Matches or exceeds the spec's 74% flat-state benchmark on hierarchical constraint data.

**Why dot-notation instead of new Greek letters:** Avoids symbol exhaustion. Any domain can create unlimited sub-namespaces without consuming top-level symbols. Configuration, permissions, routing, policies - all benefit from `σ:domain.sub.path`.

---

## Extension 3: Chained Negation with Zero-LLM Execution

**Problem:** The spec demonstrates single negation (`¬load(*)`). Production systems need exclusion sets - lists of banned patterns.

**Proposal:** Space-separated `¬` chains, each scoping to the immediately following token.

```
σ:design.ban ¬gradient-text ¬glassmorphism ¬side-stripe ¬bounce ¬elastic
```

Equivalent to `∉{gradient-text, glassmorphism, side-stripe, bounce, elastic}` but more compact and **machine-parseable without an LLM.**

**The key insight:** These ban lists are executable by deterministic regex matchers. 10 pattern detectors parse `¬` sequences into rules that run at zero cost, zero latency, zero tokens. LLN becomes **dual-purpose**:

1. Compressed context for LLMs (the original use case)
2. Machine-readable config for deterministic systems (new)

**Production evidence:** 10 anti-pattern detectors run on chained `¬` notation. Zero LLM calls. Zero false positives.

---

## Extension 4: Cross-Stack Polyglot Proof

**Problem:** Is LLN tied to a specific data domain, or is it genuinely stack-agnostic?

**Evidence:** The same grammar encodes two completely different technology stacks:

**Stack A (component-based):**
```
σ:section#hero min-h-[90vh] bg-gradient-to-br from-slate-950
  σ:el Button size=lg className="bg-indigo-600 hover:bg-indigo-500"
    → "Get Started"
  σ:repeat 3x → Card className="bg-slate-900/50"
```

**Stack B (class-based):**
```
σ:section#hero → section.py-5.gradient-bg, container: row.align-items-center,
  col-lg-6 (h1.display-4.fw-bold, p.lead, btn.btn-primary.btn-lg "Get Started"),
  col-lg-6 (img-fluid.rounded.shadow)
```

Both use `σ:section#id`, `σ:el`, `→` - only the VALUE syntax changes.

| Stack | Total LLN chars | σ: sections | Categories |
|---|---|---|---|
| Stack A | 270,425 | 161 | 23 |
| Stack B | 108,660 | 200 | 23 |
| **Combined** | **379,085** | **361** | **46 blueprints** |

**Note on symbols used:** `σ:section`, `σ:el`, `σ:repeat`, `→` in these examples are **domain-specific vocabulary** - not proposed as spec additions. They demonstrate that the existing `σ:` prefix + dot-notation (Extension 2) supports arbitrary domain vocabularies without grammar changes. Implementations define their own σ: sub-prefixes.

**Conclusion:** LLN is a notation **layer** - independent of target language, like JSON is independent of consumer language. This is the strongest argument for standardisation.

---

## Extension 5: ASCII-Safe Notation Mode

**Problem:** Not all LLMs handle Unicode reliably at low parameter counts. Edge-deployed models, embedded systems, and some tokenizers mangle Greek symbols.

**Proposal:** The LLN grammar supports an ASCII-safe mode where Greek symbols are replaced by uppercase prefixes.

**1:1 symbol mapping:**

| Greek (spec) | ASCII equivalent | Meaning |
|---|---|---|
| `σ:` | `STATE:` | Mutable state |
| `θ:` | `THREAD:` | Threads/workstreams |
| `ψ:` | `LOG:` | Session log |
| `λ:` | `BOOT:` | Bootstrap/load order |
| `ω:` | `RULES:` | Rules/constraints |
| `ι:` | `ID:` | Identity |
| `ρ:` | `RAW:` | Raw/source data |
| `τ:` | `TH:` | Threshold |
| `π:` | `ENTITY:` | People/entities |
| `κ:` | `KNOW:` | Knowledge context (Extension 8) |
| `¬` | `¬` (kept) | Negation (widely supported in UTF-8) |

**Domain-specific prefixes** (e.g., `R:` for rule, `B:` for band, `EFF:` for effective) are already ASCII and work in both modes.

**Production example (ASCII mode):**
```
STATE:project|MyApp|active
STATE:stack|vite-react
THREAD:active|["generation"]
BOOT: 1->STATE 2->RULES 3->ID 4->LOG[-10]
```

**Production evidence:** 52.1% compression across 54 entries × 17 jurisdictions using pure ASCII notation. Grammar unchanged.

**Recommendation:** Add an "ASCII Compatibility" section to the spec noting that implementations MAY use ASCII prefixes when Unicode is unreliable. The grammar rules (`:` binding, `|` separator, `;` sequence) are identical in both modes.

---

## Extension 6: Pre-Encoding Compression (Implementation Guide)

**Note:** This is NOT a grammar change. It's an implementation recommendation for a companion "LLN Implementation Guide" document, separate from the core spec.

**Problem:** The spec targets structured data (key-value, arrays, objects). Real-world knowledge bases contain natural language prose (notes, descriptions, conditions) that resists pure symbol replacement.

**Recommended practice:** A pre-encoding step that strips filler words before LLN encoding:

```
Input:  "The income tax personal allowance for the tax year 2025-26 is £12,570"
Strip:  "income tax personal allowance tax year 2025-26 £12,570"
Encode: "AL:Personal Allowance 12.6K"
```

The strip layer removes: articles (a, an, the), auxiliaries (is, are, was, were, has, have), prepositions (for, of, in, to, with, by, from), conjunctions (and, or, but), and other high-frequency low-information words.

**Production evidence:** Adds 10-20% compression on prose-heavy fields (notes, descriptions, conditions) on top of structural LLN encoding.

**Recommendation:** Add an "LLN Implementation Guide" companion document (not the core spec) noting that implementations MAY strip low-information words before applying LLN notation. The core grammar is unchanged.

---

## Extension 7: Temporal Primitives

**Problem:** Any domain with versioned or time-bound data (regulations, API versions, deprecation cycles, schema migrations) needs lifecycle encoding. The base spec has no temporal primitives.

**Proposed symbols:**

| Symbol | Meaning | Category |
|---|---|---|
| `EFF:` | Effective from (date) | temporal |
| `EXP:` | Expires / until (date) | temporal |
| `TY:` | Period identifier (e.g., tax year, API version) | temporal |
| `NOW` | Currently active | temporal |
| `UPCOMING` | Effective in future | temporal |
| `SUNSET` | Deprecated, still valid | temporal |
| `HIST` | Historical, no longer valid | temporal |
| `PREV` | Previous version | temporal |

**Examples:**

```
R:uk-it-bands J:UK D:IT TY:2025-26 NOW EFF:2025-04-06
R:uk-it-bands-next J:UK D:IT TY:2026-27 UPCOMING EFF:2026-04-06
R:uk-it-old J:UK D:IT TY:2024-25 HIST EXP:2025-04-05
```

**Production evidence:** 54 knowledge entries across 17 jurisdictions use temporal resolution to filter rules by query date. Upcoming changes within 365 days are surfaced as warnings.

---

## Extension 8: New Symbol - κ (kappa) = Knowledge

**Problem:** The spec's Greek letters cover state (σ), threads (θ), session logs (ψ), identity (ι), rules (ω), bootstrap (λ), raw data (ρ), and others. Missing: a symbol for **domain knowledge context** injected into agent prompts.

**Proposal:** `κ` (kappa) = knowledge/domain context. Used when encoding expert knowledge that isn't state, isn't rules, and isn't raw data - it's synthesised domain understanding.

```
κ:network{overlay:VXLAN|nodes:25|health:degraded}
κ:tax{jurisdiction:UK|domain:CGT|applicable-rules:3}
κ:design{register:brand|sections:[hero,pricing,cta]}
```

**Production evidence:** Used in infrastructure diagnostics achieving 94-97% compression on domain knowledge context (7,000+ chars of raw diagnostic JSON → 400-500 chars of κ-encoded notation).

**Why not `σ:knowledge` with dot-notation (Extension 2)?**

We considered this. Three reasons κ needs its own symbol:

1. **Different mutability category.** In the spec's category model, `σ` is `Δ` (mutable - changes on every save). Knowledge context is **ephemeral** - it's assembled per-request from multiple sources, used once, then discarded. It doesn't persist. `σ` persists. Putting ephemeral data under a persistent symbol breaks the category contract.

2. **Different lifecycle.** `σ:state` is written by the system and read later. `κ:knowledge` is derived at query time by combining σ (state) + ω (rules) + ρ (raw data) into a compressed expert context. It's a **computed view**, not stored data. The spec's reconstruction rule `Σ(state) last(Δσ)` wouldn't apply to κ because κ is never saved.

3. **Bootstrap ordering.** The `λ:` bootstrap sequence loads files in order: `1→σ 2→ω 3→ι 4→ψ[-10]`. Knowledge context (κ) is assembled AFTER bootstrap, from the loaded components. It's not a file to load - it's a synthesis step. A top-level symbol makes this distinction clear in the load sequence.

**If you prefer:** We're open to `σ:κ` (sigma sub-kappa) as a compromise - using dot-notation but with a reserved sub-namespace. Either way, the semantic distinction (synthesised ephemeral context vs persistent state) needs to be expressible.

**Relationship to existing symbols:**
- `σ` = mutable system state (persists, changes on every save)
- `ω` = immutable rules/constraints (persists, rarely changes)
- `ρ` = raw/source data (persists, immutable)
- **`κ` = synthesised knowledge context** (ephemeral, derived from σ + ω + ρ, assembled per-request, not stored)

---

## Extension 9: Temporal Decay Encoding

**Problem:** Static data has a single truth value. Dynamic systems (NPCs, IoT sensors, user preferences, cache entries) have values that **fade over time**. The spec has no way to express temporal decay.

**Proposal:** `DECAY:rate` and `HEAL:rate` suffixes on any value line. Rate is a 0-1 float representing proportion lost/gained per time unit.

**Time unit convention:** The time unit MUST be documented in the domain dictionary. The spec does not mandate a default - implementations choose what fits their domain (per-hour for real-time systems, per-day for slow-changing state, per-tick for simulations).

```
BLF:king_fair "King rules justly" CONF:0.85 DECAY:0.01        belief loses 1%/day (domain: per-day)
TRAUMA:battle SEV:0.8 TRIGGERS:[loud_noise,fire] HEAL:0.01    trauma heals 1%/day
EMO:pride:0.7:completed_sword DECAY:0.05                       emotion fades 5%/hour (domain: per-hour)
```

**Explicit unit override (optional):** Implementations MAY append a unit suffix for clarity: `DECAY:0.01/d` (per day), `DECAY:0.05/h` (per hour), `DECAY:0.1/tick` (per simulation tick).

**Why this matters:** Any system with temporal state (cache TTL, confidence decay, sentiment drift, sensor reliability) benefits from encoding decay rate alongside the value. Enables downstream systems to compute current value without re-querying: `current = original × (1 - decay)^elapsed`.

**Production evidence:** NPC psychology encoding achieves 66% compression with decay-annotated beliefs, trauma, and emotions.

---

## Extension 10: Concurrent State Entries

**Problem:** The spec models state as single-valued (`σ:status|active`). Real systems often have **multiple concurrent values** for the same dimension - emotions, priorities, confidence levels that coexist.

**Proposal:** Multiple lines with the same prefix type are concurrent by default. No `|` overloading - each entry gets its own line:

```
EMO:pride:0.7:completed_sword
EMO:worry:0.4:strange_travelers
EMO:nostalgia:0.3:old_songs
GOAL:g1 PRI:0.9 PROG:0.3
GOAL:g2 PRI:0.7 PROG:0.1 BLOCKED:[g1]
```

**Semantics:** Multiple entries sharing the same prefix (e.g., `EMO:`) coexist simultaneously with independent values. This is NOT the same as `|` (separator/or) - each line is a distinct concurrent state. The `|` operator's existing meaning (separator) is unchanged.

**Why this matters:** Emotions, priorities, risk factors, confidence levels - many real-world dimensions are multi-valued. A user can be both excited AND nervous. A system can have both high-priority AND medium-priority goals active.

**Why separate lines, not `|` on one line:** The spec defines `|` as "separator/or." Overloading it to mean "and also" creates ambiguity. Separate lines are unambiguous, greppable, and consistent with how the spec already handles multiple `σ:` entries (each on its own line).

**Production evidence:** NPCs hold 2-5 concurrent emotions with independent intensities and decay rates. Psychology encoding achieves 66% compression.

---

## Extension 11: Trigger Pattern Lists

**Problem:** Chained `¬` (Extension 3) defines what to EXCLUDE. Systems also need to define what ACTIVATES a state change - pattern-match conditions.

**Proposal:** `TRIGGERS:[pattern1,pattern2,...]` - a list of conditions that activate a state transition.

```
TRAUMA:battle SEV:0.8 TRIGGERS:[loud_noise,fire,screaming,metal_clang]
CHAIN:c1 TRIGGER:theft PENDING:3
GOAL:g2 BLOCKED:[g1]
```

When an incoming event matches any trigger pattern, the associated state activates (trauma resurfaces, consequence chain advances, goal unblocks).

**Matching semantics:** Default is **case-insensitive substring match** against the incoming event payload. Implementations MAY support additional matching modes (exact, regex, semantic) and MUST document their strategy in the domain dictionary. The spec defines the notation (`TRIGGERS:[...]`) but leaves matching strategy to implementors - consistent with how `¬` ban lists also defer execution semantics to the implementation.

**Why this matters:** This is the complement to chained `¬` (ban lists):
- `¬a ¬b ¬c` = "never these" (exclusion)
- `TRIGGERS:[a,b,c]` = "activate on these" (inclusion)

Together they give LLN a complete activation/inhibition vocabulary - useful for alerting systems, rule engines, event-driven architectures, and any conditional state machine.

**Production evidence:** Trauma trigger matching runs as deterministic pattern-match (like `¬` ban lists - zero LLM cost). If game event contains a trigger keyword, trauma state re-activates.

---

## Model Fidelity

The spec benchmarks compression ratios but not comprehension fidelity. We tested on **Ollama gemma4 (self-hosted):**

| Notation Mode | Fidelity | Tested On |
|---|---|---|
| Greek (σ/θ/ψ) | 0.94+ | PRISM, ORACLE |
| ASCII-safe (R:, B:, TH:) | 0.94+ | ATLAS |
| Mixed | 0.94+ | ORACLE |

**Fidelity** = percentage of LLN-encoded prompts where the LLM's response demonstrates correct interpretation (measured via structured output comparison against expanded-prose baseline).

**Key findings:**
1. Inline dictionary injection is **required** - without it, fidelity drops to 0.71
2. Greek vs ASCII has no fidelity difference
3. LLN compact + dictionary achieves within-noise fidelity of full-prose prompts
4. All three domains (regulatory, coding, gaming) achieve equivalent fidelity

**Recommendation:** Add a "Model Compatibility" section to the spec noting that implementations MUST include dictionary context in the system prompt.

---

## Grammar Impact Summary

| What changes | How |
|---|---|
| New Greek letter | κ (kappa) = knowledge context |
| New message types | TASK, SIG, GATE (conductor protocol) |
| New temporal symbols | EFF:, EXP:, TY:, NOW, UPCOMING, SUNSET, HIST, PREV |
| New decay/heal | DECAY:rate, HEAL:rate (temporal fading) |
| New activation | TRIGGERS:[...] (pattern-match conditions) |
| New blocking | BLOCKED:[...] (dependency graphs) |
| Grammar extension | `.` dot-notation within σ: for hierarchical namespaces |
| New capability | Chained ¬ as machine-executable ban lists |
| Pattern documentation | Concurrent entries (multiple lines with same prefix coexist) |
| Implementation guide | Pre-encoding stop-word stripping (companion doc, not core spec) |
| Portability | ASCII-safe mode (same grammar, ASCII symbols) |

| What doesn't change |
|---|
| Core operators (: \| ; # @ → [] {} ¬ Δ ∈ ⊤ ⊥ ∅) |
| Grammar rules (spec ::= header section+) |
| Position-determines-role principle |
| Categories (Δ mutable, ∅ immutable, K constant) |
| Reconstruction rules (Σ) |
| Self-hosting capability |

---

## Conformance Evidence

See `benchmarks/` directory for full production data:

- `benchmarks/BENCHMARK-ATLAS.md` - 54 entries × 9 domains × 17 jurisdictions (regulatory advisory)
- `benchmarks/BENCHMARK-PRISM.md` - 23 categories × 2 stacks × 7 data types (code generation)
- `benchmarks/BENCHMARK-ORACLE.md` - 6 data types × 62 symbols (gaming / AI NPCs)
- `benchmarks/FIDELITY.md` - model compatibility report

### Three Domains, One Grammar

| | ATLAS | PRISM | ORACLE |
|---|---|---|---|
| **Domain** | Regulatory advisory | Source code generation | Gaming / AI NPCs |
| **Data** | Rules, rates, deadlines | State, design, conductor msgs | Psychology, beliefs, social |
| **Notation** | ASCII-safe | Greek (σ/θ) | Mixed |
| **Compression** | 52.1% | 64.6% | 62% |
| **Dictionary** | 112 entries | 64 entries | 62 entries |
| **Grammar changes** | Zero | Zero | Zero |
| **LLM** | Ollama gemma4 | Ollama gemma4 | Ollama gemma4 |

**The grammar is the constant. The vocabulary is domain-specific.**

---

*Submitted: 2026-05-11 | Author: Joe Wee (Tyga.Cloud Ltd) | Base: LLN v1.0 by Halil Ibrahim Baysal (h-network)*
