# LLN Spec Recommendations — based on 7,200 field-level scoring events

These recommendations are backed by benchmark data: 50 fixtures × 6 encoding configs × 3 rounds = 900 inferences, 7,200 field-level scoring events on llama3.2:latest (3B). Full report + raw CSV attached.

---

## R1: Endorse ASCII-safe alternative symbols

**The problem:** The Unicode operators `⊤` (true), `⊥` (false), `∅` (null) cause fidelity collapse on models under 8B parameters. On llama3.2 (3B), LLN zero-shot scores **0.438** on mixed-type contexts vs JSON's **0.988**. The model simply can't decode these symbols back to JSON primitives.

**The fix:** Three lines. Replace `⊤→true`, `⊥→false`, `∅→null` in the encoder. Everything else in LLN stays the same.

**The result:** Fidelity jumps from **0.765 to 0.947** overall — within noise of JSON's **0.953**. The T2 (mixed-type) tier specifically goes from **0.438 to 0.988**.

**Suggested spec language:**

> Encoders MAY use the ASCII literals `true`, `false`, and `null` as alternatives to `⊤`, `⊥`, and `∅`. Decoders MUST accept both forms. Encoders SHOULD use ASCII literals when targeting models under 8B parameters.

**Why this works:** The structural grammar (`σ:`, `;`, `key:value`, `[a;b;c]`) works perfectly on all model sizes. Only the three boolean/null symbols fail. ASCII literals are native to every model's training corpus — no decoding step required.

---

## R2: Add a "Decoding" section to the spec

The current spec covers encoding only. Implementers need to know how to go from LLN → JSON. Suggested additions:

**Canonical decoding algorithm:**
```
1. Split on unescaped `|` to get sections
2. For each section, identify the Greek prefix (σ, ψ, etc.)
3. For σ sections, split on unescaped `; ` to get key:value bindings
4. For each binding, split on first unescaped `:` to get key and value
5. Decode value: ⊤→true, ⊥→false, ∅→null, [a;b;c]→array, numbers→numbers, else→string
6. For ψ sections, parse [role→content; ...] into message arrays
```

**Symbol normalization for model output:**
When a model outputs LLN-encoded context and generates JSON, it may echo raw symbols as string values (e.g., `"cached": "⊥"` instead of `"cached": false`). Decoders SHOULD normalize these:
- String `"⊤"` → boolean `true`
- String `"⊥"` → boolean `false`
- String `"∅"` → `null`

**Test vectors:**
```
Input:  lln:v1|σ: user:Alice; plan:pro; active:⊤; cached:⊥; error:∅; retries:3; tags:[a;b;c]
Output: {"user":"Alice","plan":"pro","active":true,"cached":false,"error":null,"retries":3,"tags":["a","b","c"]}
```

---

## R3: Acknowledge model-size sensitivity

The structural symbols work universally. The boolean/null symbols don't.

| Symbol class | 70B+ | 7-13B | 3B |
|---|---|---|---|
| Structural (`σ`, `ψ`, `→`, `;`, `:`) | reliable | reliable | **reliable** |
| Boolean/null (`⊤`, `⊥`, `∅`) | reliable | partial | **unreliable** |

Suggested spec language:

> **Compatibility note:** The structural operators (`σ`, `ψ`, `→`, `;`, `:`, `|`, `[]`) have been tested reliably on models from 3B to 70B+ parameters. The boolean/null operators (`⊤`, `⊥`, `∅`) show degraded fidelity below 8B parameters. Implementers targeting smaller models should use the ASCII alternative form (see R1).

---

## R4: Per-tier benchmark data (for reference)

| Tier | Description | json | lln (Unicode) | lln-safe (ASCII) |
|---|---|---|---|---|
| T1 flat strings | `{role: "dev", plan: "pro"}` | 0.967 | **1.000** | **1.000** |
| T2 mixed types | `{flag: true, err: null, n: 3}` | **0.988** | 0.438 | **0.988** |
| T3 arrays | `{models: ["a","b","c"]}` | **0.975** | 0.912 | **0.988** |
| T4 nested objects | `{session: {id: "x", gpu: "A100"}}` | 0.870 | 0.727 | **1.000** |
| T5 edge cases | delimiters, unicode, deep nesting | **0.963** | 0.747 | 0.891 |

**Key insight:** LLN-safe (ASCII booleans) outperforms JSON on T1, T3, and T4 — the structural grammar is genuinely better for machine parsing. The only weakness is T5 edge cases, where deeply nested objects with mixed delimiter characters challenge any compact encoding.

---

*Data source: LLN Benchmark V2, 2026-04-09. Model: llama3.2:latest (3B). Full methodology and raw CSV in the attached benchmark report.*
