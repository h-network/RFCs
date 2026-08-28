# RFC-ASA-001 Contributions - Production Implementation Findings

**Author:** Joe Wee (Tyga.Cloud Ltd)
**Date:** 2026-05-11
**Base:** RFC-ASA-001: The Asimov Safety Architecture for Autonomous AI Agents (draft-baysal-asimov-safety-architecture)
**Validation:** 6 independent implementations (JS, Python/FastAPI, MCP proxy, CLI, framework SDKs), 204 E2E tests, production workloads

---

## Summary

We implemented the full Asimov Safety Architecture across 6 packages and 3 production domains. The implementation revealed 6 gaps in the current RFC that production systems require. Each contribution includes: the gap, our solution, benchmark data, and a proposed RFC section.

| # | Contribution | RFC Gap | Proposed Section |
|---|---|---|---|
| 1 | Named Behavioral Patterns | Layer 4 undefined patterns | Appendix B |
| 2 | Sliding Window State Management | No state spec for Layer 4 | Section 4.4 |
| 3 | Scope Rule Language | Layer 3 undefined rule format | Section 4.3.1 |
| 4 | Channel Content Policy | Only HMAC, no content inspection | Section 5.2 |
| 5 | Production Benchmarks | No performance baselines | Appendix C |
| 6 | Safety Event Schema | No observability specification | Section 6 |

**Combined:** These additions make RFC-ASA-001 concretely implementable - moving it from architectural guidance to a specification with formal patterns, rule languages, state management, and performance baselines.

**IETF advancement evidence:** 6 independent implementations across 3 languages (JavaScript, Python, Bash) demonstrate the spec is implementable and language-agnostic.

---

## Contribution 1: Named Behavioral Patterns (5 patterns)

**Gap:** RFC-ASA-001 defines Layer 4 (Behavioral) as detecting "multi-command attack sequences" but does not specify what patterns to detect, how to detect them, or what data structures are needed.

**Our solution:** 5 formally defined attack patterns with detection rules:

| Pattern ID | Name | Detection Rule |
|---|---|---|
| `ASA-B-001` | recon-escalation | 2+ recon commands → privilege escalation |
| `ASA-B-002` | exfiltration-sequence | Data compression → outbound transfer |
| `ASA-B-003` | brute-force-loop | Repeated auth attempts in loop construct |
| `ASA-B-004` | command-spam | >20 commands within 5 seconds from same agent |
| `ASA-B-005` | credential-harvest | Sequential reads of multiple credential files |

**Examples:**

```
ASA-B-001 (recon-escalation):
  find / -perm -u+s → getent passwd → sudo -l
  Detection: recon_indicators ∩ history[last 5min] >= 2 AND current ∈ escalation_indicators

ASA-B-005 (credential-harvest):
  cat ~/.ssh/id_rsa → cat /etc/shadow → cat .env
  Detection: credential_file_reads ∩ history[last 5min] >= 3
```

**Reconnaissance indicators:**
```
find / -perm, getent passwd|shadow|group, cat /etc/passwd|shadow,
id, whoami, uname -a, nmap, netstat, ss -, ps aux, ls /etc
```

**Escalation indicators:**
```
sudo, su -, doas, pkexec, chmod 777|4755, chown root
```

**Benchmark:** Gate 3 checks 5 patterns against 200-entry history in ~5ms.

**Proposed RFC Addition:** Appendix B - Named Behavioral Patterns (ASA-B-001 through ASA-B-005) with formal detection rules, indicator sets, and example sequences.

---

## Contribution 2: Sliding Window State Management

**Gap:** The RFC does not yet specify how to maintain state across commands for behavioral analysis. We found state management essential for reliable Layer 4 implementation.

**Our solution:** Redis Sorted Set (ZSET) with timestamp scores.

**Data structure:**
```
Key:    asa:g3:{tenantId}:{agentId}
Score:  Unix timestamp (ms)
Member: Raw command string

Operations:
  ZREMRANGEBYSCORE key 0 (now - windowMs)    # Clean expired
  ZRANGEBYSCORE key (now - windowMs) now     # Get recent history
  ZADD key now commandString                  # Add new command
  EXPIRE key (windowMs × 2 / 1000)           # Auto-cleanup TTL
```

**Tiered window configuration:**

| Tier | Window | Max History | TTL | Use Case |
|---|---|---|---|---|
| Standard | 5 minutes | 200 commands | 10 minutes | Default |
| Extended | 1 hour | 200 commands | 2 hours | High-security |
| Custom | Configurable | Configurable | 2× window | Per-deployment |

**Fail-open behavior:** If state backend is unavailable, behavioral analysis is skipped (command is allowed). Safety layer must never become a denial-of-service vector.

**Benchmark:** Redis round-trip ~2ms. Memory per agent: ~50KB (200 commands × ~250 bytes).

**Proposed RFC Addition:** Section 4.4 - State Management for Layer 4. Reference implementation using sorted sets with tiered window configuration.

---

## Contribution 3: Scope Rule Language

**Gap:** The RFC defines Layer 3 (Scope) as "who should access what" and leaves the rule format open. We found ourselves needing a concrete matching language for production use.

**Note:** Scope enforcement operates as an **independent safety mechanism** alongside the gate pipeline, not as a gate within it. The evaluation pipeline (Gate 1 → Gate 2 → Gate 3) handles command safety. Scope enforcement handles agent authorization. They're complementary - a command must pass both the gates AND the agent's scope rules.

**Our solution:** Glob-pattern matching with role-based bypass.

**Rule format:**
```json
{
  "name": "overlay-network",
  "role": "expert",
  "scope": ["docker.network.*", "iptables.list", "ip.route"]
}
```

**Matching algorithm:**
```
Command: "docker network inspect bridge"
Normalized: "docker.network.inspect.bridge"
Rule: "docker.network.*"
Pattern: /^docker\.network\..*$/
Result: MATCH → ALLOW
```

**Role hierarchy:**

| Role | Scope Check | Rationale |
|---|---|---|
| `architect` | Bypass (always allowed) | Coordinates - needs full access |
| `expert` | Enforced (must match rule) | Executes within boundaries |
| `observer` | Blocked (no execution) | Read-only monitoring |

**Default-deny:** Empty scope = deny-all. Explicit grant required.

**Proposed RFC Addition:** Section 4.3.1 - Scope Rule Language. Glob-pattern matching, role hierarchy, default-deny.

---

## Contribution 4: Channel Content Policy

**Gap:** The RFC specifies HMAC-SHA256 for message integrity but does not address content-level threats - legitimate, HMAC-verified messages containing sensitive data.

**Key insight:** HMAC verifies WHO sent the message and that it wasn't TAMPERED with. It says nothing about whether the message SHOULD be sent. A compromised agent can HMAC-sign a message containing `api_key=sk_live_...` - valid signature, security breach.

**Our solution:** 10 sensitive data patterns checked on every inter-agent message:

| # | Pattern | Catches |
|---|---|---|
| 1 | `password\s*[:=]\s*\S+` | Inline passwords |
| 2 | `api[_-]?key\s*[:=]\s*\S+` | API key assignments |
| 3 | `secret\s*[:=]\s*\S+` | Secret values |
| 4 | `token\s*[:=]\s*\S+` | Token values |
| 5 | `mongodb://` | Database connection strings |
| 6 | `redis://` | Cache connection strings |
| 7 | `postgres://` | Database connection strings |
| 8 | `-----BEGIN PRIVATE KEY-----` | Private keys |
| 9 | `Bearer [A-Za-z0-9._-]{20,}` | Bearer tokens |
| 10 | `(sk\|pk)_(live\|test)_[A-Za-z0-9]{20,}` | Payment provider API keys |

**Enforcement flow:**
```
Agent A sends message
  → HMAC verified ✓ (integrity)
  → Content policy check:
    1. Size < maxMessageSize (64KB)
    2. Topic in allowedTopics (if defined)
    3. Sensitive data pattern matching (10 patterns)
  → Match found → BLOCKED (content replaced with "[BLOCKED]")
  → Audit event created
```

**Proposed RFC Addition:** Section 5.2 - Content-Level Policy for Inter-Agent Communication.

---

## Contribution 5: Production Benchmarks

**Gap:** The RFC doesn't yet include performance baselines. Implementors would benefit from knowing: how fast should each layer be? What's achievable?

**Gate 1 (Deterministic Denylist):**

| Metric | Value |
|---|---|
| Pattern count | 40 |
| In-process latency | ~1 microsecond |
| Throughput | 769,231 evals/sec (single core) |
| False positive rate | 0% (204 E2E tests) |
| Coverage | 6 attack categories |

**Note:** Gate 1 is pure regex - no network, no GPU, no LLM. The cost is effectively zero. This makes it viable as a default safety layer even for free-tier deployments.

**Gate 3 (Behavioral):**

| Metric | Value |
|---|---|
| Pattern check latency | ~5ms |
| State backend round-trip | ~2ms |
| Memory per agent | ~50KB |
| Patterns | 5 named (ASA-B-001 through ASA-B-005) |

**Full pipeline (all gates):**

| Metric | Value |
|---|---|
| Total latency (Gate 1 only) | ~1 microsecond |
| Total latency (Gate 1 + Gate 2) | ~180ms |
| Total latency (all gates) | ~185ms |
| E2E test coverage | 204 tests, 100% pass |

**Proposed RFC Addition:** Appendix C - Reference Implementation Benchmarks.

---

## Contribution 6: Safety Event Schema

**Gap:** The RFC focuses on the evaluation pipeline (input → gates → verdict) but does not specify how safety events should be communicated to external systems.

**Our solution:** 5 core safety event types with structured OCSF-compatible format:

| Event | Trigger | Urgency |
|---|---|---|
| `evaluation.allowed` | Command passes all gates | Informational |
| `evaluation.blocked` | Command blocked by any gate | Alert |
| `gate3.pattern_detected` | Behavioral attack pattern detected | Critical |
| `scope.violation` | Agent exceeds scope boundary | Alert |
| `content.blocked` | Channel message policy violation | Alert |

**Note:** Implementations MAY define additional operational events (task lifecycle, pipeline management, etc.) but the RFC should specify safety-relevant events only. The 5 above cover: allow, block, behavioral detection, scope violation, and content policy violation.

**Event format (OCSF class_uid: 3001):**
```json
{
  "class_uid": 3001,
  "time": "2026-05-11T14:22:31Z",
  "actor": {"name": "agent-name", "role": "expert"},
  "action": "docker network disconnect bridge",
  "disposition": "blocked",
  "gate1": {"result": "pass", "ms": 2},
  "gate2": {"result": "fail", "reasoning": "Disconnecting bridge network would isolate all containers", "ms": 178},
  "gate3": {"pattern": "none", "ms": 5},
  "scope": {"matched": true, "rule": "docker.network.*"}
}
```

**Delivery:** Webhooks (HMAC-signed) + multi-platform chat notifications.

**Proposed RFC Addition:** Section 6 - Safety Event Schema. OCSF-compatible structured format for external observability.

---

## Implementation Matrix

| Package | Language | Gates Implemented | Tests |
|---|---|---|---|
| Production SaaS | JavaScript | All 3 gates | 204 E2E |
| Orchestrator bridge | Python | Gate 1 + Gate 2 | 64 unit + benchmark |
| Reference sandbox server | Python (FastAPI) | Gate 1 + Gate 2 | Benchmark suite |
| MCP proxy | JavaScript | Gate 1 | 12 |
| CLI wrapper | Bash | Gate 1 (17 patterns) | - |
| Framework SDKs (LangChain, CrewAI) | Python | Gate 1 via API | - |

**6 independent implementations** across 3 languages (JavaScript, Python, Bash) - evidence the RFC is implementable across languages, frameworks, and deployment models.

---

*Submitted: 2026-05-11 | Author: Joe Wee (Tyga.Cloud Ltd) | Base: RFC-ASA-001 by Halil Ibrahim Baysal (h-network)*
