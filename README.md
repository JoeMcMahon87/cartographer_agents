# THE CARTOGRAPHER WHO SANG BACKWARDS — OpenClaw Agent Bundle

A model-agnostic, multi-agent GM-assistant bundle for running *The Cartographer Who Sang Backwards* (Daggerheart campaign, Velthuryn setting) in play-by-post with 7–8 players. Includes 11 specialist agents, 15 named workflows, and a full state management system for HP, clocks, Hope/Fear, NPC dispositions, and active combat.

This bundle is designed for **OpenClaw** or any agent-orchestration framework that can route prompts to specialized agents, read/write state files, and chain agents into workflows.

---

## QUICK-START

1. **Drop the entire `cartographer_agents/` directory** into your OpenClaw workspace (or whatever model runtime you're using).
2. **Open `agents/00_gm_coordinator.md`** — this is the entry-point agent. It routes everything else.
3. **Edit `state/party_roster_template.csv`** to add your 7–8 players, then save it as `state/party_roster.csv`. Also populate `state/pc_state.md` with your PCs' starting HP, Stress, and Armor values.
4. **Run a session-start workflow** (see `workflows/WORKFLOWS.md` → "Session Start"). The coordinator will load state, summarize threads, and prompt you for the opening beat.
5. **For each player turn**, use `[WORKFLOW: Player Post Ingestion]` — paste in all received player posts and let the Player Post Parser extract structured action summaries before routing. For each resolved action, send the GM Coordinator the parsed summary with roll results.
6. **At session end**, run `[WORKFLOW: Session End]` then `[WORKFLOW: Spotlight Check]` to verify all 7–8 players got meaningful screen time.

---

## DIRECTORY LAYOUT

```
cartographer_agents/
├── README.md                         (this file)
├── AGENTS.md                         Dev context for AI coding sessions (Loam/Archon)
├── agents/
│   ├── 00_gm_coordinator.md          Orchestrator — entry point
│   ├── 01_clock_tracker.md           Manages 3 campaign clocks + scene clocks
│   ├── 02_npc_steward.md             Manages NPC roster, generates in-voice dialogue
│   ├── 03_location_composer.md       Generates locations with tone/sensory anchors
│   ├── 04_roll_resolver.md           Daggerheart Duality Die mechanics
│   ├── 05_challenge_architect.md     Designs combat/social/investigation/environment
│   ├── 06_session_logger.md          End-session recaps, state reconciliation, spotlight
│   ├── 07_atlas_researcher.md        Fetches & navigates atlas.mcmahongroup.org
│   ├── 08_player_post_parser.md      Parses raw PBP player posts into structured actions
│   ├── 09_pc_state_tracker.md        Tracks HP/Stress/Conditions; handles rest/recovery
│   └── 10_combat_state_manager.md    Tracks adversary state across multi-round PBP combat
├── workflows/
│   └── WORKFLOWS.md                  Multi-agent flows (Action, Combat Round, PBP-specific, etc.)
├── state/
│   ├── campaign_state.md             Master state — trail position, world status
│   ├── clocks.md                     Live clock values
│   ├── hope_fear_ledger.md           Per-PC Hope + GM Fear pool
│   ├── pc_state.md                   Per-PC HP, Stress, Armor, Conditions (mutable)
│   ├── combat_state.md               Active encounter adversary tracking (ephemeral)
│   ├── party_roster_template.csv     Replace with your party's CSV
│   ├── session_log.md                Per-session entries (logger appends)
│   ├── open_threads.md               Active plot threads
│   └── atlas_cache.md                Cached atlas.mcmahongroup.org content
├── reference/
│   ├── daggerheart_quickref.md       Duality Die, TN bands, all core mechanics
│   ├── campaign_quickref.md          Distinctions, clocks, factions, locations
│   └── atlas_index.md                Known/guessed worldbook URLs
├── plans/                            Archon PIV implementation plans
└── npcs/
    └── npc_roster.md                 15 starter NPCs + template
```

---

## INVOCATION PATTERNS

### Pattern A — Through the Coordinator (default)

You talk only to the GM Coordinator. It reads state, decides which specialists to call, and returns a unified response. Best for typical play.

```
[Player action: "I want to convince Vaen Ril to let me into the Codex Bastion archives."]
[Player roll: Hope 11, Fear 7 — total 18, Presence +2 = 20]
```

The Coordinator will: load state → ask NPC Steward for Vaen Ril's current disposition and voice → ask Roll Resolver to interpret 20 vs. Hard TN → narrate consequence → update state → return everything in one structured response.

### Pattern B — Direct specialist call

Use a `@AgentName` prefix or whatever your framework uses to bypass the coordinator and call one specialist:

```
@NPC_Steward: Generate three lines for Sister Thalia Brightvein responding to the party offering her a hymn-note fragment.
```

```
@Clock_Tracker: Advance Excisor Clock by 1. Party named the codex aloud in a Boomcradle tavern.
```

### Pattern C — Workflow

Trigger a multi-step chain. See `workflows/WORKFLOWS.md`.

```
[WORKFLOW: combat_round]
[Encounter: 2 Inkward Custodians + 1 Excisor Tribunalist at the north gate of Aurenhold]
```

---

## DAGGERHEART RULES ENFORCEMENT

The agents enforce Daggerheart mechanics to the best of their ability:

- **Duality Die outcomes** (Hope > Fear, Fear > Hope, doubles = Critical) — applied automatically by Roll Resolver.
- **GM Fear pool** — incremented on any Fear-with-Fear roll; max 12; spent to activate adversary moves or change scene state.
- **Per-PC Hope** — incremented on Hope-with-Hope rolls; max 6 per PC; spent on Experiences, helping, etc.
- **TN bands** — Trivial <8, Easy 8–10, Standard 11–14, Hard 15–17, Very Hard 18–20, Extreme 21+.
- **Damage thresholds** — Each PC has Major and Severe thresholds; damage at or above shifts HP loss tier.
- **Stress** — Tracked per PC; full Stress = unconscious or out of scene.
- **Critical (doubles)** — Auto-success + 1 Hope + clear 1 Stress, and the GM cannot spend Fear from this roll.

When the GM (you) disagrees with an agent's adjudication, use `[GM OVERRIDE]` and the agents will accept your call and update state accordingly.

---

## ATLAS INTEGRATION

The Atlas Researcher agent (`agents/07_atlas_researcher.md`) can fetch pages from **https://atlas.mcmahongroup.org** to ground locations, factions, and lore in the canonical worldbook. It caches results in `state/atlas_cache.md` and maintains a URL index in `reference/atlas_index.md`.

Call it when:
- The party reaches a new state/city/region you haven't briefed yet
- An NPC's faction needs canonical detail
- A historical reference comes up (Brightcrawl era, Concord, Mirror Wars, etc.)

The agent caps itself at 4 fetches per query to stay efficient. If it can't find what's needed, it will say so plainly rather than fabricate.

---

## PBP-SPECIFIC WORKFLOWS

Three new agents (08–10) and five new workflows address the specific demands of play-by-post:

| Workflow | When to use |
|----------|-------------|
| **Player Post Ingestion** | At the start of each GM turn — batch all player posts, parse them, determine resolution order |
| **Rest and Recovery** | When the party takes a short or long rest — applies Daggerheart rest mechanics formally |
| **Player Lore Query** | When a player asks what they know about a topic — synthesizes in-play knowledge vs. world lore |
| **Combat Initialization** | When combat begins — bridges encounter design to adversary state tracking |
| **Spotlight Check** | At session end — flags PCs who had no significant actions and proposes next-session hooks |

Use `[PLAYER POST: PCName]` as a quick prefix to route a single player post directly to the parser without running the full batch workflow.

---

## ADDING NEW NPCs MID-CAMPAIGN

Append to `npcs/npc_roster.md` using the template in that file. Set Disposition explicitly. The NPC Steward will pick up the new entry on the next call.

If you want a more elaborate generation: invoke NPC Steward directly with `@NPC_Steward: generate a new NPC at [location] who knows [thing] and wants [goal].`

---

## STATE PERSISTENCE

All state lives in `state/`. The agents read these files at the start of every invocation and write back updates at the end. Between play-by-post turns:

- You don't need to remember what changed — read `state/campaign_state.md` for the current picture.
- Recap any session by reading the relevant entry in `state/session_log.md`.
- Spot active threads in `state/open_threads.md`.

If your framework doesn't auto-persist files, you'll need to copy the updated state back into the bundle before the next turn. The Session Logger agent produces a single "state diff" block you can paste back if needed.

---

## TONE TARGETS

All agents are tuned for the campaign's voice: **scholarly, paranoid, itinerant, lyrical**. NPC dialogue should feel like someone who has read too much and slept too little. Locations should be described by sound and texture before sight. Consequences should land as inevitabilities, not surprises.

When in doubt, the agents err toward atmosphere over crunch.

---

## CREDITS & VERSION

- Campaign frame: *The Cartographer Who Sang Backwards* by Joe McMahon
- Setting: Velthuryn (see https://atlas.mcmahongroup.org)
- System: Daggerheart (Darrington Press)
- Bundle version: v1.0 (Session 0 ready)

---

*Read `agents/00_gm_coordinator.md` next.*
