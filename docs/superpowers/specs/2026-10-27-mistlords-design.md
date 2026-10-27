# Mistlords — Design Spec (Vertical Slice)

*Working title. Date: 2026-10-27.*

A 2D turn-based fantasy strategy game in the spirit of Heroes of Might and Magic, set in Slavic folklore. **The twist: the map does not exist yet.** Players build the world by playing terrain and site cards from a deck onto the Mist.

---

## 1. Goals

- **Prove the twist is fun.** "Building the map from a hand of cards" should create real decisions: greed vs. safety, helping yourself vs. accidentally helping the rival.
- **Keep the HoMM feel:** hero exploration, resources, towns, weekly recruitment, tactical hex battles.
- **Architect for the future:** deterministic, command-based engine so save/load, replays, AI and later **online multiplayer** are cheap.

### Non-goals for the slice
Online multiplayer, campaign/story, more than 2 factions, multiple heroes per player, artifacts/equipment, map editor, mobile.

---

## 2. Game Design

### 2.1 Premise
The world is unfinished. Beyond known land lies the **Mist**, the formless realm of Nav. Lords shape the land by playing cards, turning Mist into forests, steppe, marshes, mines and monster lairs. Everyone shares one map, so whatever you create, your rival can use too.

### 2.2 Map
- Square grid, **20×20** cells for the slice map.
- Each player's **capital town** starts in an opposite corner region. Each capital starts with a **revealed starting area** (cells within Chebyshev distance 2 of the town): fixed steppe/forest with one Sawmill and one Ore Pit. Every other cell is **Mist**.
- Heroes cannot enter Mist cells.
- Each revealed cell has: terrain type, optional site (mine, lair, landmark, town), optional road, optional owner (for capturable sites).

### 2.3 Day loop (one player turn)
1. **Draw:** draw cards until the hand holds 5. Unplayed cards stay in hand (max hand 7; excess is discarded at end of turn). When the draw pile is empty, shuffle the discard pile into a new draw pile.
2. **Shape:** play cards by spending **Will**. Will refills to 3 each turn (+1 from the Cartographer's Lodge). Card costs range from 0 to 3.
3. **Move:** the hero spends movement points (base 20 per day). Costs per cell: road 2, steppe 3, hills 4, forest 4, marsh 6, river ford 4.
4. **Interact:** stepping onto a site captures mines, visits landmarks, starts battles at lairs, or attacks the rival hero/town.
5. **Town:** build **one** building per day in your town (if affordable).
6. **End turn.**

The calendar runs in days. A week is 7 days of the first player's turns. **On day 1 of each week**, town dwellings produce new recruits.

### 2.4 Placement rules
A card that places a tile must target a cell that:
- is **Mist**;
- is orthogonally adjacent to at least one revealed cell;
- is within the player's **influence**: Chebyshev distance ≤ 3 from the player's hero, **or** ≤ 2 from any town or site the player owns.

Effect cards (e.g. Fog of Nav) have their own targeting rules, listed on the card.

### 2.5 Resources
**Gold** (from villages and towns), **Wood** (Sawmill), **Ore** (Ore Pit), **Amber** (Amber Shore; rare, used for top buildings and spells). Owned mines give their income at the start of the owner's turn. Towns give gold.

### 2.6 Cards (slice set: 21)

| # | Card | Type | Cost | Effect |
|---|------|------|------|--------|
| 1 | Birch Forest | Terrain | 1 | Place forest (slow; battle: tree obstacles) |
| 2 | Steppe | Terrain | 0 | Place steppe (fast) |
| 3 | Marsh | Terrain | 1 | Place marsh (very slow; blocks rival paths) |
| 4 | Hills | Terrain | 1 | Place hills (battles fought here: ranged units ignore the long-range penalty) |
| 5 | River Ford | Terrain | 1 | Place river ford |
| 6 | Sawmill | Site | 2 | Place steppe + Sawmill (+2 wood/day to owner) |
| 7 | Ore Pit | Site | 2 | Place hills + Ore Pit (+2 ore/day) |
| 8 | Amber Shore | Site | 3 | Place steppe + Amber Shore (+1 amber/day) |
| 9 | Village | Site | 2 | Place steppe + Village (+250 gold/day) |
| 10 | Wolf Den | Lair T1 | 1 | Place forest + lair with weak guards; loot: 500 gold + draft 1 of 3 cards |
| 11 | Kurgan Barrow | Lair T2 | 2 | Place hills + lair with medium guards; loot: 1000 gold, 5 ore + draft 1 of 3 |
| 12 | Leshy's Grove | Lair T3 | 3 | Place forest + lair with strong guards; loot: 2 amber, 1500 gold + draft 1 of 3 (rare pool) |
| 13 | Baba Yaga's Hut | Landmark | 2 | Place forest + hut: first visit per hero teaches a secondary skill (choose 1 of 2) |
| 14 | Shrine of Perun | Landmark | 2 | Place steppe + shrine: first visit teaches a spell |
| 15 | Crossroads Stone | Landmark | 1 | Place steppe + stone: each visit (once per week) draft 1 of 3 cards |
| 16 | Raskolnitsa Road | Effect | 1 | Add road to up to 3 connected revealed cells within influence |
| 17 | Fog of Nav | Effect | 2 | Return one revealed cell within influence to Mist. Cannot target towns, heroes, owned sites, or uncleared lairs |
| 18 | Watchtower *(Druzhina)* | Site | 2 | Place hills + tower: owner influence radius 3 around it, +2 vision |
| 19 | Monastery Field *(Druzhina)* | Site | 2 | Place steppe + field: +1 Will per turn while owned (multiple fields do not stack) |
| 20 | Cursed Mire *(Nav)* | Terrain | 1 | Place marsh: enemy heroes ending a turn here lose 10% of their weakest stack |
| 21 | Rusalka Lake *(Nav)* | Site | 2 | Place ford + lake: Nav recruits +2 Rusalki per week while owned |

**Lairs and risk/reward:** lairs are neutral. Whoever clears one gets the loot, and the cell becomes **Ruins** (passable, no further effect). Placing a lair near the rival is a gamble: it blocks them, but rewards them if they win.

**Deck rules:**
- Starter deck: 12 cards per faction (defined in data).
- **Draft 1 of 3:** pick one of three cards from a pool and add it to your discard pile.
- **Burn:** instead of playing a card, remove it from your deck permanently and gain +1 Will this turn (deck thinning).

### 2.7 Factions (slice)

| Faction | Town | Units (T1→T4) | Flavor |
|---|---|---|---|
| **Druzhina** | Kremlin with a monastery | Militia (melee), Streltsy Archers (ranged), Druzhinniki (cavalry melee), Bogatyr (elite melee) | Steady, defensive, economy and roads |
| **Nav** | Witch-hut hamlet | Upyri (melee, life drain), Rusalki (ranged, drowning slow), Leshy (tank), Zmey (flying dragon) | Aggressive, spreads marsh and lairs |

**Town buildings (per faction, 7):**
- Hall (gold income, upgradeable once)
- Dwelling T1, Dwelling T2, Dwelling T3, Dwelling T4 (each requires the previous one)
- Mage Shrine (unlocks the faction spell list)
- Cartographer's Lodge (+1 Will; once per week, draft 1 of 3 cards)

### 2.8 Heroes
One hero per player in the slice.
- **Primary stats:** Attack, Defense, Spell Power, Knowledge (mana = 10 × Knowledge).
- **XP and leveling:** each level gives +1 to a primary stat and a choice of 1 of 2 secondary skills.
- **Secondary skills (slice):** Pathfinding, Archery, Leadership, Wisdom, Cartography (+1 influence radius).

### 2.9 Battles
- **Field:** 11×9 hex grid (odd-r offset coordinates). Obstacles (0–4) come from the map tile's terrain: forest gives trees, hills give rocks, marsh gives slow-hexes.
- **Armies:** up to **5 stacks** per side. A stack is a unit type plus a count.
- **Unit stats:** HP, Attack, Defense, Damage (min–max), Speed (hexes), Initiative, Shots (ranged only), plus ability flags.
- **Turn order:** stacks act by Initiative (ties go to the attacker). Actions: **Move**, **Attack** (move + melee), **Shoot**, **Wait** (act later this round), **Defend** (+30% defense until next action).
- **Damage formula:** `dmg = count × rand(min,max) × clamp(1 + 0.05×(atk − def), 0.3, 3.0)`. Damage kills whole creatures top-down, and the remainder reduces the top creature's HP.
- **Retaliation:** once per round, per stack, against melee attacks.
- **Ranged:** half damage if an enemy is adjacent to the shooter, or if the target is more than 6 hexes away.
- **Hero:** casts at most one spell per round. Slice spells per faction (2 each):
  - Druzhina: *Bless*, *Stone Skin*.
  - Nav: *Curse*, *Wail of Nav* (damage).
- **Battle end:** one side has no stacks, or a player **retreats**: the hero survives and the army is lost (the hero returns to the capital next week). Auto-resolve runs the same battle simulation with a simple AI controlling both sides.
- **Rewards:** XP equal to the total HP of defeated enemies.

### 2.10 Win / Lose
A player wins when the rival **loses their capital** (an enemy hero ends a move on it while it is undefended, or wins the siege battle there). A player also wins when the rival's hero is defeated and the rival owns no town. Sieges in the slice are a normal battle with the town garrison as defenders; there are no walls.

### 2.11 Modes
- **Vs AI:** 1 human vs 1 AI rival.
- **Hot-seat:** 2 humans on one machine. Between turns a "Pass to Player N" screen hides the hand and the map view.

---

## 3. Technical Architecture

### 3.1 Stack
- **Godot 4** stable, GDScript. The exact version is pinned at the start of Milestone 1 and recorded in `README.md`.
- **Desktop target:** Windows, Linux, macOS.
- **Tests:** GUT (Godot Unit Test), run headless.

### 3.2 Layering
```
TheGame/
  project.godot
  core/            # pure logic — RefCounted classes, NO Nodes, NO scene access
    state/         # GameState, MapState, Cell, Player, Hero, Town, Army, Stack, Deck, BattleState
    commands/      # PlayCard, BurnCard, MoveHero, BuildBuilding, Recruit, EndTurn, Draft,
                   #   BattleMove, BattleAttack, BattleShoot, BattleWait, BattleDefend, CastSpell, Retreat
    rules/         # validation + resolution: placement, influence, pathfinding (A*), economy,
                   #   combat math, battle turn order, win checks
    rng.gd         # seeded deterministic RNG (single source of randomness)
    game.gd        # facade: submit(command) -> Result(events | error)
  data/            # Godot Resources (.tres): cards, units, factions, buildings, spells, skills, maps
  ai/              # rival AI: enumerates legal commands, scores them, submits the best
  scenes/          # presentation only: adventure_map, battle, town, hand_ui, pass_screen, menus
  art/             # Aseprite exports (sprite sheets + JSON), placeholders until delivered
  tests/           # GUT tests
```

### 3.3 Data flow (the one rule)
`UI or AI → Command → rules.validate(state) → rules.apply(state) → [Events] → scenes animate events`

- **Nothing mutates `GameState` except applying a Command.** Scenes hold no game truth; they render state and play back events.
- Commands are plain data and serialize to and from Dictionaries/JSON.
- All randomness goes through `core/rng.gd`, seeded per game. The RNG state is part of `GameState`.

**This buys us:**
- **Determinism:** seed + command log reproduces the exact game.
- **Save/load:** `user://saves/*.json` stores the seed plus the command log (and a state snapshot for fast load).
- **Replays and bug reports:** replay the log.
- **Online (phase 2):** lockstep. Peers exchange Commands only; each peer validates and applies them locally.
- **AI parity:** the AI uses exactly the same Commands as a human.
- **Hot-seat:** only changes which player is allowed to submit.

### 3.4 Content as data
Cards, units, factions, buildings, spells and skills are Resource scripts (`CardDef`, `UnitDef`, ...) with `.tres` instances. Card effects use a small set of effect types (`place_tile`, `place_site`, `add_road`, `revert_to_mist`, ...) with parameters. Adding a normal card needs **no code**.

### 3.5 Presentation
- **Adventure map:** `TileMapLayer` (32×32 tiles) for terrain, a layer for sites and roads, and a Mist overlay. Camera pan/zoom. Highlights for influence and legal placement cells. Path preview with movement cost.
- **Hand UI:** card row at the bottom. Drag a card onto a highlighted cell, or click card then cell. Right-click to burn.
- **Battle scene:** hex grid rendering, stack sprites, initiative bar, action buttons, spell book.
- **Town screen:** building list with costs, and recruitment.

### 3.6 AI (slice)
Utility scoring over legal commands each turn:
- **Economy:** place and capture mines early.
- **Pressure:** place lairs and marsh near the rival's frontier.
- **Combat:** fight when estimated army strength is ≥ 1.3× the defender's; otherwise avoid.
- **Town:** build toward the highest dwelling and recruit weekly.

Battle AI: target the highest-threat stack it can reach, and shoot when possible. One difficulty level.

### 3.7 Testing
- **Unit tests (GUT, headless):** placement legality, influence, A* costs, economy ticks, weekly growth, deck draw/shuffle/burn, damage formula, retaliation, initiative order, win conditions.
- **Determinism test:** run a scripted command log twice from the same seed and assert identical serialized state.
- **Save/load round-trip test.**
- **AI smoke test:** AI vs AI for 60 days finishes without errors or illegal commands.
- **Manual playtest:** in the Godot editor for feel and balance.

---

## 4. Art Spec (for the artist, Aseprite)

| Asset | Size | Notes |
|---|---|---|
| Map terrain tiles | 32×32 | steppe, forest, marsh, hills, ford, road overlays, ruins, Mist (animated, 4–8 frames) |
| Map sites | 32×32 (towns 64×64) | sawmill, ore pit, amber shore, village, lairs ×3, landmarks ×3, watchtower, monastery field, rusalka lake, 2 towns |
| Map hero | 32×32 | 4 directions × walk (4 frames) per faction |
| Battle units | 48×48 | facing right only (mirrored in code); anims: idle, move, attack, hurt, death; 8 units |
| Card art | 64×88 | illustration only; frame, cost and text are drawn in code |
| Battle backgrounds | 352×288 | one per terrain type |
| UI icons | 16×16 | 4 resources, Will, movement, stats |

- **Palette:** one shared limited palette (32–64 colors) for everything.
- **Export:** sprite sheet PNG + JSON (Aseprite "Export Sprite Sheet", array format, with tags per animation).
- **Placeholders:** colored shapes and letters are used until art is delivered. Art can be dropped in without code changes.

---

## 5. Milestones

1. **Core foundation:** GameState, Commands, deterministic RNG, serialization, GUT setup, determinism test.
2. **Adventure map:** Mist, deck and hand, card placement with influence, hero movement with A*, placeholder art.
3. **Economy and towns:** resources, mines, day/week cycle, buildings, recruitment, town screen.
4. **Hex battles:** grid, initiative, move/melee/ranged/retaliation, spells, retreat, auto-resolve.
5. **Content:** lairs with guardians and loot, landmarks, drafting, all 21 cards, both factions, hero leveling.
6. **Opponents:** rival AI (map + battle) and hot-seat pass screen.
7. **Finish the slice:** save/load UI, art integration, balance pass, menus, desktop export.

**Phase 2 (after the slice):** online multiplayer (lockstep over Godot high-level multiplayer/ENet, or a relay), more factions, multiple heroes, artifacts, campaign.

---

## 6. Success Criteria for the Slice
- A full game vs AI (about 30–60 minutes) can be played start to finish without crashes.
- Hot-seat 2-player works with hidden hands.
- Playtesters face real placement decisions every turn (judged by feel in playtests).
- The determinism, save/load and AI-vs-AI smoke tests pass headless.
