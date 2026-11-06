  <img src="Subterranean.png">

# Subterranean
Cave exploration scenario script with mechanical to narrative navigation bridge

# AI Dungeon Cave Survival Engine

> **The cave is procedurally generated. The danger is real. The storyteller doesn't get to change the map.**
>
> An experimental, behind-the-scenes JavaScript engine for an **AI Dungeon** survival-horror scenario inspired by *Sub Terra*, the cooperative cave-exploration board game by Inside the Box.

![Version](https://img.shields.io/badge/version-v0.5.1-blue) ![Status](https://img.shields.io/badge/status-playtest%20prototype-orange) ![Platform](https://img.shields.io/badge/platform-AI%20Dungeon-purple) ![Language](https://img.shields.io/badge/language-JavaScript-yellow)

## The premise

A corporate expedition descends into an uncharted cave system. A collapse seals the original route to the surface. The ascent rope was **deliberately cut**. The team must map an unstable labyrinth, survive its hazards, and discover what the corporation knew before sending them underground.

This is **not** a board-game UI pasted into a text adventure. The player interacts in ordinary language while a hidden rules engine handles exploration, action points, hazards, injuries, and the cave's persistent geography.

## What works today

| System | Current implementation |
|---|---|
| **Procedural cave map** | Persistent 2D grid with coordinates, tile exits, rotation, and legal connections |
| **Exploration** | Move into unknown passages or reveal adjacent tiles without entering |
| **Action points** | 2 AP per round; the round advances automatically when AP reaches 0 |
| **Cave tile deck** | 64 type-distributed cave tiles, plus an Exit tile in the final six draws; exact tile shapes are still provisional |
| **Hazard deck** | 30-card prototype deck covering Tremor, Flood, Gas, Cave-In, and Horror |
| **Environmental state** | Persistent rope, collapse, flood, Horror, and gas flags on individual tiles |
| **NPC simulation** | Six NPC specialists with basic autonomous follow behavior; Medic can heal |
| **Natural-language actions** | Direct actions detected in Input, with AI-assisted interpretation for uncertain intent |
| **Story synchronization** | Structured player speech, verified events, current-state snapshots, and Output safeguards |
| **Debug logging** | `ST | ...` logs across Library, Input, Context, and Output |

### Example: say it naturally

```text
"Let's go north."                  → Attempt MOVE N
"East it is then."                 → Attempt MOVE E
"I look around."                   → Free N/E/S/W report
"Let's check the eastern passage." → Reveal E without entering
"Maybe we should go west?"         → Discussion, no automatic movement
"Sarah, what do you think?"        → NPC dialogue, no AP cost
```

The script distinguishes **what the player said** from **what the player mechanically attempted**. NPCs can respond to the spoken words without the storyteller inventing movement or spending AP.

## Commands

Natural language is the intended interface. Explicit commands provide deterministic control when phrasing is ambiguous.

| Command | Cost | Effect |
|---|---|---|
| `#look` | Free | Inspect the current room; output N/E/S/W information |
| `#look N/E/S/W` | 1 AP if unknown | Reveal a connected tile in that direction **without moving**; known tiles can be inspected for free |
| `#move N/E/S/W` | Usually 1 AP | Enter a connected tile; exploration can reveal a tile on entry. Some terrain costs 2 AP |
| `#reveal N/E/S/W` | 1 AP | Explicitly draw and place an unexplored adjacent tile |
| `#rest` | 2 AP | Heal 1 HP if injured |
| `#hide` | 2 AP | Attempt concealment with a D6 check |
| `#rope` | 2 AP | Attempt to secure a rope on an eligible ledge or slide |
| `#dig [N/E/S/W]` | 2 AP | Clear reachable collapsed rubble |
| `#exert` | 0 AP | Gain +1 AP once per round; risk 1 HP on the round-end check. Declare **before** the last AP is spent |
| `#endturn` | — | End the round early and resolve NPCs and a hazard |
| `#status` | Free | Inspect HP, AP, location, exits, and deck counts |
| `#help` | Free | Display the in-game command reference |
| `#escape` | 1 AP | Attempt escape while on the actual Exit tile |
| `#skill`, `#fight` | — | Reserved; specialist abilities and custom combat are **not yet implemented** |

## How the engine works

```text
PLAYER INPUT (Do / Say / explicit #command)
             |
             v
INPUT MODIFIER — normalize speech and classify intent
             |
             +-- Clear action --> execute JavaScript mechanics
             +-- Possible action --> request AI intent assessment
             +-- Dialogue only --> preserve the player's words
             |
             v
STATE + EVENT LEDGER — record verified changes
             |
             v
CONTEXT MODIFIER — send current map facts, player speech,
                   verified events, and narration constraints
             |
             v
AI STORYTELLER — provide atmosphere, dialogue, and reactions
             |
             v
OUTPUT MODIFIER — parse deferred intent, enforce key
                  map reports, and provide factual event fallbacks
```

**The governing rule:** JavaScript decides *what happened*. The AI decides *how it feels and sounds*. It must not invent unexplored rooms, unauthorized movement, mechanical injuries, or hazards that contradict the map.

The game state lives under `state.subterra`. Cave tiles are indexed on a coordinate-based 2D map; permanent hazard-location arrays identify eligible tiles, while each tile's own flags track its current condition. Game rounds are separate from AI Dungeon message counts, so conversation can remain free-form.

## Installing in AI Dungeon

1. Create or open an AI Dungeon scenario with scripting enabled.
2. Paste the complete `Library.txt`, `Input.txt`, `Context.txt`, and `Output.txt` contents into their matching script tabs.
3. Use `AI_Instructions_Replacement.txt` as the scenario's AI Instructions. Configure the scenario Opening, Plot Essentials, and Author's Note separately.
4. Start a **fresh adventure** for the cleanest test, then try `#status`, `#look`, `#look N`, and `#move N`.
5. Check script logs for `ST | INPUT`, `ST | INTENT`, `ST | AP`, `ST | ROUND-END`, `ST | NPC-PHASE`, and `ST | HAZARD` entries.

**v0.5.1 packaging note:** v0.5.1 is a hotfix to v0.5. If updating manually, use the **v0.5 Library and Context** and the **v0.5.1 Input and Output** scripts.

## Prototype limitations and roadmap

The engine is actively being tested in live AI Dungeon adventures. Important work remains:

- **NPC expedition groups:** Prevent solo scouting, keep a companion with the player, and limit voluntary separation to approximately six connected tiles.
- **Board-game fidelity:** Replace sample tile exit patterns and the provisional equal-split hazard deck with exact reference data and finalized placement rules.
- **Specialist abilities and combat:** Implement character-role skills, richer NPC behavior, and the planned `#fight` mechanics.
- **Hazard-deck endgame:** Define what exhaustion means. The old 500-response escape gate and 1000-response battery failure have been removed.
- **Narration reliability:** Continue improving Say-mode parsing, directional intent detection, event delivery, and consistency between AI prose and script state.
- **Resources and story integration:** Expand equipment, light management, expedition injuries, and the corporate mystery as the simulation matures.

This is a **playtest prototype**, not a finished or fully rules-accurate implementation. AI classification and narration are probabilistic; explicit commands remain the reliable fallback.

## Testing and feedback

When reporting a bug, the most useful evidence is:

1. The **exact player input**, including whether it used Do, Say, or `#command`.
2. The **story excerpt** showing unexpected behavior.
3. The relevant **Input, Context, and Output logs** (`ST | ...`).

Please especially report mismatches such as narrated movement without an AP cost, duplicate direction reports, omitted NPC/hazard events, or an AI-created room that doesn't exist in the JavaScript map.

## Credits and disclaimer

Inspired by **Sub Terra**, designed by **Tim Pinder** and originally published by **Inside the Box Board Games**. This project is an **unofficial fan-made adaptation and scripting experiment**, not affiliated with or endorsed by the original game's creators or publisher. The corporate-expedition story and AI Dungeon implementation are separate creative additions.
