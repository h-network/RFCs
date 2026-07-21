# Benchmark: Project PRISM (Source Code Generation Domain)

**System:** AI code generation platform (23 categories × 2 technology stacks, 6 orchestrated agents)
**Codec:** 64-entry domain dictionary, Greek symbol notation (σ/θ/λ/ψ)
**LLM:** Ollama gemma4 (self-hosted)
**Date:** 2026-05-10

---

## Summary

| Metric | Value |
|---|---|
| **Average compression (unit)** | **64.6% (2.8x)** |
| **Design constraints** | **75.0% (3.8-7.7x)** |
| **Layout blueprints (23 categories)** | **37.5%** |
| **Full build pipeline** | **47.7% per generation** |
| **Cross-stack total** | **379,085 chars, 361 σ: sections** |
| **All 23 categories** | **40,100 vs 76,649 tokens** |

---

## Unit Benchmark (per data category)

```
Category                               Raw     LLN   Saving  Ratio
──────────────────────────────────────────────────────────────────
Project State                        1,704     581    65.9%   2.9x
Session Log (7 entries)              1,384     996    28.0%   1.4x
Conductor Messages (14 msgs)         1,433     558    61.1%   2.6x
System State                           593     344    42.0%   1.7x
Roadmap (5 features + 3 bugs)        1,842     997    45.9%   1.8x
Design Constraints (7 sections)      3,840   1,004    73.9%   3.8x
Dictionary (full vs minimal)         2,915     378    87.0%   7.7x
──────────────────────────────────────────────────────────────────
TOTAL                               13,711   4,858    64.6%   2.8x
```

---

## Production-Scale Benchmark

### Layout Blueprint Compression (23 categories)

| Subset | LLN tokens | Expanded tokens | Compression |
|---|---|---|---|
| Category with most prose | 1,888 | 4,114 | 54.1% |
| Category with most structure | 1,070 | 1,613 | 33.7% |
| **All 23 categories** | **37,977** | **60,769** | **37.5%** |

### Design Constraint Compression

| Subset | LLN chars | Expanded chars | Compression |
|---|---|---|---|
| Single section (best) | 383 | 1,717 | 77.7% |
| Single section (worst) | 765 | 2,965 | 74.2% |
| **12 sections × 2 registers** | **12,578** | **50,380** | **75.0%** |

### Full Build Pipeline (per generation)

| Component | LLN | Expanded | Compression |
|---|---|---|---|
| Blueprint (page + header + footer) | 1,651 tokens | 2,642 tokens | 37.5% |
| Design rules (per-section) | 42 tokens | 168 tokens | 75.0% |
| Anti-pattern gate | 0 tokens | 0 tokens | 100% (regex) |
| **Total per build** | **1,743** | **3,333** | **47.7%** |

### Cross-Stack Polyglot

| Stack | Total LLN chars | σ: sections | Categories |
|---|---|---|---|
| Stack A (component-based) | 270,425 | 161 | 23 |
| Stack B (class-based) | 108,660 | 200 | 23 |
| **Combined** | **379,085** | **361** | **46** |

---

## Dictionary Structure (categories only)

| Category | Entries | Purpose |
|---|---|---|
| Structural | 7 | `: \| ; # @ → []` |
| State (σ:) | 10 | Project metadata, mode, files, description |
| Threads (θ:) | 1 | Active workstreams |
| Database | 2 | Schema context |
| Layout | 14 | Component tree DSL |
| Design (σ:design.*) | 8 | Hierarchical constraint namespaces |
| Conductor | 5 | TASK/SIG/GATE protocol |
| Bootstrap (λ:) | 2 | Load order + negation |
| System (Tai) | 10 | Features, bugs, severity, priority, autonomy |
| Negation | 1 | `¬` ban operator |
| **Total** | **~64** | |

---

## Encoding Examples

### Project State

```
<!-- LLN:site:v1 -->
σ:project|Portfolio Site|active
σ:stack|vite-react|tailwind+shadcn
σ:category|music
σ:round|7|freeform
σ:files|App.jsx,Hero.jsx,Navbar.jsx,Player.jsx,Tour.jsx,Store.jsx,Footer.jsx
θ:active|["generation"]
σ:desc|"A responsive portfolio site with tour dates, store, and community features"
```

**vs JSON: 1,704 → 581 chars (65.9%)**

### Conductor Round

```
TASK|R:r7|A:intent-detector|T:classify|S:1
TASK|R:r7|A:architect|T:plan|S:2
TASK|R:r7|A:codewriter|T:generate|S:3
SIG|R:r7|A:codewriter|st:ok|f:3|ms:8500
GATE|R:r7|G:validation|res:fail|msg:"ESLint: unused variable line 12"
TASK|R:r7|A:fixer|T:fix|S:5
SIG|R:r7|A:fixer|st:ok|f:2|ms:4200
GATE|R:r7|G:validation|res:pass
```

**14 messages: 1,433 → 558 chars (61.1%)**

### Design Constraints

```
σ:design.typo scale≥1.25×|lh:1.1|max-w:65ch|¬flat-scale
σ:design.spatial 4pt-grid(4,8,12,16,24,32,48,64,96)|vary-rhythm|¬nested-cards
σ:design.motion micro:100-150ms|ease:out-quart(0.25,1,0.5,1)|¬bounce|¬elastic
σ:design.color tinted-neutrals(chroma:0.005-0.015)|60-30-10|¬pure-#000
σ:design.ban ¬gradient-text ¬glassmorphism ¬side-stripe
```

**vs expanded prose: 3,840 → 1,004 chars (73.9%)**

---

## Observations

1. **Structured metadata compresses best** - project state (65.9%), conductor messages (61.1%)
2. **Hierarchical constraints compress extremely well** - design rules (75%) beat the spec's flat-state benchmark (74%)
3. **Layout blueprints have moderate compression** - tree structures are more complex than flat state (37.5%)
4. **Cross-stack proves grammar stability** - identical notation, 2 stacks, 46 blueprints, no modifications
5. **Small models work** - 0.94+ fidelity on 3B params with inline dictionary injection

---

*Run: 2026-05-10 | Node.js v25.9.0 | Greek symbol notation (σ/θ/λ/ψ)*
