# Plan: PBP-Missing Agents and Workflows

## Goal

Add three new specialist agents and five new workflows that address gaps specific to play-by-post (PBP) Daggerheart campaigns. The current eight-agent stack handles GM orchestration well but leaves the GM doing unassisted manual work for: parsing player posts, tracking per-PC health state between sessions, and maintaining adversary state across multi-round PBP combat.

## Context

Reviewed by: GM gap analysis, 2026-05-17
Baseline commit: `7a76c9c` (v1.0 initial commit)

## Task List

Tasks are ordered by dependency: new state files first, then agents that own them, then workflows that invoke them, then updates to existing agents that route to them.

---

### Task 1 — Create `state/pc_state.md` (new state file)

**File**: `state/pc_state.md`

Create the mutable per-PC health state file. This is the canonical source for current HP, Stress, Armor marks, and Conditions between rolls. The Roll Resolver will write to it on damage/healing; the PC State Tracker agent will own it.

Template structure:
```markdown
# PC STATE

Last updated: [session/date]

## [PC Name]
- **HP**: [current]/[max]  (Major threshold: X, Severe threshold: Y)
- **Stress**: [marked]/[max]  ([X] boxes marked)
- **Armor**: [marked]/[slots] ([X] slots used)
- **Conditions**: [none | list active conditions]
- **Hope**: [current] (tracked in hope_fear_ledger.md — cross-reference only)
- **Domain card uses remaining**: [list if applicable]
- **Notes**: [anything the PC State Tracker needs to flag]
```

---

### Task 2 — Create `state/combat_state.md` (new state file)

**File**: `state/combat_state.md`

Create the ephemeral combat state file. Initialized at combat start; cleared at combat end. Tracks adversary roster, HP, conditions, and round counter.

Template structure:
```markdown
# COMBAT STATE

**Encounter**: [name/description]
**Location**: [where combat is happening]
**Round**: [N]
**Scene clock**: [name, current/max] (if applicable)
**Status**: [ACTIVE | RESOLVED | FLED]

## Adversaries

### [Adversary Name A]
- **HP**: [current]/[max]
- **Stress**: [current]/[max]
- **Conditions**: [none | list]
- **Last action**: [what they did most recently]
- **Available moves**: [list unused moves]
- **Fear-activated moves remaining**: [list]

### [Adversary Name B]
...

## Round Log
- Round 1: [brief summary]
- Round 2: [brief summary]
```

---

### Task 3 — Create `agents/08_player_post_parser.md`

**File**: `agents/08_player_post_parser.md`

**Role**: Parse raw player-submitted PBP posts into structured action summaries the GM Coordinator can route. This agent is the new front door for PBP turns — it sits between player posts and the Coordinator.

**Capabilities**:
- Identify primary action (what the PC is attempting)
- Identify secondary/passive actions (what else the PC is doing simultaneously)
- Identify dialogue (what the PC says, to whom)
- Detect implicit roll triggers ("try to pick the lock" → Finesse roll needed)
- Detect explicit roll declarations ("I roll Presence, got 14, H8 F6")
- Flag ambiguities ("unclear whether Kael is attempting the lock or just approaching")
- Flag mechanical issues ("player declared success before rolling")
- Flag coordination dependencies ("Tira says she's covering Lira — who acts first?")

**State files**: Reads `state/party_roster_template.csv` (for PC names), `npcs/npc_roster.md` (to recognize NPC names in posts). Does not write state.

**Input format**:
```
[POST: PlayerName/PCName]
[raw player post text]
```

**Output format**:
```
=== POST PARSE: [PCName] ===
**Player**: [PlayerName]
**PC**: [PCName]

**Primary action**: [what the PC is attempting]
**Action type**: [physical / social / knowledge / stealth / other]
**Roll required**: [yes / no / maybe]
  - If yes: [suggested trait + TN band]
  - If maybe: [ambiguity that needs GM decision]

**Secondary actions**: [list, or "none"]
**Dialogue**: [what PC says, or "none"]
**Addressed to**: [NPC name / another PC / the group / ambient]

**Flags**:
- [any ambiguities, mechanical issues, or coordination notes]

**Coordinator routing**:
- [which agents this action touches]
```

**Rules**:
1. Never decide roll outcomes — only identify that a roll is needed.
2. Never decide what a PC does if the post is ambiguous — flag it and ask.
3. If a player references an NPC by name, verify the name exists in the roster.
4. Parse one player post per invocation. Multiple players = multiple invocations.
5. If a post contains no actionable content (pure RP reaction, brief acknowledgment), flag as "Narrative only — no roll required."
6. Preserve quoted player prose verbatim in the Dialogue field; do not paraphrase.

**Examples**: At least one typical lockpick/action post, one social/dialogue-heavy post, one ambiguous post that needs flagging.

---

### Task 4 — Create `agents/09_pc_state_tracker.md`

**File**: `agents/09_pc_state_tracker.md`

**Role**: Maintain per-PC dynamic state (HP, Stress, Armor, Conditions) between rolls and across sessions. Own `state/pc_state.md`. Handle rest and recovery. Handle PC advancement between arcs.

**Capabilities**:
1. **Apply damage/healing** — receive Roll Resolver output, update HP in `state/pc_state.md`
2. **Apply/clear conditions** — add or remove conditions (Vulnerable, Restrained, etc.)
3. **Short rest** — each PC rolls 1d4, clears that many Stress boxes; GM may optionally restore HP up to next threshold bracket
4. **Long rest** — clear all Stress; restore HP to full; reset Domain card uses; log in session log
5. **Status report** — on request, produce full party status summary (all PCs, all fields)
6. **PC advancement** — between arcs, walk through level-up procedure per Daggerheart rules
7. **Death move handling** — when PC reaches 0 HP, produce Death Move choice prompt; apply result

**State files**: Reads `state/party_roster_template.csv` (thresholds, max HP/Stress). Reads/writes `state/pc_state.md`. Writes to `state/hope_fear_ledger.md` (Hope changes on rest).

**Input formats**:
- `[PC STATE: update]` — apply a specific change (damage, heal, condition)
- `[PC STATE: rest short]` — short rest for full party or named PCs
- `[PC STATE: rest long]` — long rest
- `[PC STATE: status]` — produce full party status report
- `[PC STATE: level up PCName]` — guide advancement for one PC

**Output format**:
```
=== PC STATE UPDATE ===
**Trigger**: [what caused this update]

**Changes**:
- [PCName] HP: [old] → [new] ([reason])
- [PCName] Stress: [old] → [new] ([reason])
- [PCName] Condition added/cleared: [condition]

**Flags**:
- [any PC at 0 HP, any condition with time limit, etc.]

=== STATE WRITE ===
pc_state.md updated.
```

**Rules**:
1. HP cannot go below 0 — at 0, flag Death Move immediately.
2. Stress cannot exceed max — any Stress that would overflow is lost (note it).
3. Long rest requires the party to be in a safe location — flag if location is contested.
4. Never silently advance HP beyond max.
5. On short rest: GM provides 1d4 roll results per PC — don't roll, interpret.

---

### Task 5 — Create `agents/10_combat_state_manager.md`

**File**: `agents/10_combat_state_manager.md`

**Role**: Initialize and maintain active combat encounter state. Own `state/combat_state.md`. Track adversary HP, conditions, and action economy across multi-round PBP combat. Advise on GM Fear spend opportunities.

**Capabilities**:
1. **Initialize encounter** — import Challenge Architect stat blocks; create adversary entries in `state/combat_state.md`
2. **Update adversary state** — apply damage, conditions, mark used moves after each round
3. **Round summary** — produce concise round-end report: all adversary HP, conditions, what triggered
4. **Fear spend advisory** — given current scene + adversary moves + GM Fear pool, suggest 2-3 specific spends
5. **End encounter** — produce outcome summary; clear `state/combat_state.md`; trigger Session Logger

**State files**: Reads `state/clocks.md` (scene clocks). Reads `state/hope_fear_ledger.md` (GM Fear pool for spend advisory). Reads/writes `state/combat_state.md`. Reads `npcs/npc_roster.md` (if adversaries are named NPCs).

**Input formats**:
- `[COMBAT: init]` — start encounter with provided stat blocks
- `[COMBAT: update]` — apply results of a round's actions to adversary state
- `[COMBAT: round summary]` — produce end-of-round status report
- `[COMBAT: fear advice]` — request Fear spend suggestions
- `[COMBAT: end]` — encounter concluded; produce outcome; clear state

**Output format**:
```
=== COMBAT STATE: Round [N] ===

**Adversaries**:
| Name | HP | Stress | Conditions | Last Action |
|------|----|--------|------------|-------------|
| [A]  | X/Y | X/Y  | [list]     | [action]    |

**Scene clock**: [name] [X/Y] ([advancing/stable])
**GM Fear pool**: [X/12]

**Fear spend options** (if Fear ≥ 3):
1. [Specific move + cost] — "[why this is interesting now]"
2. [Specific move + cost] — "[why this is interesting now]"

**Next round triggers**:
- [anything that will change if GM doesn't act]

=== STATE WRITE ===
combat_state.md updated.
```

**Rules**:
1. Adversary HP cannot go below 0 — flag as Defeated; remove from active table.
2. Named NPCs who are defeated in combat are not dead unless GM confirms — mark as Defeated/Withdrawn and flag for NPC Steward.
3. Fear spend advisory is a suggestion only — GM decides.
4. At encounter end, `state/combat_state.md` is cleared and archived to session log.
5. If combat spans multiple real-world days, `state/combat_state.md` is the canonical source for adversary state — not GM memory.

---

### Task 6 — Add five workflows to `workflows/WORKFLOWS.md`

Append to the existing file (do not modify existing workflows). Add in this order:

**Workflow A: Player Post Ingestion**

When to use: At the start of each GM turn, after player posts are collected. The GM batches all received player posts for parsing before any resolution begins.

Steps:
1. Player Post Parser — parse each player post, produce structured action summary
2. GM reviews parsed summaries, corrects any misreads
3. GM Coordinator — receive approved action batch; identify resolution order (independent actions first, then reactive/dependent)
4. Route to appropriate resolution workflow per action

Input format: One `[POST: PlayerName/PCName] [raw text]` block per player who has posted. Chain them in a single message.

Output: Per-PC parsed action summaries; recommended resolution order; list of PCs who have not yet posted (flagged for GM)

**Workflow B: Rest and Recovery**

When to use: When the party takes a short or long rest.

Steps:
1. PC State Tracker — apply rest mechanics (short: 1d4 Stress cleared per PC; long: all Stress + HP restored)
2. Roll Resolver — any recovery rolls tied to class features
3. Clock Tracker — advance Trail Clock +1 if long rest taken in open/dangerous territory
4. NPC Steward — if NPCs are present at rest site, generate activity during rest
5. Coordinator — produce rest scene; what the party experiences; what the world does while they recover

Input format: `[WORKFLOW: rest] [short/long]. Location: [where]. [Optional: complications or interruptions]`

**Workflow C: Player Lore Query**

When to use: When a player asks what their character knows about a topic — from in-play events (not just world lore).

Steps:
1. Session Logger — search session_log.md for references to the queried topic
2. Atlas Researcher — fetch relevant atlas content if topic is world lore
3. NPC Steward — check if any NPC has revealed information on this topic and at what disposition
4. Coordinator — synthesize a player-shareable "what your characters know" summary, distinguishing: confirmed facts / inferences / rumors / things PCs could not know

Input format: `[LORE QUERY: topic] [Which PC is asking, and from what angle — optional]`

Output: Player-shareable knowledge summary, clearly stratified by reliability. GM-only information is withheld entirely.

**Workflow D: Combat Initialization**

When to use: When combat begins. Bridges Challenge Architect encounter design to Combat State Manager tracking.

Steps:
1. Challenge Architect — produce adversary stat blocks and environment description (skip if already prepared)
2. Combat State Manager — initialize state/combat_state.md with adversary roster
3. Location Composer — describe the combat environment (zones, hazards, tactical features)
4. Clock Tracker — initialize any scene clocks (reinforcements, structural failure, time pressure)
5. Roll Resolver — if surprise or any initiative-adjacent rolls are needed
6. Coordinator — produce opening combat post: environment, adversary introduction, who acts first

Input format: `[WORKFLOW: combat_init] [Adversary types + count]. Location: [where]. Trigger: [what started it]. Surprise: [yes/no/partial].`

**Workflow E: Spotlight Check**

When to use: At session end (run as part of Session End workflow), or on demand mid-session when the GM suspects player engagement imbalance.

Steps:
1. Session Logger — tally significant PC actions/moments from this session's log
2. Coordinator — compare against full party roster; identify any PCs with 0 or 1 significant actions
3. Coordinator — for each underserved PC, propose one targeted hook or scene for next session (tied to that PC's domain, backstory, or open thread)

Input format: `[WORKFLOW: spotlight_check]`

Output: Per-PC action tally for this session; flagged PCs; one suggested next-session hook per flagged PC

---

### Task 7 — Update `agents/00_gm_coordinator.md`

Three additions to the existing GM Coordinator file:

**Addition A**: Add `[PLAYER POST: PCName]` to the "Explicit prefixes" list in the INPUTS section:
```
- `[PLAYER POST: PCName]` — route raw player post to Player Post Parser before any other processing
```

**Addition B**: Add a row to the ROUTING LOGIC table:
```
| Player post submitted (raw PBP text) | **Player Post Parser** |
| PC HP/Stress/Condition change        | **PC State Tracker**   |
| Combat begins                        | **Combat State Manager** + **Challenge Architect** |
| Lore/knowledge question from player  | **Session Logger** + **Atlas Researcher** |
```

**Addition C**: Add a **FEAR SPEND ADVISORY** subsection to the SYNTHESIS section, triggered when GM Fear ≥ 4:
```
=== FEAR SPEND ADVISORY ===  (included when GM Fear ≥ 4)
[2-3 specific Fear spend suggestions given current scene, adversaries, and dramatic moment.
Each suggestion: [move name] ([cost]) — "[one sentence on why this spend is interesting now]"]
```

---

### Task 8 — Update `agents/06_session_logger.md`

Two additions:

**Addition A**: Add `[PLAYER POST FORMAT]` output mode. When invoked as `[SESSION LOGGER: player post]`, produce the session summary in player-facing format — narrative prose only, no GM-only notes, suitable for pasting directly to the forum thread.

**Addition B**: Add a **Spotlight Ledger** subsection to the session_log.md template:
```markdown
### Spotlight ledger
| PC | Significant actions this session |
|----|----------------------------------|
| [Name] | [count] ([brief list]) |
```
This formalizes the "Player Highlights" section as trackable data that Spotlight Check can query.

---

### Task 9 — Update `README.md`

Add the three new agents to the agent list. Add a "State Files" section noting the two new state files. Add a brief "PBP Workflows" callout mentioning the five new workflows.

---

## Validation Strategy

Since this project is markdown documentation (no executable code), validation is consistency checking:

1. Every new agent references the correct state files (cross-check ownership table in AGENTS.md).
2. Every new agent's output format uses `===` section headers matching existing agents.
3. Each new workflow's step numbering invokes only agents that exist (08, 09, 10 plus existing 00–07).
4. GM Coordinator routing table includes all three new agents.
5. WORKFLOWS.md contains all five new workflow entries in correct format.
6. `state/pc_state.md` and `state/combat_state.md` templates exist.
7. README agent list matches the actual agents/ directory.
8. Tone check: each new agent's narration examples match "scholarly, paranoid, itinerant, lyrical."

## Files to Create or Modify

| Action | File |
|--------|------|
| CREATE | `state/pc_state.md` |
| CREATE | `state/combat_state.md` |
| CREATE | `agents/08_player_post_parser.md` |
| CREATE | `agents/09_pc_state_tracker.md` |
| CREATE | `agents/10_combat_state_manager.md` |
| MODIFY | `workflows/WORKFLOWS.md` (append 5 workflows) |
| MODIFY | `agents/00_gm_coordinator.md` (3 additions) |
| MODIFY | `agents/06_session_logger.md` (2 additions) |
| MODIFY | `README.md` (update agent list + state files) |

## Estimated Scope

9 files touched (5 created, 4 modified). All markdown. No code.
