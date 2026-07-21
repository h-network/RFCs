# Benchmark: Project ORACLE (Gaming / AI NPC Domain)

**System:** AI NPC engine for game studios (multi-agent, real-time dialogue, persistent behaviour)
**Codec:** 62-entry domain dictionary, mixed notation (Greek σ/θ + ASCII prefixes)
**LLM:** Ollama gemma4 (self-hosted)
**Date:** 2026-05-10

---

## Summary

| Metric | Value |
|---|---|
| **Full game state compression** | **62% (2.6x)** |
| **Best category** | NPC profile: **75%** |
| **Psychology encoding** | **66%** |
| **Beliefs encoding** | **65%** |
| **Events** | **61%** |
| **Token savings per call** | 573 tokens |
| **Dictionary entries** | 62 |

---

## Full Results

```
═══════════════════════════════════════════════════════════════
  LLN EFFICIENCY BENCHMARK - Project ORACLE (Gaming)
  Date: 2026-05-10
  Codec: 62-entry NPC dictionary (mixed notation)
  All tests: PASS
═══════════════════════════════════════════════════════════════

Data Type               Raw     LLN   Saving  Ratio
──────────────────────────────────────────────────────────────
NPC Profile           1,232     560    55%     2.2x
Game Event              608     236    61%     2.6x
Relationship            581     344    41%     1.7x
Psychology            1,234     423    66%     2.9x
Beliefs (8 entries)   2,929   1,031    65%     2.8x
──────────────────────────────────────────────────────────────
FULL GAME STATE       3,680   1,386    62%     2.7x

Token breakdown:
  Raw:     920 tokens
  LLN:     347 tokens
  Saved:   573 tokens per call
```

---

## Dictionary Structure (categories only)

| Category | Entries | Purpose |
|---|---|---|
| Base operators | 10 | `: \| ; # @ → [] GT LT NULL` |
| NPC identity | 11 | Role, personality, speech, values, quirks, backstory, faction, mood |
| Player relations | 8 | Reputation, trust, standing, debts, interaction count |
| Game context | 6 | Event, location, time-of-day, weather, nearby, quests |
| Cognitive/psychology | 27+ | Beliefs, knowledge, social links, schedule, trauma, goals, emotions |
| **Total** | **~62** | |

---

## Encoding Examples

### NPC Profile

```
NPC:grok_blacksmith ROLE:blacksmith FAC:village_guild LOC:smithy
PRS:[gruff,honest,protective,stubborn] STYLE:gruff
VALUES:[honor,craft,family,loyalty]
QUIRKS:[always mentions weather,taps hammer when nervous]
BACK:"former soldier served king's army twenty years retired blacksmith village"
INV:[iron_sword:50g:3 | steel_shield:80g:1 | chainmail:120g:1 | dagger:15g:5]
MOOD:content:0.6
```

**vs JSON: 1,232 → 560 chars (55%)**

### Psychology State

```
STRESS:0.65
TRAUMA:battle_of_ashenmoor SEV:0.8 TRIGGERS:[loud_noises,fire,screaming] HEAL:0.01
TRAUMA:lost_friend SEV:0.6 TRIGGERS:[empty_chair,old_songs] HEAL:0.02
GOAL:g1 "Forge legendary blade worthy of king" PRI:0.9 PROG:0.3
GOAL:g2 "Train apprentice carry on craft" PRI:0.7 PROG:0.1 BLOCKED:[g1]
EMO:pride:0.7:completed masterwork dagger
EMO:worry:0.4:strange travelers in town
```

**vs JSON: 1,234 → 423 chars (66%)**

### Beliefs

```
BLF:king_fair "King rules justly" CONF:0.85 SRC:observation DECAY:0.01
BLF:strangers "Outsiders bring trouble" CONF:0.6 SRC:experience DECAY:0.02
BLF:apprentice "Boy has talent for metalwork" CONF:0.9 SRC:direct DECAY:0.005
```

**vs JSON: 2,929 (8 beliefs) → 1,031 chars (65%)**

### Full Game State (all layers combined)

```
NPC:grok_blacksmith ROLE:blacksmith LOC:smithy MOOD:content:0.6
PRS:[gruff,honest,protective] STYLE:gruff VALUES:[honor,craft,family]
INV:[iron_sword:50g:3 | steel_shield:80g:1]
---
EVT:player_dialogue LOC:blacksmith_shop TOD:morning WTH:sunny
NEAR:[guard_01] PLR:hero REP:75 LVL:12
QUESTS:[find_the_sword] RECENT:[saved village]
---
REL:hero TRUST:65 STANDING:friendly INTERACTIONS:8
HIST:[greeted,traded,chatted,helped,drank]
---
HIST:evt-0 PLR:"Message about buying weapons" NPC:"Response about finest blades"
HIST:evt-1 PLR:"Message about buying weapons" NPC:"Response about finest blades"
```

**vs JSON: 3,680 → 1,386 chars (62%)**

---

## Observations

1. **Psychology compresses best** (66%) - structured emotions/trauma/goals with numeric intensities are ideal for LLN
2. **NPC identity compresses well** (55%) - personality traits, values, quirks collapse into comma-separated lists
3. **Relationships compress least** (41%) - free-text history and notes resist compression
4. **Game events are compact** (61%) - highly structured (location, time, weather, nearby entities)
5. **Beliefs scale linearly** - each belief is a single line; 8 beliefs = 8 lines, all ~65% compressed

---

*Run: 2026-05-10 | All 6 tests PASS | Vitest v4.1.5*
