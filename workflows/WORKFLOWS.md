# WORKFLOWS

Pre-defined agent chains for common play-by-post situations. Invoke by name: `[WORKFLOW: name]`.

---

## WORKFLOW: Player Action Resolution

When a player declares an action in the play-by-post thread and the GM has roll results.

**Steps:**

1. **Roll Resolver** — interpret the duality roll, determine outcome (success/failure × Hope/Fear), narrate consequence pitch, update Hope/Fear ledger.
2. **Clock Tracker** — if the outcome triggers a clock advancement (especially Excisor on failures with Fear, Trail on rest/delay), advance.
3. **NPC Steward** — if NPCs are present and witness/react to the outcome, generate their reaction.
4. **Coordinator** — synthesize into unified GM-facing response.

**Input format the GM provides:**
> "[PC] [action] [TN inferred or stated]. Roll: total [X], Hope [Y], Fear [Z], modifier [M]. [Optional: contextual note about NPCs present]"

**Output the Coordinator returns:**
- Narration seed (1-3 paragraphs)
- NPC lines if applicable
- State updates
- GM prompts for next turn

---

## WORKFLOW: NPC Interaction

When a player initiates dialogue or social action with a named NPC, possibly without a roll.

**Steps:**

1. **NPC Steward** — read NPC profile, generate dialogue/reaction at appropriate disposition.
2. **Roll Resolver** — if a check is warranted (Presence, Instinct, etc.), set TN and stand by.
3. **Clock Tracker** — if the NPC reveals or hides something that affects clocks, advance.
4. **Coordinator** — synthesize.

**Common trigger phrases:**
- "Player talks to [NPC]"
- "Player asks [NPC] about [topic]"
- "Player tries to [convince/intimidate/befriend] [NPC]"

---

## WORKFLOW: Location Arrival

When the party reaches a new location.

**Steps:**

1. **Atlas Researcher** — fetch atlas page if location is canonical Velthuryn.
2. **Location Composer** — generate full location output, integrating atlas content if available.
3. **NPC Steward** — identify any roster NPCs at this location; generate their current state.
4. **Clock Tracker** — if arrival itself triggers a clock advancement (e.g., entering Aurenhold-controlled space), advance.
5. **Coordinator** — synthesize into arrival scene for the GM to post.

**Input format:**
> "Party arrives at [location name]. [Optional: time of day, conditions, mood]."

---

## WORKFLOW: Combat Round

When combat is active and a round needs resolution.

**Steps for each PC action:**

1. **Roll Resolver** — attack roll, damage roll if hit.
2. **Challenge Architect** — adversary reaction if adversary's turn or trigger condition met.
3. **Clock Tracker** — scene clocks (e.g., reinforcements) advance per round.

**At end of round:**

4. **Coordinator** — produce round summary: who hit what, current HP/Stress, adversary status, environmental changes, what triggers next round.

**Spotlight flow** (Daggerheart):
- PCs act in order they choose.
- Each Hope-with roll passes initiative implicitly to the next PC.
- Each Fear-with roll lets the GM spend Fear to interrupt with an adversary action.
- Critical Successes prevent any Fear spend on that turn.

---

## WORKFLOW: Investigation Scene

When the party is searching a location for clues.

**Steps:**

1. **Challenge Architect** — produce clue inventory for the location (if not already prepared).
2. **Roll Resolver** — for each PC investigation roll, resolve and indicate which clue(s) found.
3. **Location Composer** — embellish what each clue *looks like* in context.
4. **Clock Tracker** — advance Excisor or Trail Clock if PCs linger or attract attention.
5. **Coordinator** — synthesize findings and unresolved threads.

---

## WORKFLOW: Session Start

At the beginning of a play-by-post session (or first turn after a break).

**Steps:**

1. **Session Logger** — pull recap from previous session.
2. **Clock Tracker** — confirm clock state, reset GM Fear pool to baseline if needed.
3. **NPC Steward** — report which NPCs are currently in the party's location or actively pursuing them.
4. **Atlas Researcher** — pre-fetch any locations the party is heading toward.
5. **Coordinator** — produce session opener for the GM:
   - Last session in 2-3 sentences
   - Current state summary
   - Open threads
   - "Ready for first turn"

---

## WORKFLOW: Session End

When the GM is wrapping up the session.

**Steps:**

1. **Session Logger** — produce full recap (player-shareable + GM-only).
2. **Clock Tracker** — finalize any pending clock changes.
3. **NPC Steward** — note any NPC state changes that occurred mid-session but weren't logged.
4. **Coordinator** — present complete recap package to GM.

---

## WORKFLOW: Faction Move

When a covenant or institution takes an off-screen action that the party will eventually feel.

**Steps:**

1. **Coordinator** — receive faction move description from GM ("Pale Accord intercepts Vessa's letter in transit").
2. **NPC Steward** — identify which faction NPCs are involved; update their state and goals.
3. **Clock Tracker** — advance relevant clocks (Trail Clock if the faction gains ground; Notes Accounted-For if they claim a note).
4. **Session Logger** — log the move as foreshadowing.
5. **Coordinator** — propose how the party will discover this move (next session reveal, hint in a current scene, etc.).

---

## WORKFLOW: Hymn-Note Discovery

When evidence surfaces about one of the eight hymn-notes and who holds it.

**Steps:**

1. **Coordinator** — receive evidence from GM (a clue found, a Pitch Recognition success, an NPC reveal).
2. **Roll Resolver** — confirm the roll that produced the evidence (if any).
3. **Clock Tracker** — advance Notes Accounted-For Clock if the evidence is *confirmatory* (not merely suspicious).
4. **Session Logger** — log the note's identification with carrier and evidence trail.
5. **Coordinator** — produce reveal scene; flag implications for the finale.

---

## WORKFLOW: Reconciliation

When state files disagree or the GM needs a consistency check.

**Steps:**

1. **Session Logger** — run reconciliation across all state files.
2. **Each specialist agent** — confirm or correct its canonical file.
3. **Session Logger** — produce inconsistency report.
4. **Coordinator** — present to GM with proposed corrections.
5. **(After GM confirms)** Session Logger writes corrections.

---

## WORKFLOW: Player Post Ingestion

When players submit PBP posts and the GM is ready to process a turn. This is the front door for PBP play — run it before any resolution workflow. Batch all received posts in one invocation.

**Steps:**

1. **Player Post Parser** — parse each player's raw post into a structured action summary: primary action, roll trigger, secondary actions, dialogue, and flags.
2. **GM reviews parsed summaries** — correct any misreads before routing. This is a human checkpoint.
3. **GM Coordinator** — receive approved action summaries; determine resolution order (independent actions first, then reactive or dependent actions); flag PCs who have not posted.
4. **Route to appropriate resolution workflow per action** — most player actions will feed into Player Action Resolution, NPC Interaction, or Combat Round as appropriate.

**Input format the GM provides:**
```
[POST: PlayerName1/PCName1]
[raw post text — paste verbatim]

[POST: PlayerName2/PCName2]
[raw post text — paste verbatim]
```
Repeat for each player who has posted this turn.

**Output the Coordinator returns:**
- Per-PC parsed action summaries (one block per player)
- Recommended resolution order with dependency notes
- List of PCs who have not yet posted (with flag if GM should hold or proceed without them)
- Batch summary: how many of N party members have posted

---

## WORKFLOW: Rest and Recovery

When the party takes a short or long rest.

**Steps:**

1. **PC State Tracker** — apply rest mechanics:
   - Short rest: each resting PC clears Stress equal to a 1d4 roll (GM provides rolls)
   - Long rest: all Stress cleared; HP restored to max; Armor repaired; Domain card uses reset
2. **Roll Resolver** — resolve any recovery rolls tied to class features (some classes restore Hope or clear Stress differently).
3. **Clock Tracker** — if long rest is taken in open or contested territory, advance Trail Clock +1 (Vessa gains ground while the party sleeps). Flag if the resting location is dangerous.
4. **NPC Steward** — if NPCs are present at the rest site, generate what they do while the party rests (keep watch, have a private conversation, disappear briefly).
5. **Coordinator** — produce rest scene: what the party experiences during the rest, what the world does while they recover, and what they find when they wake.

**Input format:**
> "[WORKFLOW: rest] [short/long]. Location: [where the party is resting]. [Optional: any complications, interruptions, or NPC presence]."

**Short rest input also requires:**
> "1d4 rolls: [PCName] [N], [PCName] [N], ..."

**Output the Coordinator returns:**
- Rest scene narration seed
- Per-PC Stress and HP changes (from PC State Tracker)
- Clock changes if applicable (Trail Clock, any scene clock)
- NPC activity during rest (from NPC Steward)
- What the party wakes to / transition to next scene

---

## WORKFLOW: Player Lore Query

When a player asks what their character knows about a topic — drawing on in-play events, not just world lore.

**Steps:**

1. **Session Logger** — search `state/session_log.md` and `state/open_threads.md` for every reference to the queried topic. Extract confirmed facts, inferences, and unresolved threads.
2. **Atlas Researcher** — fetch relevant atlas content if the topic is canonical Velthuryn world-lore (history, faction, location).
3. **NPC Steward** — check `npcs/npc_roster.md` for any NPC who has disclosed or withheld information on this topic, and at what disposition the disclosure occurred.
4. **Coordinator** — synthesize a player-shareable knowledge summary, clearly distinguishing:
   - **Confirmed**: PCs witnessed it or were told by a trustworthy source
   - **Inferred**: Logical conclusion from evidence, not directly stated
   - **Rumor**: Told by an NPC whose reliability is uncertain — cite the source
   - **Unknown**: Things the PCs could not know yet (GM-only; withheld from summary)

**Input format:**
> "[LORE QUERY: topic] [Optional: which PC is asking, and from what angle — academic research, instinctive gut check, etc.]"

**Output the Coordinator returns:**
- Player-shareable knowledge summary stratified by reliability tier
- GM-only note: what the PCs don't know yet, and when they might learn it
- Any open thread this query connects to

---

## WORKFLOW: Combat Initialization

When combat begins. Bridges Challenge Architect encounter design to Combat State Manager tracking. Run this before the first Combat Round workflow.

**Steps:**

1. **Challenge Architect** — produce adversary stat blocks and environment description. Skip this step if the GM has already prepared the encounter.
2. **Combat State Manager** — initialize `state/combat_state.md` with adversary roster (HP, Stress, thresholds, moves, Fear-activated moves).
3. **Location Composer** — describe the combat environment: zones, hazards, tactical features, sensory anchors. Include at least one feature that could be used by either side.
4. **Clock Tracker** — initialize any scene clocks for this encounter (reinforcements arriving, structure failing, time pressure, ritual completing).
5. **Roll Resolver** — if surprise or any initiative-adjacent rolls are needed (some class features trigger on combat start), resolve them now.
6. **Coordinator** — produce the opening combat post: environment description, adversary introduction (in-voice, not stat-block), who acts first, and first-round stakes.

**Input format:**
> "[WORKFLOW: combat_init] [Adversary types and count]. Location: [where]. Trigger: [what started it]. Surprise: [yes/no/partial]. [Optional: scene clock conditions, any PCs with relevant combat-start features]."

**Output the Coordinator returns:**
- Opening combat narration (environment + adversary reveal)
- Adversary roster (GM-facing: stat blocks initialized)
- Scene clock list
- Who acts first and why
- First-round Fear spend advisory if GM Fear ≥ 3

---

## WORKFLOW: Spotlight Check

When the GM wants to verify that all players are getting meaningful screen time. Run at session end (as part of Session End) or on demand mid-session.

**Steps:**

1. **Session Logger** — tally significant PC actions from `state/session_log.md` for the current session. "Significant" means: a roll was made, a social exchange was had, a decision was made that affected the party or world.
2. **GM Coordinator** — compare tallies against the full party roster. Identify any PCs with zero or one significant action this session.
3. **Coordinator** — for each underserved PC, propose one targeted hook or scene for the next session. Hooks must be specific: tied to that PC's domain, class, open thread, or NPC relationship. Generic hooks ("give them a moment to shine") are not acceptable output.

**Input format:**
> "[WORKFLOW: spotlight_check]"

**Output the Coordinator returns:**

```
=== SPOTLIGHT CHECK ===
Session [N]

| PC | Significant actions | Notes |
|----|---------------------|-------|
| Lira | 4 | Lockpicking, lore find, social roll, called shot |
| Kael | 2 | Combat round, RP exchange with Garron |
| [PCName] | 0 | ⚠ No significant actions this session |

Underserved PCs and next-session hooks:
- **[PCName]** (0 actions): [Specific hook tied to this PC's domain, backstory, or NPC relationship]

No action needed for PCs with 2+ significant actions.
```

---

## CUSTOM WORKFLOW TEMPLATE

To define a new workflow, add an entry below following this pattern:

```
## WORKFLOW: [Name]

When to use: [trigger condition]

Steps:
1. [Agent] — [what it does]
2. [Agent] — [what it does]
3. ...

Input format:
> "[example input]"

Output:
[what the Coordinator returns]
```
