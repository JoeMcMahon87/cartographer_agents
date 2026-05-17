# AGENTS.md — The Cartographer Who Sang Backwards

## Project Overview

This is a **model-agnostic multi-agent GM-assistance bundle** for running a Daggerheart play-by-post (PBP) campaign titled *The Cartographer Who Sang Backwards*, set in the world of Velthuryn. It is a collection of markdown agent definitions, workflow chains, and state files — there is no executable code.

## Structure

```
agents/          — Agent definition files (00-07, numbered by invocation priority)
workflows/       — WORKFLOWS.md: all workflow chains in one file
state/           — Mutable campaign state (written by agents during play)
npcs/            — NPC roster (written by NPC Steward)
reference/       — Read-only reference docs (quickrefs, atlas index)
plans/           — Feature implementation plans (Archon PIV)
```

## Agent File Convention

Every agent in `agents/` follows this structure:
```markdown
# AGENT: [Name]

## ROLE
[What this agent is responsible for, what state files it owns]

## [DOMAIN-SPECIFIC SECTIONS]
[Rules, tables, mechanics specific to this agent]

## INPUTS YOU EXPECT
[Trigger phrases and input formats]

## RULES
[Numbered behavioral rules, 6-10 items]

## OUTPUT FORMAT
[Exact output template with section headers]

## EXAMPLES
[1-3 worked examples with input → output]

## EDGE CASES
[Unusual situations and how to handle them]
```

Agent files are numbered 00–NN. New agents increment the highest existing number. The GM Coordinator (00) is always the entry point.

## Workflow Convention

All workflows live in a single file: `workflows/WORKFLOWS.md`. Each workflow entry has:
- A `## WORKFLOW: Name` header
- When-to-use description
- Numbered steps (Agent — what it does)
- Input format block
- Output description

## State File Ownership

| File | Owned by |
|------|----------|
| `state/campaign_state.md` | Session Logger |
| `state/clocks.md` | Clock Tracker |
| `state/hope_fear_ledger.md` | Roll Resolver |
| `state/pc_state.md` | PC State Tracker (new) |
| `state/combat_state.md` | Combat State Manager (new) |
| `state/session_log.md` | Session Logger |
| `state/open_threads.md` | Session Logger |
| `state/atlas_cache.md` | Atlas Researcher |
| `npcs/npc_roster.md` | NPC Steward |

## Tone Constraint

**All agent narration must be**: scholarly, paranoid, itinerant, lyrical.

Reject: generic high-fantasy, random encounters, off-the-shelf monsters, tavern brawls.

## Key Mechanics (Daggerheart)

- **Duality Die**: 2d12 (Hope die + Fear die) + modifier vs. TN
- **Hope > Fear**: PC +1 Hope. **Fear > Hope**: GM +1 Fear. **Doubles**: Critical Success
- **TN bands**: Easy 8-10, Standard 11-14, Hard 15-17, Very Hard 18-20, Extreme 21+
- **Hope max**: 6/PC. **GM Fear max**: 12.
- **Stress**: 5-6/PC. **Rest**: Short (1d4 Stress), Long (all Stress + HP)
- **Damage thresholds**: Major and Severe per PC; damage at 1/2/3/4 HP based on bracket

## Validation for New Agents

When adding a new agent, verify:
1. Role section clearly states which state files it owns (read vs. read/write)
2. Output format uses `===` section headers consistently with existing agents
3. Examples include at minimum: one typical case, one edge case
4. GM Coordinator routing table in `00_gm_coordinator.md` is updated to include the new agent
5. WORKFLOWS.md is updated with any new workflows the agent enables
6. README.md agent list is updated

## In Scope for Changes

- Adding new agent definition files
- Adding workflows to WORKFLOWS.md
- Updating GM Coordinator routing table
- Creating new state files
- Updating README agent list

## Out of Scope

- Modifying the Daggerheart rules reference (`reference/daggerheart_quickref.md`) — this is RAW
- Modifying campaign narrative content (NPC backstories, clock triggers, ending branches) without GM sign-off
- Changing agent file numbering for existing agents
