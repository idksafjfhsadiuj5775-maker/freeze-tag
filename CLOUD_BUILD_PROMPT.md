# FREEZE TAG — Ultimate Cloud Build Prompt

**Use this prompt in Cloud Build with "1 Option, High Quality" (or whatever fits 10 sparks).**

---

## GAME OVERVIEW

Freeze Tag is a team-based multiplayer game where two teams (FREEZERS vs RUNNERS) compete in rounds. Freezers chase and tag Runners to freeze them. Runners dodge Freezers and rescue frozen teammates. Round ends when all Runners are frozen or time runs out. Best of 5 rounds wins.

**Target:** 2-16 players, ages 10+, top-down 2D arena, casual competitive.
**Style:** Ice/cyber aesthetic, 44x34 arena, orthographic camera.

---

## STEP 1: IMPORT ALL CSL FILES

My GitHub repo has 100 CSL files organized in subdirectories. Import them ALL using the editor's script import system:

### From scripts/main.csl (entry point — run this FIRST):
- Copy ALL 64 import lines into the main CSL script file
- This gives you: GameConfig, MatchManager, GameLoop, PlayerClass, FreezeSystem, all 15 abilities, all 8 effects, all 8 map files, all 5 UI files, all 24+ system files

### Key configuration file:
- `scripts/game/GameConfig.csl` — Contains EVERY tunable setting. Read this and apply the default values to the scene:
  - round_time = 120.0, freeze_duration = 15.0, min_players = 2, max_players = 16
  - arena_width = 44, arena_height = 34, player_base_speed = 5.0
  - tag_range = 1.5, rescue_range = 1.5, rescue_time = 2.0
  - freezer_ratio = 0.3, powerup_interval = 10.0
  - sudden_death_enabled = true, rounds_per_match = 5

### Effect init files to import second:
- `scripts/effects/init.csl` — Contains ALL visual effects: Frozen_Effect (ice particles + blue tint), Rescue_Effect (yellow burst), Invincible_Effect (flashing), Slow_Effect (purple), Speed_Boost_Effect (yellow trail), Shield_Effect (circle halo), Teleport_Effect (magenta burst), Dash_Effect (cyan trail), and environmental effects (Lava_Damage, Healing_Zone, Wind, Zero_Gravity, Spawn, Elimination, Victory).

---

## STEP 2: CREATE THE SCENE

### 2A — Fundamentals
Create a new scene named "FreezeTagArena". Set the camera:
- Camera type: ORTHOGRAPHIC
- Position: (0, 0, -50)
- Size/OrthoHeight: 25 (so the 44x34 arena fills the screen)
- Background color: (0.1, 0.1, 0.15, 1) — dark arena feel

### 2B — Arena Floor
Create `Entity` named "ArenaFloor":
- Position: (0, 0)
- Scale: (44, 34)
- Add `Sprite_Renderer` component: color = (0.2, 0.25, 0.35, 1), choose a grid/tile texture from asset catalog (tile_02 or grid_01)
- Add optional `Tilemap` for a grid pattern

### 2C — Boundary Walls
Create 4 invisible wall entities. Each needs a `Box_Collider` or `Collision_Shape` component. Do NOT add sprites (invisible walls):

| Name | Position | Scale | Notes |
|------|----------|-------|-------|
| Wall_Top | (0, 18) | (46, 1) | Top boundary |
| Wall_Bottom | (0, -18) | (46, 1) | Bottom boundary |
| Wall_Left | (-23, 0) | (1, 36) | Left boundary |
| Wall_Right | (23, 0) | (1, 36) | Right boundary |

### 2D — Decorative Elements
Add visual decoration to make the arena feel alive (optional, from asset catalog):
- Ice crystals scattered around edges
- Floor decals marking spawn zones (blue tint on left half, green tint on right half)
- Glowing center circle at (0, 0) radius ~3
- Corner pillars or lamps

### 2E — Powerup Spawn Points
Create 8 invisible marker entities at these positions (just as reference points, the CSL script handles spawning):
POSITIONS: (-15, -8), (-8, 5), (0, 0), (8, -5), (15, 8), (-5, -12), (5, 12), (0, -8)

Add a small glowing child entity to each with a `Sprite_Renderer` (white, alpha 0.3, scale 0.5) as a visual indicator of spawn points.

---

## STEP 3: PLAYER MODEL SETUP

### 3A — Player Prefab
Create a Player Prefab named "FreezeTagPlayer":
- Add `Sprite_Renderer`: choose a humanoid/character sprite from asset catalog (e.g. "character_01"). Default white color.
- Add `Circle_Collider` with radius ~0.7 (for tag collision detection)
- Add the script: **scripts/player/PlayerClass.csl** — This gives the Player class
- Add the script: **scripts/player/FreezeSystem.csl** — This gives freeze/rescue methods

The Player class automatically handles:
- WASD/arrow key movement at config.player_base_speed
- Team assignment (FREEZERS blue, RUNNERS green) via MatchManager
- Freeze state machine: NONE → SLOWED → FULL/PERMAFROST
- Rescue mechanics when Runners touch frozen Runners
- Invincibility, shield, invisibility, phase state timers
- Score tracking, streak tracking
- Arena boundary clamping

### 3B — Spawn Setup
- **Set Player Spawn Location** in the scene settings to "None" (CSL script handles spawning via the MatchManager class which calls player.spawn() on round start)
- Freezers spawn on the LEFT side of arena: x ~= -20, random y
- Runners spawn on the RIGHT side: x ~= 20, random y
- The spawn() method in PlayerClass automatically positions players based on their team

### 3C — Player Model Setup in Editor
In the Player model/settings:
- Set Movement Type to "Custom" (CSL handles movement)
- Enable "Custom Player Script" and attach the Player class
- Set initial Color to white (1,1,1,1)
- Enable "Can collide with entities"

---

## STEP 4: CONNECT SCRIPTS TO THE SCENE

### 4A — Global Game Scripts
Add a "GameManager" entity (empty, at origin) and attach these scripts. They run the game loop:

1. **scripts/game/MatchManager.csl** — Game state machine (WAITING → COUNTDOWN → PLAYING → OVERTIME → ROUND_END → MATCH_END). Team assignment, round counting, scoring.
2. **scripts/game/GameLoop.csl** — Main update loop calling ao_start and ao_update. Handles all state transitions, win condition checks (all Runners frozen → Freezers win), sudden death shrinking circle, overtime logic.
3. **scripts/game/GameConfig.csl** — All tunable parameters. The `config` variable is global and read by every other script.
4. **scripts/maps/ArenaSetup.csl** — Creates invisible wall entities at 4 boundaries, defines powerup spawn positions. Call `create_arena()` on game start.
5. **scripts/maps/PowerupSystem.csl** — Powerup spawning and collection. Handles 6 powerup types with visual colors.
6. **scripts/maps/MapDefinitions.csl** — 10 map definitions with themes, hazards, ambient colors.
7. **scripts/maps/Hazards.csl** — Environmental hazards (lava, lasers, blizzard, ghosts, wind, zero_g, poison, sandstorm, conveyors). Call `spawn_hazards(map_name)` on map load and `process_hazards(dt)` each frame.
8. **scripts/maps/WeatherSystem.csl** — Dynamic weather effects.
9. **scripts/maps/Secrets.csl** — 5 hidden secret areas with gem/coin rewards.
10. **scripts/maps/TeleportPads.csl** — Teleportation pads.
11. **scripts/maps/InteractiveObjects.csl** — Buttons, doors, switches.

### 4B — UI Scripts
Attach these to a "UIManager" entity (or add to GameManager):

1. **scripts/ui/HUD.csl** — Draws timer (large centered countdown), team indicator (top-left), scoreboard (left side with both teams), streak display (bottom-right), controls hint (bottom-center).
2. **scripts/ui/MainMenu.csl** — Title screen when match_state == WAITING. Shows "FREEZE TAG" title, player count, START button.
3. **scripts/ui/Announcements.csl** — Floating announcement messages.
4. **scripts/ui/Minimap.csl** — Small minimap showing player positions.
5. **scripts/ui/ShopUI.csl** — Cosmetics shop interface.

### 4C — System Scripts
Attach these to GameManager or dedicated manager entities:

1. **systems/Progression.csl** — Level/XP system. `add_xp(player, amount)` levels up at 100 + level*50 XP. Every 10 levels gives 50 gems bonus.
2. **systems/Achievements.csl** — Achievement tracking and rewards.
3. **systems/DailyQuests.csl** — 3 daily quests assigned per player with progress tracking and rewards.
4. **systems/DailyLogin.csl** — Login streak rewards (7-day cycle).
5. **systems/MatchRewards.csl** — End-of-match XP/coin distribution.
6. **systems/EconomySystem.csl** — 5 currencies (coins, gems, tokens, shards, tickets). Player starts with 500 coins, 100 gems, 5 tokens, 3 tickets.
7. **systems/LootBoxSystem.csl** — 2 loot boxes (Standard Crate 500 coins, Rare Crate 100 gems) with weighted random drops.
8. **systems/CraftingSystem.csl** — Item crafting recipes.
9. **systems/TradingSystem.csl** — Player-to-player trading.
10. **systems/Leaderboard.csl** — Global leaderboard by level/wins/tags.
11. **systems/FriendSystem.csl** — Friend list, invitations.
12. **systems/FriendBonus.csl** — +5% XP/reward bonus per friend (up to 10 friends, 50% max).
13. **systems/GuildSystem.csl** — Guild creation, members, level/xp, recruitment.
14. **systems/GuildUI.csl** — Guild menu UI.
15. **systems/ChatSystem.csl** — In-game text chat.
16. **systems/SpectatorMode.csl** — Spectator camera, free-fly mode.
17. **systems/MatchHistory.csl** — Stores last 50 match results per player.
18. **systems/SecretAreas.csl** — 5 hidden areas with rewards. Call `check_secrets(player)` each frame.
19. **systems/EventManager.csl** — Timed events (2x XP, bonus rewards).
20. **systems/AntiCheat.csl** — Speed hacks, impossible movement detection.
21. **systems/BotAI.csl** — AI bots that chase/flee/rescue. Call find_nearest_runner, find_nearest_freezer, find_nearest_frozen_teammate.
22. **systems/InputHelper.csl** — Mobile touch controls, controller remapping.
23. **systems/DebugCommands.csl** — Dev mode overlay with FPS, player count, state info.
24. **systems/Tutorial.csl** — 6-step tutorial for new players (level 1 only).
25. **systems/TitleSystem.csl** — 10 unlockable titles (Rookie, The Freezer, The Savior, Veteran, Legend, Prestigious, Reborn, Champion, Collector, Speed Demon).
26. **systems/MasterySystem.csl** — Mastery levels (Novice → Apprentice → Adept → Expert → Master → Grandmaster). Each level gives +1% stats.
27. **systems/PrestigeSystem.csl** — Prestige at level 50. Resets level, gives gems. 10 prestige levels.
28. **systems/RebirthSystem.csl** — Rebirth at level 100. Resets everything, gives tokens. 10 rebirth levels.
29. **systems/SeasonPass.csl** — Battle pass with 5-tier repeating reward cycle (coins, gems, skins, tokens).
30. **systems/BattlePassUI.csl** — Battle pass progress display.
31. **systems/Statistics.csl** — Full player statistics screen.
32. **systems/CollectionSystem.csl** — 4 item collections (Ice Set, Fire Set, Gold Set, Starter Set) with completion bonuses.
33. **systems/NotificationSystem.csl** — Stacking notification popups with auto-fade.
34. **systems/PlayerColors.csl** — Team colors, rarity colors, freeze tinting.
35. **systems/SoundManager.csl** — Sound effect stubs (tag, rescue, powerup, countdown, round_end, victory, defeat, boss_hit, teleport, secret).
36. **systems/MapVoting.csl** — Vote for next map between rounds. 4 choices from rotation. 10-second vote window.
37. **systems/ReplaySystem.csl** — Records last 30 seconds of gameplay. Playback controls (play, pause, speed 0.5x-2x).
38. **systems/LoreSystem.csl** — 8 lore entries (The Great Freeze, Freezer Clan, Runner Resistance, Power Crystals, Frozen Arenas, Ice King, Frost Dragon, Thaw Prophecy).
39. **systems/ReportSystem.csl** — Player reporting.
40. **systems/PerformanceStats.csl** — FPS tracking (60-frame rolling average).
41. **systems/DevTools.csl** — Dev mode overlay.
42. **systems/Animations.csl** — Bounce, pulse, shake, particle spawning (freeze, rescue, powerup, victory).
43. **systems/GameModes.csl** — 10 game modes:
    - CLASSIC — Standard freeze tag
    - INFECTION — Tagged Runners become Freezers
    - SPEED — 2x speed, half freeze duration
    - CHAOS — 50% Freezers, 5s freeze
    - SURVIVAL — 3min round, 30s freeze, respawn
    - KOTH — Hold center to win (no timer)
    - ZERO_GRAVITY — Floaty movement
    - HARDCORE — Permanent freeze, 5s rescue
    - REVERSE_TAG — Runners tag Freezers
    - BOSS_RUSH — Defeat waves of boss enemies
44. **systems/BossSystem.csl** — Boss entity with 500 HP, 4 phases, freeze attacks. Drops rewards on defeat. Color shifts from red to yellow-green as damaged.
45. **systems/Settings.csl** — Audio/video settings menu.
46. **systems/DailyRewards.csl** — 7-day daily reward calendar with streak tracking.
47. **systems/QuestSystem.csl** — 7 quest types (tag, rescue, win, powerup, maps, boss, secret) with progress bars.
48. **systems/Matchmaking.csl** — Skill-based team balancing using snake draft. Skill = level + prestige*50 + win_rate*100.
49. **systems/CollectionSystem.csl** — Set collection bonuses.

### 4D — Ability Scripts
Each ability is a separate file. Connect ability buttons to the UI:

| File | Ability | Cooldown | Key | Description |
|------|---------|----------|-----|-------------|
| abilities/DashAbility.csl | Dash | 5s | Q | Quick burst forward |
| abilities/ShieldAbility.csl | Shield | 10s | E | Absorbs 3 tags |
| abilities/SpeedBoostAbility.csl | Speed Boost | 8s | Shift | 2x speed for 5s |
| abilities/TeleportAbility.csl | Teleport | 15s | F | Teleport to cursor |
| abilities/FreezeAuraAbility.csl | Freeze Aura | 12s | 1 | AoE freeze around Freezer |
| abilities/InvisibilityAbility.csl | Invisibility | 15s | 2 | Stealth for 5s |
| abilities/MassRescueAbility.csl | Mass Rescue | 20s | 3 | Unfreeze all nearby Runners |
| abilities/RescueAuraAbility.csl | Rescue Aura | 10s | 4 | AoE rescue pulse |
| abilities/SuperJumpAbility.csl | Super Jump | 6s | Space (hold) | Launch over enemies |
| abilities/UltraFreezeAbility.csl | Ultra Freeze | 25s | 5 | Instant freeze in radius |
| abilities/TimeSlowAbility.csl | Time Slow | 20s | 6 | Slow nearby enemies |
| abilities/MagnetAbility.csl | Magnet | 12s | 7 | Pull items/players toward you |
| abilities/RepulsionAbility.csl | Repulsion | 12s | 8 | Push enemies away |
| abilities/CloneAbility.csl | Clone | 18s | 9 | Decoy clone |
| abilities/PhaseShiftAbility.csl | Phase Shift | 14s | 0 | Walk through walls/tags |

### 4E — Effect Scripts
The 8 effect files (and the comprehensive init.csl) are auto-triggered by the game systems:
- FrozenEffect — Freeze visual (blue tint, ice particles, movement disabled)
- SpeedEffect — Speed boost (yellow trail)
- RescueEffect — Rescue flash (yellow burst particles)
- SlowEffect — Slowed visual (purple tint)
- InvincibleEffect — Flashing invincibility
- PowerupPickupEffect — Pickup animation
- SpawnEffect — Spawn particles
- VictoryEffect — Victory particles

---

## STEP 5: MONETIZATION SETUP (Cosmetics Only — NO Pay-to-Win)

### Currencies (configure in Economy settings):
| Currency | Starting Amount | Max | Purchase With |
|----------|----------------|-----|---------------|
| coins | 500 | 999999 | Play, watch ads |
| gems | 100 | 99999 | Play, purchase |
| tokens | 5 | 9999 | Events, premium |
| shards | 0 | 99999 | Crafting, collection bonuses |
| tickets | 3 | 99 | Daily login |

### Cosmetics Shop (in scripts/ui/ShopUI.csl):
- Skins: blue (common), red (rare), gold (epic), green (uncommon), purple (rare)
- Trails: ice (rare), fire (epic), neon (legendary)
- Pets/Hats: penguin (epic), dragon (mythic)
- Emotes: wave, dance, freeze, flex

### Products to create in AllOut store:
- "Starter Pack" — 500 coins + skin_blue + trail_ice (one-time purchase)
- "Freezer Bundle" — gems + fire trail (premium)
- "Legend Bundle" — gold skin + penguin pet + tokens (ultra)

### Ad Placement:
- Show interstitial ad after each match end (player choice: watch for +50% XP bonus)
- Reward video ad: watch for 100 coins (3x per day)
- Banner ad on main menu (optional)

---

## STEP 6: PUBLISHING SETTINGS

### Game Settings:
- Game Title: "Freeze Tag"
- Description: "A fast-paced multiplayer Freeze Tag game. Freeze all opponents or rescue your teammates to win! 10 maps, 15 abilities, and tons of cosmetics."
- Genre: Action / Party
- Max Players: 16
- Min Players: 2
- Age Rating: All ages (10+)
- Monetization: Cosmetics only (skins, trails, pets, emotes)
- Language: English

### Server Settings:
- Region: Auto
- Tick Rate: 20 (default for 2D)
- Timeout: 30s
- Allow Spectators: Yes
- Allow Bots: Yes (fill empty slots)

---

## STEP 7: WHAT THE CORE GAME LOOP DOES (for reference)

1. **Game Starts** → state = WAITING, main menu shows "FREEZE TAG"
2. **2+ players join** → START button appears
3. **START clicked** → COUNTDOWN (5s), "3, 2, 1" shown
4. **Round begins** → PLAYING state, teams assigned (30% Freezers, 70% Runners)
5. **Every frame** → Player movement, tag checks (Freezer touches Runner → freeze), rescue checks (Runner touches frozen Runner → rescue), powerup spawns every 10s, hazards process, secrets check, UI draws
6. **All Runners frozen** → ROUND_END, Freezer wins +1
7. **Time runs out** → OVERTIME (30s) with shrinking deadly circle (sudden death)
8. **Sudden death circle** starts at radius 20, shrinks 0.3/sec. Players outside take freeze damage.
9. **Overtime ends** → ROUND_END, whoever froze more Runners wins
10. **Score to win** = 3 points (best of 5 rounds)
11. **Match ends** → Show winner announcement, distribute rewards (XP, coins based on performance), check achievements/titles, check quests
12. **Back to WAITING** → Map vote starts (4 random maps, 10s vote), next match on winning map

### Scoring:
- Tag a Runner: +15 points
- Get frozen (Runner): +10 points
- Rescue a Runner: +25 pts rescuer, +20 pts rescued
- Win round: +50 points bonus
- Streak: displayed on HUD (green < 5, yellow < 10, red 10+)

---

## FINAL CHECKLIST

Before clicking "Apply to Live," verify:

- [ ] Camera is orthographic at (0, 0, -50), ortho height 25
- [ ] Arena floor exists at (0, 0) sized 44x34
- [ ] 4 invisible wall entities with colliders at boundaries
- [ ] Player prefab with Sprite_Renderer, Circle_Collider (r=0.7), custom Player script
- [ ] GameManager entity with all system scripts attached
- [ ] UIManager entity with HUD, MainMenu, Announcements, Minimap, ShopUI scripts
- [ ] All 15 ability files imported
- [ ] All 8 effect files imported
- [ ] Economy configured with 5 currencies
- [ ] Products created in store (Starter Pack, Freezer Bundle, Legend Bundle)
- [ ] Ad placements configured
- [ ] Game settings match the table above
- [ ] GitHub repo linked so future CSL pushes auto-sync
- [ ] Test with 2+ clients: movement, tagging, freezing, rescuing, powerups, round cycle
- [ ] Test sudden death (overtime timer runs out)
- [ ] Test bot AI fills empty slots
- [ ] Test shop purchases and cosmetic equipping
- [ ] Test progression (XP, levels, prestige, rebirth)
- [ ] Test daily quests and rewards
- [ ] Test map voting between rounds
- [ ] Test all 10 game modes
- [ ] Test boss battles
- [ ] Test secret areas
- [ ] Test abilities (cooldowns, key binds, visuals)
- [ ] Test spectate mode
- [ ] Test friend/guild systems
- [ ] Test trading system
- [ ] Test crafting system
- [ ] Test loot boxes
- [ ] Test replay system
- [ ] Check performance: stable 60 FPS with 16 players
- [ ] Apply to Live!
