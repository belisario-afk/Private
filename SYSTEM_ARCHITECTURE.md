# KillaDome System Architecture Diagram

## High-Level System Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              RUST SERVER (OXIDE)                             │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         KillaDome Plugin Core                          │ │
│  │                                                                        │ │
│  │  ┌──────────────────┐        ┌──────────────────┐                    │ │
│  │  │  Oxide Hooks     │────────│ Command Handlers │                    │ │
│  │  │                  │        │                  │                    │ │
│  │  │ • Init           │        │ • Chat (/kd)     │                    │ │
│  │  │ • OnConnect      │        │ • Console (kd.*) │                    │ │
│  │  │ • OnDisconnect   │        │ • UI (killadome.)│                    │ │
│  │  │ • OnDeath        │        │                  │                    │ │
│  │  └────────┬─────────┘        └────────┬─────────┘                    │ │
│  │           │                           │                              │ │
│  │           └───────────────┬───────────┘                              │ │
│  │                           │                                          │ │
│  │              ┌────────────▼────────────┐                             │ │
│  │              │   Session Manager       │                             │ │
│  │              │                         │                             │ │
│  │              │ _activeSessions         │                             │ │
│  │              │ Dictionary<ulong,       │                             │ │
│  │              │   PlayerSession>        │                             │ │
│  │              └────────────┬────────────┘                             │ │
│  │                           │                                          │ │
│  └───────────────────────────┼──────────────────────────────────────────┘ │
│                              │                                            │
│  ┌───────────────────────────┴──────────────────────────────────────────┐ │
│  │                        System Modules Layer                          │ │
│  │                                                                      │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │ │
│  │  │ DomeManager  │  │   LobbyUI    │  │LoadoutEditor │            │ │
│  │  │              │  │              │  │              │            │ │
│  │  │• StartMatch  │  │• ShowUI      │  │• SelectItem  │            │ │
│  │  │• EndMatch    │  │• DestroyUI   │  │• EquipItem   │            │ │
│  │  │• Queue       │  │• 5 Tabs      │  │              │            │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘            │ │
│  │                                                                      │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │ │
│  │  │ Attachment   │  │  AlterEgo    │  │   Weapon     │            │ │
│  │  │   System     │  │   System     │  │ Progression  │            │ │
│  │  │              │  │              │  │              │            │ │
│  │  │• Definitions │  │• AE Stats    │  │• GetLevel    │            │ │
│  │  │• CalcStats   │  │• Synergies   │  │• Upgrade     │            │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘            │ │
│  │                                                                      │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │ │
│  │  │ VFXManager   │  │ SFXManager   │  │ForgeStation  │            │ │
│  │  │              │  │              │  │   System     │            │ │
│  │  │• PlayVFX     │  │• PlaySFX     │  │              │            │ │
│  │  └──────────────┘  └──────────────┘  │• UpgradeCost │            │ │
│  │                                       │• UpgradeAtt  │            │ │
│  │  ┌──────────────┐  ┌──────────────┐  └──────────────┘            │ │
│  │  │ BloodToken   │  │   StoreAPI   │                               │ │
│  │  │  Economy     │  │              │  ┌──────────────┐            │ │
│  │  │              │  │• Purchase    │  │SaveManager   │            │ │
│  │  │• Award       │  │• Tebex       │  │              │            │ │
│  │  │• Spend       │  │              │  │• LoadProfile │            │ │
│  │  │• GetBalance  │  └──────────────┘  │• SaveProfile │            │ │
│  │  └──────────────┘                    └──────┬───────┘            │ │
│  │                                              │                      │ │
│  │  ┌──────────────┐  ┌──────────────┐         │                      │ │
│  │  │ AntiExploit  │  │ Telemetry    │         │                      │ │
│  │  │              │  │   System     │         │                      │ │
│  │  │• RateLimit   │  │              │         │                      │ │
│  │  │• Validate    │  │• RecordKill  │         │                      │ │
│  │  └──────────────┘  └──────────────┘         │                      │ │
│  │                                              │                      │ │
│  └──────────────────────────────────────────────┼──────────────────────┘ │
│                                                 │                        │
│  ┌──────────────────────────────────────────────▼──────────────────────┐ │
│  │                      Data Persistence Layer                         │ │
│  │                                                                     │ │
│  │        oxide/data/KillaDome/{steamId}.json (Per-Player Files)      │ │
│  │                                                                     │ │
│  │  ┌──────────────────────────────────────────────────────────┐     │ │
│  │  │ PlayerProfile Structure:                                 │     │ │
│  │  │                                                          │     │ │
│  │  │ • SteamID                                                │     │ │
│  │  │ • Loadouts (Primary/Secondary/Attachments/Skins)         │     │ │
│  │  │ • WeaponLevels (Dictionary)                              │     │ │
│  │  │ • AttachmentLevels (Dictionary)                          │     │ │
│  │  │ • OwnedSkins (List)                                      │     │ │
│  │  │ • Tokens (int)                                           │     │ │
│  │  │ • Stats (Kills/Deaths/Matches)                           │     │ │
│  │  │ • IsVIP (bool)                                           │     │ │
│  │  │ • LastUpdated (DateTime)                                 │     │ │
│  │  └──────────────────────────────────────────────────────────┘     │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           External Dependencies                             │
│                                                                              │
│  ┌────────────────────┐          ┌────────────────────┐                    │
│  │  ImageLibrary      │          │    Tebex API       │                    │
│  │    Plugin          │          │   (Optional)       │                    │
│  │                    │          │                    │                    │
│  │ • GetImage()       │          │ • Payment          │                    │
│  │ • Store PNGs       │          │   Processing       │                    │
│  └────────────────────┘          └────────────────────┘                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Data Flow Diagrams

### Player Connection Flow

```
Player Connects
      │
      ▼
OnPlayerConnected(player)
      │
      ├─────────────────────────────────────┐
      │                                     │
      ▼                                     ▼
SaveManager.LoadPlayerProfile()     DomeManager
      │                                     │
      │ Returns PlayerProfile               │
      │                                     │
      ▼                                     │
Create PlayerSession                        │
      │                                     │
      ├─> Store in _activeSessions         │
      │                                     │
      ▼                                     │
TeleportToLobby(player) ─────────────────>│
      │                                     │
      │                                     │
      ▼                                     │
Timer (1s delay)                            │
      │                                     │
      ▼                                     │
LobbyUI.ShowLobbyUI(player)                 │
      │                                     │
      ▼                                     │
   Display UI                               │
```

### Purchase Flow

```
Player Clicks "BUY" in Store
             │
             ▼
UI Command: killadome.purchase <itemId> <cost>
             │
             ▼
    CmdPurchase(player, itemId, cost)
             │
             ├────────────────────────────┐
             │                            │
             ▼                            ▼
  AntiExploit.CheckRateLimit()    Get PlayerSession
             │                            │
             │ Pass                       │
             │                            ▼
             │                   Check Token Balance
             │                            │
             │                            │ Sufficient
             │                            │
             └────────────────┬───────────┘
                              │
                              ▼
            StoreAPI.PurchaseItem(steamId, itemId, cost)
                              │
                              ├──────────────────────────┐
                              │                          │
                              ▼                          ▼
         BloodTokenEconomy.SpendTokens()    Add to OwnedSkins
                              │                          │
                              │ Success                  │
                              │                          │
                              └──────────┬───────────────┘
                                         │
                                         ▼
                         SaveManager.SavePlayerProfile()
                                         │
                                         ▼
                            LobbyUI.ShowLobbyUIWithTab("store")
                                         │
                                         ▼
                                   UI Refreshes
```

### Kill Event Flow

```
Player Kills Another Player
            │
            ▼
OnEntityDeath(victim, hitInfo)
            │
            ├────────────────────────────────┐
            │                                │
            ▼                                ▼
Extract Attacker Info             Extract Victim Info
            │                                │
            ▼                                │
BloodTokenEconomy.AwardTokens()              │
    (attacker, tokensPerKill)                │
            │                                │
            ▼                                │
TelemetrySystem.RecordKill()                 │
    (attacker, victim)                       │
            │                                │
            ├──────────────┐                 │
            │              │                 │
            ▼              ▼                 │
Increment          Increment                 │
attacker.          victim.                   │
TotalKills         TotalDeaths               │
            │              │                 │
            └──────┬───────┘                 │
                   │                         │
                   └─────────────────────────┘
                               │
                               ▼
                    Timer (3s delay)
                               │
                               ▼
                    TeleportToLobby(victim)
                               │
                               ▼
                      victim.Respawn()
```

### Loadout Application Flow

```
Player Enters Arena
         │
         ▼
TeleportToArena(player)
         │
         ▼
ApplyLoadout(player)
         │
         ├─────────────────────────────┐
         │                             │
         ▼                             ▼
Get PlayerSession              player.inventory.Strip()
         │
         │
         ▼
Get Loadout from Profile
         │
         ├─────────────────┬─────────────────┐
         │                 │                 │
         ▼                 ▼                 ▼
GiveWeapon(        GiveWeapon(         Give Ammo
  Primary,           Secondary,
  Attachments,       Attachments,
  Skins)             Skins)
         │                 │
         ├─────────────────┘
         │
         ▼
For Each Weapon:
    ├─> Map weapon name to item ID
    ├─> Create item (ItemManager.CreateByName)
    ├─> Apply skin (item.skin = skinId)
    ├─> For each attachment:
    │      ├─> Create attachment item
    │      └─> Add to weapon.contents
    └─> player.inventory.GiveItem(weapon)
         │
         ▼
    Loadout Applied
```

### Synergy Calculation Flow

```
Player Equips Attachments
            │
            ▼
AlterEgoSystem.CalculateModifiers(attachments, profile)
            │
            ▼
╔═══════════════════════════════════════════════════════╗
║            11-Step Synergy Engine                     ║
╠═══════════════════════════════════════════════════════╣
║ Step 1-2: Gather Active AE Attachments               ║
║           Convert to Percent Changes                  ║
║           ├─> For each attachment:                    ║
║           │   ├─> Get AttachmentStats                 ║
║           │   ├─> Convert mul to percent              ║
║           │   │   (0.90 → -10%, 1.10 → +10%)         ║
║           │   └─> Add to list                        ║
║                                                       ║
║ Step 3:   Sum All Percent Changes                    ║
║           ├─> RecoilMul: -10% + -8% = -18%          ║
║           ├─> DamageMul: +3% + +7% = +10%           ║
║           └─> ... for each stat                      ║
║                                                       ║
║ Step 4:   Apply Level-Based Scaling                  ║
║           ├─> For each attachment:                    ║
║           │   ├─> Get level from profile             ║
║           │   ├─> scaling = 1.0 + (level × 0.05)    ║
║           │   └─> Multiply percent change            ║
║                                                       ║
║ Step 5:   Check Pair Synergies (10 rules)           ║
║           ├─> For each pair rule:                    ║
║           │   ├─> Check if both attachments equipped ║
║           │   └─> Apply bonus multipliers            ║
║                                                       ║
║ Step 6:   Check Triple Synergies (5 rules)          ║
║           ├─> For each triple rule:                  ║
║           │   ├─> Check if all 3 equipped            ║
║           │   └─> Apply bonus multipliers            ║
║                                                       ║
║ Step 7:   Convert Percent → Multipliers              ║
║           ├─> mul = 1.0 + (percent / 100)           ║
║           │   (-18% → 0.82, +10% → 1.10)            ║
║                                                       ║
║ Step 8:   Apply Additive Stats                       ║
║           ├─> OnHitHP += ...                         ║
║           ├─> BleedChance += ...                     ║
║           └─> StaggerChance += ...                   ║
║                                                       ║
║ Step 9:   Cap Multipliers                            ║
║           ├─> Clamp(value, 0.5, 2.0)                ║
║           │   Prevents extremes                      ║
║                                                       ║
║ Step 10:  Set VFX/SFX Flags                         ║
║           ├─> Check for special attachments          ║
║           └─> Set boolean flags                      ║
║                                                       ║
║ Step 11:  Return WeaponModifierState                 ║
╚═══════════════════════════════════════════════════════╝
            │
            ▼
Return Final Modifier State
```

## Module Dependency Graph

```
┌──────────────────────────────────────────────────────────────────┐
│                        KillaDome (Main)                          │
│                     (Orchestrates everything)                    │
└───┬──────────────────────────────────────────────────────────┬───┘
    │                                                          │
    ├──────────────────────────────────┬───────────────────────┤
    │                                  │                       │
    ▼                                  ▼                       ▼
┌────────────┐                   ┌────────────┐         ┌──────────┐
│SaveManager │                   │AntiExploit │         │Telemetry │
│            │                   │            │         │  System  │
│(No deps)   │                   │(No deps)   │         │(No deps) │
└────────────┘                   └────────────┘         └──────────┘
    
    │                                  │                       │
    ├──────────────────────────────────┼───────────────────────┤
    │                                  │                       │
    ▼                                  ▼                       ▼
┌──────────────────┐          ┌────────────────┐      ┌──────────────┐
│BloodTokenEconomy │          │AttachmentSystem│      │AlterEgoSystem│
│                  │          │                │      │              │
│• Main Plugin     │          │• Main Plugin   │      │• Main Plugin │
│• Config          │          │• Config        │      │              │
└──────────────────┘          └────────────────┘      └──────────────┘
    │
    │
    ├──────────────────────────────────┬───────────────────────┐
    │                                  │                       │
    ▼                                  ▼                       ▼
┌────────────────┐          ┌──────────────────┐      ┌──────────────┐
│WeaponProgression│         │  VFXManager      │      │  SFXManager  │
│                │          │                  │      │              │
│• Main Plugin   │          │• Main Plugin     │      │• Main Plugin │
│• Config        │          └──────────────────┘      └──────────────┘
└────────────────┘
    │
    │
    ├──────────────────────────────────────────────────────────┐
    │                                                          │
    ▼                                                          │
┌───────────────────────────┐                                 │
│   ForgeStationSystem      │                                 │
│                           │                                 │
│ Depends on:               │                                 │
│ • Main Plugin             │                                 │
│ • Config                  │                                 │
│ • BloodTokenEconomy       │◄────────────────────────────────┘
│ • AttachmentSystem        │
│ • WeaponProgression       │
└───────────────────────────┘
    │
    │
    ├──────────────────────────────────┬───────────────────────┐
    │                                  │                       │
    ▼                                  ▼                       │
┌────────────────┐          ┌──────────────────┐              │
│ LoadoutEditor  │          │   StoreAPI       │              │
│                │          │                  │              │
│• Main Plugin   │          │• Main Plugin     │              │
│• AttachmentSys │          │• Config          │              │
└────────────────┘          │• BloodTokenEcon  │◄─────────────┘
                            └──────────────────┘
    │                                  │
    │                                  │
    ├──────────────────────────────────┤
    │                                  │
    ▼                                  │
┌────────────────────────────────────┐ │
│           LobbyUI                  │ │
│                                    │ │
│ Depends on:                        │ │
│ • Main Plugin                      │ │
│ • LoadoutEditor                    │◄┘
│ • ForgeStationSystem               │
│ • StoreAPI                         │
└────────────────────────────────────┘
    │
    │
    ▼
┌────────────────┐
│  DomeManager   │
│                │
│• Main Plugin   │
│• Config        │
└────────────────┘
```

## UI State Machine

```
                    Player Connects
                          │
                          ▼
                  ┌───────────────┐
                  │   NO UI       │
                  │   (Loading)   │
                  └───────┬───────┘
                          │
                          │ ShowLobbyUI()
                          ▼
                  ┌───────────────┐
             ┌────│   PLAY TAB    │────┐
             │    │   (Default)   │    │
             │    └───────────────┘    │
             │            │            │
             │            │            │
   Tab:      │            │            │    Tab:
   "loadouts"│            │            │    "store"
             │            │            │
             ▼            │            ▼
    ┌────────────┐        │      ┌──────────┐
    │ LOADOUTS   │        │      │  STORE   │
    │    TAB     │        │      │   TAB    │
    └─────┬──────┘        │      └────┬─────┘
          │               │           │
          │               │           │
          │      Tab:     │           │
          │    "stats" or │           │
          │   "settings"  │           │
          │               │           │
          │               ▼           │
          │       ┌───────────────┐   │
          └──────►│  STATS TAB /  │◄──┘
                  │  SETTINGS TAB │
                  └───────┬───────┘
                          │
                          │ Close Button
                          │ or DestroyUI()
                          ▼
                  ┌───────────────┐
                  │   NO UI       │
                  └───────────────┘

Within LOADOUTS TAB:
┌─────────────────────────────────────────────┐
│  Primary/Secondary Weapon Selection         │
│         ↓                                   │
│  [Edit Primary] [Edit Secondary]            │
│         ↓                                   │
│  [Scopes] [Silencers] [Underbarrel]        │
│         ↓                                   │
│  Display Filtered Attachments               │
│         ↓                                   │
│  Click [APPLY] → Update Loadout             │
└─────────────────────────────────────────────┘
```

## Economy Flow

```
╔══════════════════════════════════════════════════════════╗
║                  Blood Token Sources                     ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  ┌────────────────┐      ┌────────────────┐            ║
║  │ New Player     │      │ Kill Enemy     │            ║
║  │ Starting Bonus │      │ +10 tokens     │            ║
║  │ +500 tokens    │      └────────┬───────┘            ║
║  └────────┬───────┘               │                    ║
║           │                       │                    ║
║           │       ┌───────────────┘                    ║
║           │       │                                    ║
║           │       │      ┌────────────────┐            ║
║           │       │      │ Dice Game Win  │            ║
║           │       │      │ +2x bet        │            ║
║           │       │      │ (10-100)       │            ║
║           │       │      └────────┬───────┘            ║
║           │       │               │                    ║
║           └───────┼───────────────┤                    ║
║                   │               │                    ║
║                   │               │                    ║
║                   ▼               │                    ║
║           ┌────────────────┐      │                    ║
║           │ External Store │      │                    ║
║           │ (Tebex)        │      │                    ║
║           │ Variable       │      │                    ║
║           └────────┬───────┘      │                    ║
║                    │              │                    ║
║                    └──────┬───────┘                    ║
║                           │                            ║
║                           ▼                            ║
║                  ┌─────────────────┐                   ║
║                  │  Token Balance  │                   ║
║                  │  (Profile.      │                   ║
║                  │   Tokens)       │                   ║
║                  └────────┬────────┘                   ║
║                           │                            ║
╚═══════════════════════════╪════════════════════════════╝
                            │
╔═══════════════════════════╪════════════════════════════╗
║                  Token Expenditures                    ║
╠═══════════════════════════╪════════════════════════════╣
║                           │                            ║
║       ┌───────────────────┼───────────────────┐        ║
║       │                   │                   │        ║
║       ▼                   ▼                   ▼        ║
║  ┌─────────┐       ┌──────────┐       ┌──────────┐   ║
║  │Gun Skins│       │Regular   │       │Alter-Ego │   ║
║  │400-500  │       │Attachmnts│       │Attachmnts│   ║
║  │ tokens  │       │150-400   │       │550-800   │   ║
║  └─────────┘       │ tokens   │       │ tokens   │   ║
║                    └──────────┘       └──────────┘   ║
║                                                        ║
║       ┌───────────────────┼───────────────────┐        ║
║       │                   │                   │        ║
║       ▼                   ▼                   ▼        ║
║  ┌─────────┐       ┌──────────┐       ┌──────────┐   ║
║  │Weapon   │       │Attachment│       │Dice Game │   ║
║  │Upgrades │       │Upgrades  │       │Bet       │   ║
║  │100-1000 │       │100-500   │       │10-100    │   ║
║  │ tokens  │       │ tokens   │       │ tokens   │   ║
║  └─────────┘       └──────────┘       └──────────┘   ║
║                                                        ║
╚════════════════════════════════════════════════════════╝
```

## Security Architecture

```
╔═══════════════════════════════════════════════════════════╗
║                   Security Layers                         ║
╠═══════════════════════════════════════════════════════════╣
║                                                           ║
║  Layer 1: Permission System                              ║
║  ┌─────────────────────────────────────────────────┐     ║
║  │ • killadome.admin → Admin commands              │     ║
║  │ • killadome.vip → VIP features                  │     ║
║  │                                                 │     ║
║  │ Check: permission.UserHasPermission()          │     ║
║  └─────────────────────────────────────────────────┘     ║
║                          │                               ║
║                          ▼                               ║
║  Layer 2: Rate Limiting                                  ║
║  ┌─────────────────────────────────────────────────┐     ║
║  │ • Sliding window: 5 actions/second              │     ║
║  │ • Per-player tracking                           │     ║
║  │ • Queue of timestamps                           │     ║
║  │                                                 │     ║
║  │ Check: AntiExploit.CheckRateLimit()            │     ║
║  └─────────────────────────────────────────────────┘     ║
║                          │                               ║
║                          ▼                               ║
║  Layer 3: Input Validation                               ║
║  ┌─────────────────────────────────────────────────┐     ║
║  │ • Type checking (TryParse)                      │     ║
║  │ • Range validation                              │     ║
║  │ • Null checks                                   │     ║
║  │ • Session validation                            │     ║
║  └─────────────────────────────────────────────────┘     ║
║                          │                               ║
║                          ▼                               ║
║  Layer 4: Business Logic Validation                      ║
║  ┌─────────────────────────────────────────────────┐     ║
║  │ • Balance checks (tokens >= cost)               │     ║
║  │ • Ownership checks (owns item before apply)     │     ║
║  │ • Cooldown checks (dice game)                   │     ║
║  │ • Level checks (max level reached?)             │     ║
║  └─────────────────────────────────────────────────┘     ║
║                          │                               ║
║                          ▼                               ║
║  Layer 5: Data Integrity                                 ║
║  ┌─────────────────────────────────────────────────┐     ║
║  │ • Atomic file saves (temp → rename)             │     ║
║  │ • Exception handling                            │     ║
║  │ • Fallback to defaults on error                 │     ║
║  │ • Transaction-like purchases                    │     ║
║  └─────────────────────────────────────────────────┘     ║
║                                                           ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Synergy Matrix

### Pair Synergies (10 combinations)

```
┌──────────────────┬──────────────────┬────────────────────────┐
│  Attachment 1    │  Attachment 2    │  Synergy Bonus         │
├──────────────────┼──────────────────┼────────────────────────┤
│ Small Scope      │ Muzzle Brake     │ -12% recoil            │
│                  │                  │ +5% ADS speed          │
├──────────────────┼──────────────────┼────────────────────────┤
│ Small Scope      │ Holo Sight       │ -5% aimcone            │
│                  │                  │ -4% ADS speed          │
├──────────────────┼──────────────────┼────────────────────────┤
│ 8x Scope         │ Oil Filter       │ +7% velocity           │
│                  │                  │ +3% bleed chance       │
├──────────────────┼──────────────────┼────────────────────────┤
│ 8x Scope         │ Muzzle Brake     │ -5% recoil             │
│                  │                  │ +10% h. stability      │
├──────────────────┼──────────────────┼────────────────────────┤
│ Holo Sight       │ Laser Sight      │ -15% aimcone           │
│                  │                  │ -5% ADS speed          │
├──────────────────┼──────────────────┼────────────────────────┤
│ Holo Sight       │ Muzzle Boost     │ +4% fire rate          │
│                  │                  │ +5% aimcone (trade-off)│
├──────────────────┼──────────────────┼────────────────────────┤
│ Laser Sight      │ Silencer         │ -8% aimcone            │
│                  │                  │ +3% stagger chance     │
├──────────────────┼──────────────────┼────────────────────────┤
│ Laser Sight      │ Soda Can         │ -3% recoil             │
│                  │                  │ +3% damage             │
├──────────────────┼──────────────────┼────────────────────────┤
│ Muzzle Brake     │ Soda Can         │ -7% recoil             │
│                  │                  │ +2% damage             │
├──────────────────┼──────────────────┼────────────────────────┤
│ Muzzle Boost     │ Soda Can         │ +6% fire rate          │
│                  │                  │ +12% recoil (trade-off)│
└──────────────────┴──────────────────┴────────────────────────┘
```

### Triple Synergies (5 combinations)

```
┌──────────────┬──────────────┬──────────────┬─────────────────┐
│ Attachment 1 │ Attachment 2 │ Attachment 3 │ Synergy Bonus   │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ Holo Sight   │ Laser Sight  │ Muzzle Boost │ +3% fire rate   │
│              │              │              │ -7% aimcone     │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ 8x Scope     │ Oil Filter   │ Silencer     │ +5% velocity    │
│              │              │              │ +4% damage      │
│              │              │              │ +2% bleed       │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ Small Scope  │ Muzzle Brake │ Laser Sight  │ -10% recoil     │
│              │              │              │ +8% h. stability│
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ Soda Can     │ Muzzle Boost │ Muzzle Brake │ +4% fire rate   │
│              │              │              │ +5% recoil      │
│              │              │              │ +3% damage      │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ Holo Sight   │ Silencer     │ Small Scope  │ -7% ADS speed   │
│              │              │              │ -5% aimcone     │
└──────────────┴──────────────┴──────────────┴─────────────────┘
```

---

This architecture document provides visual representations of how all the systems connect and interact within the KillaDome plugin.
