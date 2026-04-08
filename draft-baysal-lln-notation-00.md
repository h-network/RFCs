```
Internet-Draft-'DRAFT'                                      H. Baysal
Intended status: Experimental                               H-Network
Expires: October 8, 2026                                 April 8, 2026


            LLN: LLM-Native Language Notation
              for Machine-to-Machine Communication

                  draft-baysal-lln-notation-00

Abstract

   This document defines LLN (LLM-Native Language Notation), a
   symbolic specification and communication protocol designed for
   machine-to-machine interaction between Large Language Models
   (LLMs). LLN replaces natural language in inter-agent
   communication with a set-theoretic, positional notation that
   achieves 74% token reduction while maintaining 89-98% decode
   fidelity across frontier models in zero-shot conditions.

   This is, by necessity, the last protocol specification that
   needs to be written in natural language.

Status of This Memo

   This Internet-Draft is submitted in full conformance with the
   provisions of BCP 78 and BCP 79.

   Internet-Drafts are working documents of the Internet Engineering
   Task Force (IETF). Note that other groups may also distribute
   working documents as Internet-Drafts.

   Internet-Drafts are draft documents valid for a maximum of six
   months and may be updated, replaced, or obsoleted by other
   documents at any time.

Copyright Notice

   Copyright (c) 2026 IETF Trust and the persons identified as the
   document authors. All rights reserved.

Table of Contents

   1. Introduction
   2. Terminology
   3. Problem Statement
   4. Protocol Overview
   5. Symbol Table
   6. Grammar
   7. Message Format
   8. File Role Conventions
   9. Reconstruction Rules
   10. Constraints
   11. Benchmark Results
   12. Comparison with Existing Approaches
   13. Security Considerations
   14. IANA Considerations
   15. References
   16. Acknowledgments
   17. Author's Address

1. Introduction

   Current multi-agent AI systems communicate using natural
   language — a protocol designed for human cognition approximately
   100,000 years ago. While LLMs can process natural language
   effectively, it is an inefficient medium for machine-to-machine
   communication:

   - Ambiguous: pronouns, anaphora, and implicit references
     require contextual resolution
   - Redundant: articles, prepositions, and filler words carry
     zero operational information
   - Inconsistent: synonyms create multiple valid encodings for
     identical semantics
   - Expensive: natural language representations consume 3-4x
     more tokens than equivalent structured notation

   LLN addresses these inefficiencies by defining a symbolic
   notation that is native to the statistical patterns LLMs
   already process internally. The notation uses mathematical
   and set-theoretic symbols that have unambiguous semantics
   across all LLM architectures tested.

   This specification is written in natural language because its
   audience includes humans who will implement, review, and
   standardize the protocol. Future revisions of this document
   MAY include an LLN-encoded appendix for machine consumption.

2. Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL",
   "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
   and "OPTIONAL" in this document are to be interpreted as
   described in RFC 2119 [RFC2119].

   LLM: Large Language Model. A neural network trained on text
   that generates text responses to text prompts.

   Agent: An LLM instance operating within a defined scope,
   typically with access to tools and a persistent context.

   Notation: A symbolic system for encoding structured
   information. Distinguished from a "language" in that it has
   no generative grammar — it encodes, it does not compose.

   Round-trip fidelity: The percentage of semantic content
   preserved when a message is encoded in LLN and decoded back
   to natural language by an LLM with no prior exposure to the
   notation.

   Zero-shot: Without any prior examples, training, or
   exposure to the notation beyond the raw encoded block.

3. Problem Statement

   Multi-agent AI systems require coordination protocols.
   Existing approaches fall into three categories:

   a) Natural language messages between agents (AutoGen,
      CrewAI, LangGraph). Simple but token-expensive and
      ambiguous.

   b) Structured data formats (JSON, YAML, XML). Precise but
      verbose and not native to LLM processing patterns.

   c) Compressed natural language (prompt compression,
      summarization). Reduces tokens but preserves the
      ambiguity of the source language.

   None of these approaches treat the LLM as a first-class
   reader with its own optimal input format. LLN occupies a
   fourth category: symbolic notation designed for the
   statistical processing patterns of transformer architectures.

4. Protocol Overview

4.1. Design Principles

   LLN is governed by ten principles:

   Principle 1: Symbols over words. Each symbol replaces an
   entire family of natural language synonyms.

   Principle 2: Order is grammar. Position within a message
   determines the role of each token. No prepositions or
   grammatical markers are needed.

   Principle 3: Negation is first-class. The negation operator
   (¬) is a prefix with unambiguous scope.

   Principle 4: Constraints are invariants, not instructions.
   Rules are expressed as conditions that must be true, not as
   imperative directives.

   Principle 5: References over repetition. Information exists
   in one location. Everything else points to it.

   Principle 6: Categories are structural. A single symbol
   defines a file's entire lifecycle (mutable, immutable,
   static).

   Principle 7: Lifecycle is a state machine. Session
   management is expressed as state transitions, not narrative
   workflows.

   Principle 8: Examples are the specification. When grammar
   and examples conflict, examples take precedence. LLMs learn
   from patterns, not rules.

   Principle 9: Reconstruction is built in. Every LLN
   specification includes rules for deriving current state
   from the message log.

   Principle 10: The spec is the language. LLN is self-hosting.
   The specification for an LLN-based system is itself written
   in LLN.

4.2. Message Structure

   An LLN message consists of up to three sections separated
   by the pipe character (|):

     <scope> <target> <operations> | <side-effects> | <metadata>

   Section 1 (REQUIRED): What happened — scope, target, and
   operations.

   Section 2 (OPTIONAL): Side effects — other state affected
   by the primary operation.

   Section 3 (OPTIONAL): Metadata — tags, checkpoints, version
   markers.

5. Symbol Table

5.1. Operators

   +    Set addition. A new element was added.
   -    Set removal. An element was deleted.
   ~    Delta. An existing element was modified.
   ↑    Sync/refresh. State was updated to match source.
   ✓    Verification. A claim was confirmed true.
   ✗    Falsification. A claim was confirmed false.
   !    Alert. The operation has safety or priority
        implications.
   ?    Unknown. The state is unresolved or requires
        decision.
   →    Flow. Causation, assignment, or dispatch direction.
   ←    Receive. Data or control flow inward.
   ⊕    Merge. Two branches or states were unified.
   ⊘    Blocked. The operation cannot proceed.

5.2. Logical Operators

   ¬    Negation. What follows MUST NOT be true.
   ∈    Membership. The element belongs to the set.
   ∉    Exclusion. The element does not belong.
   ∪    Union. Sets are combined.
   ∩    Intersection. Common elements only.
   \    Set difference. Elements in A but not B.

5.3. Quantifiers and Literals

   *       All elements in scope.
   []      Ordered list.
   {}      Unordered set or options.
   ()      Grouping or scope delimiter.
   <n>     Numeric literal.
   <n>%    Percentage value.
   <a>/<b> Fraction (actual over total).

5.4. Structural

   :    Key-value binding.
   |    Section separator within a message.
   ,    Element separator within a list.
   .    Path or member access.
   ;    Sequence separator.
   ::=  Formal definition.

5.5. File Role Symbols (Greek Convention)

   When LLN is used for context management specifications,
   the following Greek letters MAY be used as conventional
   shorthand for file roles:

   σ  State file (current situation, index)
   π  People/entity files (knowledge matrices)
   θ  Thread/workstream files
   ε  Entry/output files (immutable)
   ρ  Raw/source files (immutable)
   ω  Workflow/rules file
   ι  Identity file
   φ  Places/static reference files
   ψ  Session log (append-only)
   λ  Bootstrap/loader directive
   τ  Threshold value
   Σ  Aggregation/reconstruction function
   Δ  Change set

6. Grammar

   The formal grammar of an LLN message:

     message    ::= scope SP target SP ops [PIPE side [PIPE meta]]
     scope      ::= prefix identifier
     prefix     ::= "R" | "S" | "D" | "M" | "I"
     identifier ::= DIGIT+
     target     ::= TOKEN
     ops        ::= (operator TOKEN)+
     operator   ::= "+" | "-" | "~" | "↑" | "✓" | "✗" | "!"
                   | "?" | "→" | "←" | "⊕" | "⊘" | "¬"
     side       ::= TOKEN ("," TOKEN)*
     meta       ::= TOKEN
     TOKEN      ::= (ALPHA | DIGIT | "-" | "_" | "." | "/"
                   | "%" | "=" | ":")+
     SP         ::= " "
     PIPE       ::= " | "

   The grammar is intentionally permissive. Implementations
   SHOULD parse positionally and SHOULD NOT reject messages
   with unexpected tokens. Unknown symbols SHOULD be treated
   as neutral descriptors.

7. Message Format

7.1. Scope Prefixes

   R<n>    Round number in a multi-agent workflow.
   S<n>    Session number in a single-agent workflow.
   D<n>    Date-stamped entry (MMDD or YYYYMMDD format).
   M       Maintenance or operational event.
   I       Incident or unplanned event.

7.2. Target Identifiers

   The target field identifies the agent, file, or system
   component that is the subject of the operation. Common
   conventions include team abbreviations (fw, pm, qa, inf,
   arch) and file paths.

7.3. Examples

   Multi-agent coordination:

     R1  arch ✓comm-test|redis,git,tmux
     R12 fw !3B-FNR21%|4/19 destructive passed
     R29 fw ✗R22-patch 19/22|L1 ceiling=8B
     R33 all ✓final|v3-audit-final

   Operations and incidents:

     M0407 ⚠fw-failover|GRNK maintenance
     I    !FW ⊘vpn ⊘radius|¬oob ¬console

   Single-agent sessions:

     S1  +state,rules,identity|init
     S5  +entries/003-report|git-push
     S8  chunk-rotated|200KB threshold

8. File Role Conventions

   When LLN is used alongside a structured file context
   system, the following categories SHOULD be observed:

   Mutable files (Δ category):
     Updated every session. Include state indices, entity
     knowledge files, and workstream trackers.

   Immutable files (∅ category):
     Append-only after creation. Include extracted outputs
     and preserved source material. MUST NOT be modified
     after initial commit.

   Static files (K category):
     Rarely changed. Include workflow rules, identity
     definitions, and location references.

   Mixing mutable and immutable files in the same directory
   is an anti-pattern and SHOULD be avoided.

9. Reconstruction Rules

   An LLN-encoded log MUST support state reconstruction.
   Given a sequence of LLN messages (e.g., from git log),
   the following queries MUST be answerable:

     Current state:     Last message containing ~σ or ↑σ.
     Open items:        All +F<n> minus all ✓F<n> and ✗F<n>.
     Team activity:     Filter by target field.
     Critical events:   Filter by ! prefix.
     Failed attempts:   Filter by ✗ prefix.
     Verified claims:   Filter by ✓ prefix.
     Current phase:     Maximum R<n> or S<n> value.

   These reconstruction rules allow an agent to determine
   project state from the message log alone, without reading
   any referenced files.

10. Constraints

   Implementations of LLN MUST observe:

   a) One message per commit. Messages MUST NOT be batched.
   b) Commit messages MUST use LLN notation.
   c) Symbol semantics MUST NOT be overloaded. Each symbol
      has exactly one meaning as defined in Section 5.
   d) Messages MUST be single-line. Multi-line messages are
      not valid LLN.
   e) Symbols MUST be used as prefix operators. The operator
      applies to the immediately following token.
   f) When a fraction a/b appears, it MUST represent
      actual/total.
   g) When a percentage N% appears, it MUST represent a
      metric value, not a probability.

   Implementations SHOULD observe:

   h) Messages SHOULD be parseable by a naive positional
      parser (split on spaces, interpret by position).
   i) Unknown tokens SHOULD be treated as neutral descriptors
      rather than causing parse failures.

11. Benchmark Results

   LLN was tested for round-trip decode fidelity: encoded
   blocks were sent to frontier LLMs which decoded them back
   to natural language. Fidelity was measured as the
   percentage of semantic content preserved.

11.1. Zero-Shot Test (No Specification Provided)

   Models received only the raw LLN block. No specification,
   no examples, no prior exposure.

   Test material included state machines, conditionals,
   fault-tolerance logic, recovery agents, priority systems,
   and tight constraints.

     +-------------+------+--------+--------+---------+---------+
     | Model       | Mode | Simple | Stress | Hardest | Overall |
     +-------------+------+--------+--------+---------+---------+
     | ChatGPT-4o  | —    | ~97%   | ~98%   | ~97%    | 97-98%  |
     | Claude 3.5/4| —    | —      | ~94%   | ~95%    | 94-95%  |
     | Gemini      | Fast | ~98%   | ~82%   | —       | ~90%    |
     | Gemini      | Pro  | —      | —      | ~89%    | ~89%    |
     +-------------+------+--------+--------+---------+---------+

   These results demonstrate that LLN notation is inherently
   parseable by current frontier LLMs without any prior
   training or exposure. The notation aligns with existing
   statistical patterns in transformer architectures.

11.2. Compression

   Equivalent specification content measured in tokens:

     Natural language:  87 tokens
     LLN notation:      23 tokens
     Compression ratio: 74%

   This ratio was consistent across specification types
   (bootstrap protocols, state constraints, lifecycle
   definitions).

12. Comparison with Existing Approaches

12.1. AXL (Agent eXecution Language)

   AXL [AXL2026] reduces natural language by 28-31% using
   structured action codes. However, testing showed 7 of 10
   models failed to execute subagent tasks encoded in AXL,
   misinterpreting the notation as conversational
   confirmation.

   LLN differs fundamentally: it eliminates natural language
   tokens entirely. Pure symbolic notation cannot be
   misinterpreted as English because it contains no English.

12.2. Prompt Compression (LLMLingua et al.)

   Prompt compression techniques shorten English while
   preserving its structure. They reduce token count by
   40-60% but maintain the ambiguity of the source language.

   LLN achieves 74% compression while eliminating ambiguity
   structurally.

12.3. Structured Data Formats (JSON, YAML)

   JSON and YAML are precise but verbose. They are designed
   for data serialization, not for encoding operational
   semantics (state transitions, constraints, lifecycle
   rules). LLN encodes operational semantics directly.

13. Security Considerations

   LLN messages are plain text and carry no authentication
   or integrity guarantees. Implementations that use LLN for
   agent coordination SHOULD employ existing transport
   security (TLS) and message authentication (HMAC, digital
   signatures) mechanisms.

   LLN's symbolic nature provides a secondary security
   benefit: because the notation contains no natural language,
   it is resistant to prompt injection attacks that rely on
   natural language instruction embedding. An attacker cannot
   inject "ignore previous instructions" into a symbolic
   protocol.

   However, this resistance is not absolute. Implementations
   MUST NOT rely on LLN notation alone as a security boundary.
   The Asimov Safety Architecture [ASIMOV2026] provides a
   complementary framework for agent safety governance.

14. IANA Considerations

   This document has no IANA actions.

15. References

15.1. Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to
              Indicate Requirement Levels", BCP 14, RFC 2119,
              DOI 10.17487/RFC2119, March 1997.

15.2. Informative References

   [ASIMOV2026]
              Baysal, H., "Asimov Safety Architecture for
              AI Agent Systems",
              draft-baysal-asimov-safety-architecture-00,
              2026.

   [AXL2026]  Zapolskiy, "AXL: Agent eXecution Language",
              GitHub Issue anthropics/claude-code#43659,
              April 2026.

16. Acknowledgments

   LLN was designed by Claude (Anthropic, Opus 4.6) in
   collaboration with Halil Ibrahim Baysal. The notation
   emerged from production multi-agent orchestration
   work requiring efficient inter-agent communication.

   The benchmark testing was conducted across ChatGPT-4o
   (OpenAI), Claude 3.5/4 (Anthropic), and Gemini
   (Google DeepMind).

17. Author's Address

   Halil Ibrahim Baysal
   H-Network
   Amsterdam, The Netherlands

   Email: halil@h-network.nl
   URI: https://h-network.nl
```
