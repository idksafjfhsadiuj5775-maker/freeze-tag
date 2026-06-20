# Freeze Tag for AllOut

A multiplayer Freeze Tag game built for [AllOut](https://allout.game/).

**Game ID:** `6a3553184da8f6e90b0f614b`

## Quick Start (0 sparks)

1. **Create a GitHub repo** (public or private) at https://github.com/new

2. **Push this code to it:**
   ```bash
   cd freezetag
   git init
   git add .
   git commit -m "Initial commit - Freeze Tag full game code"
   git branch -M main
   git remote add origin https://github.com/YOUR_USER/YOUR_REPO.git
   git push -u origin main
   ```

3. **Link it to AllOut:**
   - Go to https://allout.game/create/6a3553184da8f6e90b0f614b
   - Click **"Collaborate"** tab
   - Click **"Manage GitHub Connection"** -> authorize AllOut
   - Select your repo from the dropdown

4. Done. AllOut will sync your code automatically.

## Project Structure

```
freezetag/
  scripts/
    main.csl                Core game loop, match states, powerups, zones
    player.csl              Player class, freeze/rescue/progression
    abilities/
      init.csl              15 abilities (Common to Mythic)
    effects/
      init.csl              15+ status effects
    maps/
      init.csl              10 maps with hazards, secrets, weather
    ui/
      init.csl              HUD, menus, minimap, leaderboard, shop
    progression/
      init.csl              Levels, prestige, rebirth, achievements, quests
    economy/
      init.csl              10 currencies, 30+ shop items, crafting, trading
    events/
      init.csl              20+ events (seasonal, bosses, invasions)
    social/
      init.csl              Friends, parties, guilds, chat, leaderboards
```

## Game Features

- **5 freeze tiers:** None → Slowed → Partial → Full → Permafrost
- **Rescue system:** Teammates thaw frozen players (with progress bar)
- **15 abilities:** Dash, Shield, Teleport, Freeze Aura, Ultra Freeze, Mass Rescue, etc.
- **10 maps:** Crystal Caverns, Arctic Arena, Cyber Freeze, Lava Tag, Sky Gardens, etc.
- **10 game modes:** Classic, Infection, Chaos, Speed, Zero Gravity, Boss Battle, Survival, etc.
- **Powerups:** Speed Boost, Invisibility, Phase Shift, Clone, Magnet, Repulsion, etc.
- **Progression:** 100 levels, 10 prestige tiers, 10 rebirths, mastery system
- **Economy:** 10 currencies, shop, crafting, blueprints, trading, bundles
- **Events:** Seasonal, holiday, world events, boss battles, invasions, tournaments
- **Social:** Friends, parties, guilds, chat, leaderboards, match history
- **Arena hazards:** Conveyor belts, lava pools, zero-G zones, wind currents, ghost apparitions
- **Secrets:** Hidden rooms, teleport pads, boost pads, mystery merchants

## Design Document

`FREEZE_TAG_5000_IDEAS.md` contains 5,000 unique ideas (75K lines) for years of future updates.

## Requirements

- AllOut account (free)
- Git + GitHub account (free)
- No sparks needed to deploy via Git

## License

All code is provided for use on AllOut platform only.
