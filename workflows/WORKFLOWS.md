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
