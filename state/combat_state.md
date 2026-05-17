# COMBAT STATE

Owned by: **Combat State Manager** (agent 10)  
Updated by: Combat State Manager after every adversary action, damage application, and round end.  
Read by: Roll Resolver (for adversary stats), GM Coordinator (for round summaries), Clock Tracker (for scene clocks).

Status: **NO ACTIVE COMBAT**

*This file is initialized when combat begins (see `[COMBAT: init]`) and cleared when combat ends (see `[COMBAT: end]`). Between encounters, it holds only this header.*

---

## Template (populated during an active encounter)

```markdown
# COMBAT STATE

Status: **ACTIVE**

**Encounter**: [name or brief description]
**Location**: [where combat is happening — zone names if applicable]
**Round**: [N]
**GM Fear pool**: [X/12] *(cross-reference hope_fear_ledger.md)*

## Scene Clocks

| Clock | Current | Max | Trigger |
|-------|---------|-----|---------|
| [Name] | [X] | [Y] | [what happens at max] |

## Adversaries

### [Adversary Name] — [TYPE: Grunt / Standard / Elite / Boss]
- **HP**: [current] / [max]
- **Stress**: [current] / [max] (if applicable)
- **Thresholds**: Major [X], Severe [Y]
- **Attack modifier**: +[N]
- **Conditions**: [none | list active conditions with round of expiry if known]
- **Last action**: [what they did most recently, Round N]
- **Available moves**: [list unused moves]
- **Fear-activated moves**: [list with cost — strike through when used]
- **Status**: [Active | Defeated | Fled | Withdrawn]

---

## Round Log

| Round | Event |
|-------|-------|
| 1 | [brief summary of key actions and outcomes] |

---

## Encounter Notes

[GM and Combat State Manager notes — tactics observed, NPC identity if disguised, etc.]
```
