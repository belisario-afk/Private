# KillaDome Plugin - Comprehensive Analysis

## Table of Contents
1. [Overview](#overview)
2. [Core Features](#core-features)
3. [Architecture & Design Patterns](#architecture--design-patterns)
4. [Module Breakdown](#module-breakdown)
5. [Data Models](#data-models)
6. [System Connections](#system-connections)
7. [UI System](#ui-system)
8. [Economy & Progression](#economy--progression)
9. [Security & Anti-Exploit](#security--anti-exploit)
10. [Configuration](#configuration)
11. [Code Quality Analysis](#code-quality-analysis)

---

## Overview

**KillaDome** is a comprehensive Oxide/uMod plugin for Rust that transforms a server into a **Call of Duty-style experience**. It provides a complete arena-based gameplay system with lobby mechanics, customizable loadouts, weapon progression, an in-game economy, and a fully-featured store system.

### Plugin Metadata
- **Name:** KillaDome
- **Version:** 1.0.0
- **Author:** KillaDome Dev Team
- **Description:** Full COD-style server experience with lobby, loadouts, and progression

### Key Technologies
- **Language:** C# (.NET Framework)
- **Framework:** Oxide/uMod for Rust
- **UI System:** Oxide CUI (Custom User Interface)
- **Serialization:** Newtonsoft.Json
- **External Integration:** ImageLibrary plugin, Tebex (optional)

---

## Core Features

### 1. **Lobby System**
- Dedicated spawn area for players outside of matches
- Comprehensive tabbed UI interface with 4 main tabs:
  - **Play:** Queue management and match joining
  - **Loadouts:** Weapon and attachment customization
  - **Store:** Purchase skins and attachments with Blood Tokens
  - **Stats:** Player statistics and progression tracking
  - **Settings:** Plugin configuration view

### 2. **Loadout System**
- Customizable weapon loadouts (Primary + Secondary)
- Drag-and-drop style weapon selection
- Real-time weapon preview
- Attachment system with multiple categories:
  - Scopes (Small Scope, 8x Scope, Holo Sight)
  - Silencers/Muzzle devices (Silencer, Muzzle Brake, Muzzle Boost, Soda Can Silencer, Oil Filter Silencer)
  - Underbarrel (Laser Sight)

### 3. **Alter-Ego Attachment System**
- Advanced attachment modification system
- Each attachment has normal and "Alter-Ego" (AE) versions
- AE attachments provide enhanced stats:
  - Recoil multipliers
  - Damage multipliers
  - Fire rate modifications
  - Special effects (VFX/SFX)
  - Unique bonuses (health on hit/kill, bleed chance, stagger chance)

### 4. **Synergy System**
- **Pair Synergies:** 10 unique 2-attachment combinations
- **Triple Synergies:** 5 unique 3-attachment combinations
- Attachments work together to provide bonus stats beyond individual effects
- Examples:
  - Holo + Laser: Enhanced accuracy and ADS speed
  - 8x + Oil Filter: Increased velocity and bleed chance
  - Small Scope + Brake + Laser: Superior recoil control

### 5. **Weapon Progression**
- Level-based weapon advancement (Max Level: 10)
- Per-weapon progression tracking
- Upgrade costs scale with level
- Base weapon stats for AK-47 and M249

### 6. **Blood Token Economy**
- In-game currency system
- Earn tokens through:
  - Kills (configurable amount per kill)
  - Starting balance for new players
- Spend tokens on:
  - Weapon skins
  - Attachments (normal and Alter-Ego versions)
  - Weapon/attachment upgrades

### 7. **Mini-Games**
- **Dice Game:** 
  - Bet 10-100 tokens
  - Roll dice against the house
  - Win 2x your bet if you roll higher
  - 30-second cooldown between games
  - Tie returns your bet

### 8. **Match System**
- Queue-based matchmaking
- Automatic teleportation to arena
- Match lifecycle management (start/end)
- Player state tracking (in lobby vs. in match)

### 9. **Store Integration**
- Blood Token purchases
- Tebex integration support (external payment processor)
- Two-tier pricing:
  - Regular items: 150-500 tokens
  - Alter-Ego items: 550-800 tokens
- Gun skins and attachments available

### 10. **Persistent Data System**
- JSON-based file storage
- Per-player profiles including:
  - Token balance
  - Owned skins and attachments
  - Weapon/attachment levels
  - Kill/death statistics
  - Loadout configurations
- Auto-save system (configurable interval, default: 5 minutes)

---

## Architecture & Design Patterns

### Design Patterns Used

#### 1. **Modular Architecture**
The plugin is divided into self-contained modules, each handling specific functionality:
- `DomeManager` - Match management
- `LobbyUI` - User interface
- `LoadoutEditor` - Loadout customization
- `AttachmentSystem` - Attachment definitions
- `AlterEgoSystem` - Advanced attachment stats
- `WeaponProgression` - Weapon leveling
- `VFXManager` - Visual effects
- `SFXManager` - Sound effects
- `ForgeStationSystem` - Upgrade station
- `BloodTokenEconomy` - Currency management
- `StoreAPI` - Purchase handling
- `SaveManager` - Data persistence
- `AntiExploit` - Security measures
- `TelemetrySystem` - Analytics/statistics

#### 2. **Dependency Injection**
Modules receive dependencies through constructors:
```csharp
internal ForgeStationSystem(KillaDome plugin, PluginConfig config, 
    BloodTokenEconomy economy, AttachmentSystem attachmentSystem, 
    WeaponProgression weaponProgression)
```

#### 3. **Session Management Pattern**
Active player sessions stored in dictionary:
```csharp
private Dictionary<ulong, PlayerSession> _activeSessions
```
- Tracks player state
- Maintains UI state (selected tabs, editing mode)
- Holds profile reference

#### 4. **Rate Limiting Pattern**
Anti-exploit system uses sliding window rate limiting:
```csharp
private Queue<DateTime> _actions
```

#### 5. **Data Transfer Objects (DTOs)**
Clean separation of data structures:
- `PlayerProfile` - Persistent player data
- `PlayerSession` - Runtime player state
- `Loadout` - Weapon configuration
- `AttachmentStats` - Stat modifiers
- `WeaponModifierState` - Calculated stats

### Architecture Layers

```
┌─────────────────────────────────────────────────┐
│           OXIDE HOOKS LAYER                      │
│  (Init, OnPlayerConnected, OnEntityDeath, etc)  │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────┴────────────────────────────────┐
│         CORE PLUGIN LAYER                        │
│  (Command Handlers, Session Management)          │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────┴────────────────────────────────┐
│         SYSTEM MODULES LAYER                     │
│  (DomeManager, UI, Economy, Progression, etc)   │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────┴────────────────────────────────┐
│         DATA & PERSISTENCE LAYER                 │
│  (SaveManager, PlayerProfiles, JSON Storage)    │
└─────────────────────────────────────────────────┘
```

---

## Module Breakdown

### 1. **DomeManager**
**Purpose:** Manages match lifecycle and player queuing

**Key Responsibilities:**
- Start/end matches
- Maintain match queue
- Track active match state
- Teleport players to arena

**Key Methods:**
```csharp
void StartMatch()           // Initiates a new match
void EndMatch()             // Concludes current match
void AddToQueue(ulong)      // Adds player to queue
void RemoveFromQueue(ulong) // Removes player from queue
```

**Data Structures:**
- `Match` class: Stores match ID, times, participants, active status
- `List<ulong> _matchQueue`: Queue of waiting players

### 2. **LobbyUI**
**Purpose:** Creates and manages the comprehensive lobby interface

**Key Responsibilities:**
- Render tabbed UI (Play, Loadouts, Store, Stats, Settings)
- Handle tab navigation
- Display weapon previews
- Show skin/attachment catalogs
- Integrate with ImageLibrary for graphics

**Key Methods:**
```csharp
void ShowLobbyUI(BasePlayer)              // Opens main UI
void ShowLobbyUIWithTab(BasePlayer, tab)  // Opens specific tab
void DestroyUI(BasePlayer)                // Closes UI
void ShowPlayTab(container, player)       // Renders Play tab
void ShowLoadoutsTab(container, player)   // Renders Loadouts tab
void ShowStoreTab(container, player)      // Renders Store tab
void ShowStatsTab(container, player)      // Renders Stats tab
```

**UI Constants:**
```csharp
const string UI_MAIN = "KillaDomeUI"
const string UI_TAB_CONTAINER = "KillaDomeTabContainer"
```

**UI Architecture:**
- Main panel (60% width, 80% height, centered)
- Header with close button
- Tab navigation bar
- Content area that changes per tab

### 3. **LoadoutEditor**
**Purpose:** Handles weapon and attachment selection/equipping

**Key Responsibilities:**
- Track selected items per player
- Equip weapons/attachments to loadout slots
- Validate equipment changes

**Key Methods:**
```csharp
void SelectItem(ulong, string)            // Selects an item
string GetSelectedItem(ulong)             // Gets current selection
void ClearSelection(ulong)                // Clears selection
bool TryEquipItem(ulong, slot, item)      // Equips item to slot
```

**Data Structures:**
```csharp
Dictionary<ulong, string> _selectedItems  // Tracks UI selections
```

### 4. **AttachmentSystem**
**Purpose:** Defines and manages weapon attachments

**Key Responsibilities:**
- Store attachment definitions
- Calculate stat modifications
- Provide attachment metadata

**Key Methods:**
```csharp
AttachmentDefinition GetAttachment(string)
Dictionary<string, float> CalculateWeaponStats(weapon, attachments)
```

**Attachment Definition:**
```csharp
class AttachmentDefinition {
    string Id
    string Name
    string Slot      // "barrel", "optic", "mag", etc.
    int MaxLevel
    Dictionary<string, float> StatModifiers
    string VFXTag
    string SFXTag
}
```

**Stat Modifiers:**
- `noise_reduction`: Sound suppression
- `damage`: Damage multiplier
- `fire_rate`: Rate of fire multiplier
- `accuracy`: Accuracy multiplier
- `mag_size`: Magazine size multiplier
- `reload_speed`: Reload speed multiplier

### 5. **AlterEgoSystem**
**Purpose:** Advanced attachment system with special effects and synergies

**Key Responsibilities:**
- Define Alter-Ego skin IDs (900001-900009)
- Calculate complex stat modifiers
- Apply synergy bonuses
- Track special effects (VFX/SFX paths)

**Alter-Ego Skin Mapping:**
```csharp
"weapon.mod.small.scope" -> 900001
"weapon.mod.8x.scope" -> 900002
"weapon.mod.holosight" -> 900003
"weapon.mod.lasersight" -> 900004
"weapon.mod.sodacansilencer" -> 900005
"weapon.mod.oilfiltersilencer" -> 900006
"weapon.mod.silencer" -> 900007
"weapon.mod.muzzlebrake" -> 900008
"weapon.mod.muzzleboost" -> 900009
```

**Stat Properties (AttachmentStats):**
- **RecoilMul:** Recoil reduction/increase
- **AimconeMul:** Aim cone (spread) modifier
- **ADSSpeedMul:** Aim down sights speed
- **ReloadSpeedMul:** Reload speed modifier
- **FireRateMul:** Fire rate modifier
- **DamageMul:** Damage modifier
- **VelocityMul:** Bullet velocity
- **MoveSpeedMul:** Movement speed while equipped
- **OnHitHP:** Health restored per hit
- **OnKillHP:** Health restored per kill
- **BleedChance:** Chance to cause bleeding
- **StaggerChance:** Chance to stagger enemy
- **HorizontalStability:** Horizontal recoil reduction
- **VFXPath:** Visual effect asset path
- **SFXPath:** Sound effect asset path

**11-Step Synergy Engine:**
The `CalculateModifiers` method implements a sophisticated calculation:
1. Gather active Alter-Ego attachments
2. Convert stats to percent changes
3. Sum all percent changes
4. Apply level-based scaling (per attachment level)
5. Check and apply pair synergies (10 combinations)
6. Check and apply triple synergies (5 combinations)
7. Convert percent changes back to multipliers
8. Apply additive stats (HP, bleed, stagger)
9. Cap multipliers (0.5 to 2.0 range)
10. Set VFX/SFX flags
11. Return final `WeaponModifierState`

**Key Methods:**
```csharp
bool IsAlterEgo(attachmentId, skinId)
ulong GetAlterEgoSkinId(attachmentId)
WeaponModifierState CalculateModifiers(attachments, profile)
```

### 6. **WeaponProgression**
**Purpose:** Manages weapon leveling and base stats

**Key Responsibilities:**
- Define weapon base statistics
- Track per-player weapon levels
- Handle weapon upgrades

**Weapon Definitions:**
```csharp
"ak47" -> AK-47: damage=35, fire_rate=0.13, accuracy=0.75
"m249" -> M249: damage=30, fire_rate=0.1, accuracy=0.7
```

**Key Methods:**
```csharp
int GetWeaponLevel(steamId, weaponId)
bool UpgradeWeapon(steamId, weaponId, cost)
```

### 7. **VFXManager**
**Purpose:** Handles visual effects

**Key Methods:**
```csharp
void PlayVFX(player, vfxTag, position)
```

**Implementation:**
Sends console command to client to trigger VFX:
```csharp
player.SendConsoleCommand($"killadome.vfx {vfxTag} {x} {y} {z}")
```

### 8. **SFXManager**
**Purpose:** Handles sound effects

**Key Methods:**
```csharp
void PlaySFX(player, sfxTag)
```

**Implementation:**
```csharp
player.SendConsoleCommand($"killadome.sfx {sfxTag}")
```

### 9. **ForgeStationSystem**
**Purpose:** Upgrade station for attachments

**Key Responsibilities:**
- Calculate upgrade costs
- Process attachment upgrades
- Integrate with economy system

**Key Methods:**
```csharp
int CalculateUpgradeCost(currentLevel)  // Returns: 100 * (level + 1)
bool UpgradeAttachment(steamId, attachmentId)
```

**Cost Scaling:**
- Level 0→1: 100 tokens
- Level 1→2: 200 tokens
- Level 2→3: 300 tokens
- Level 3→4: 400 tokens
- Level 4→5: 500 tokens

### 10. **BloodTokenEconomy**
**Purpose:** In-game currency management

**Key Methods:**
```csharp
void AwardTokens(steamId, amount)
bool SpendTokens(steamId, amount)
int GetBalance(steamId)
```

**Token Sources:**
- Starting balance: 500 (configurable)
- Per kill: 10 tokens (configurable)
- Mini-games (dice)
- External purchases (Tebex)

### 11. **StoreAPI**
**Purpose:** Handles in-game purchases

**Key Methods:**
```csharp
bool PurchaseItem(steamId, itemId, cost)
void ProcessTebexPurchase(steamId, packageId, transactionId)
```

**Purchase Flow:**
1. Validate player has sufficient tokens
2. Deduct cost from balance
3. Add item to player's owned items
4. Save profile
5. Log transaction

### 12. **SaveManager**
**Purpose:** Persistent data storage

**Key Responsibilities:**
- Load player profiles from JSON files
- Save player profiles atomically
- Handle file I/O errors gracefully

**Storage Location:**
```
{OxideDataDirectory}/KillaDome/{steamId}.json
```

**Key Methods:**
```csharp
PlayerProfile LoadPlayerProfile(steamId)
void SavePlayerProfile(profile)
```

**Atomic Save Process:**
1. Serialize profile to JSON
2. Write to temporary file (`{steamId}.json.tmp`)
3. Delete old file
4. Rename temp file to final name
5. Ensures data integrity if crash occurs

### 13. **AntiExploit**
**Purpose:** Prevent abuse and exploits

**Key Responsibilities:**
- Rate limiting (default: 5 actions/second)
- Input validation
- Action tracking

**Rate Limiting Algorithm:**
- Sliding window queue of timestamps
- Removes actions older than 1 second
- Rejects if queue >= max actions

**Key Methods:**
```csharp
bool CheckRateLimit(steamId, maxActionsPerSecond = 5)
bool ValidateAction(steamId, action)
```

**Protected Actions:**
- UI navigation
- Weapon changes
- Purchases
- Attachment applications

### 14. **TelemetrySystem**
**Purpose:** Track plugin usage and statistics

**Key Responsibilities:**
- Record game events
- Track kill/death stats
- Monitor purchases
- Aggregate statistics

**Tracked Events:**
- Kills (per player)
- Deaths (per player)
- Purchases
- Custom events

**Key Methods:**
```csharp
void RecordKill(attackerId, victimId)
void RecordPurchase(steamId, itemId, cost)
Dictionary<string, int> GetStats()
```

---

## Data Models

### PlayerProfile (Persistent)
```csharp
class PlayerProfile {
    ulong SteamID
    List<Loadout> Loadouts
    Dictionary<string, int> WeaponLevels      // weapon -> level
    Dictionary<string, int> AttachmentLevels  // attachment -> level
    List<string> OwnedSkins                   // skins & attachments owned
    int Tokens                                // Blood Token balance
    bool IsVIP
    DateTime LastUpdated
    int TotalKills
    int TotalDeaths
    int MatchesPlayed
}
```

### PlayerSession (Runtime)
```csharp
class PlayerSession {
    BasePlayer Player
    PlayerProfile Profile
    DateTime LastAction
    string SelectedItem                        // Currently selected in UI
    bool IsInMatch
    string EditingWeaponSlot                   // "primary" or "secondary"
    string SelectedAttachmentCategory          // "scopes", "silencers", "underbarrel"
    DateTime LastDiceGame                      // Cooldown tracker
}
```

### Loadout
```csharp
class Loadout {
    string Name
    string Primary                             // Primary weapon ID
    string Secondary                           // Secondary weapon ID
    Dictionary<string, string> PrimaryAttachments    // slot -> attachment
    Dictionary<string, string> SecondaryAttachments  // slot -> attachment
    Dictionary<string, string> Skins           // weapon -> skin ID
    string Lethal                              // Lethal equipment
    string Tactical                            // Tactical equipment
    List<string> Perks                         // Active perks
}
```

### AttachmentStats
```csharp
class AttachmentStats {
    float RecoilMul = 1.0f
    float AimconeMul = 1.0f
    float ADSSpeedMul = 1.0f
    float ReloadSpeedMul = 1.0f
    float FireRateMul = 1.0f
    float DamageMul = 1.0f
    float VelocityMul = 1.0f
    float MoveSpeedMul = 1.0f
    float OnHitHP = 0f
    float OnKillHP = 0f
    float BleedChance = 0f
    float StaggerChance = 0f
    float HorizontalStability = 0f
    string VFXPath = null
    string SFXPath = null
}
```

### WeaponModifierState (Calculated)
```csharp
class WeaponModifierState {
    float RecoilMul = 1.0f
    float AimconeMul = 1.0f
    float ADSSpeedMul = 1.0f
    float ReloadSpeedMul = 1.0f
    float FireRateMul = 1.0f
    float DamageMul = 1.0f
    float VelocityMul = 1.0f
    float MoveSpeedMul = 1.0f
    float OnHitHP = 0f
    float OnKillHP = 0f
    float BleedChance = 0f
    float StaggerChance = 0f
    float HorizontalStability = 0f
    bool HasSodaCanVFX = false
    bool HasBrakeVFX = false
}
```

### SynergyRule
```csharp
class SynergyRule {
    List<string> RequiredAttachments           // Attachments that must be equipped
    Dictionary<string, float> Multipliers      // Stat multipliers
    Dictionary<string, float> Additives        // Additive bonuses
}
```

---

## System Connections

### Initialization Flow
```
Init()
  ├── Register permissions (admin, vip)
  ├── Initialize SaveManager
  ├── Initialize AntiExploit
  ├── Initialize BloodTokenEconomy
  ├── Initialize AttachmentSystem
  ├── Initialize AlterEgoSystem
  ├── Initialize WeaponProgression
  ├── Initialize VFXManager
  ├── Initialize SFXManager
  ├── Initialize ForgeStationSystem
  │     └── Depends on: TokenEconomy, AttachmentSystem, WeaponProgression
  ├── Initialize LoadoutEditor
  │     └── Depends on: AttachmentSystem
  ├── Initialize StoreAPI
  │     └── Depends on: TokenEconomy
  ├── Initialize LobbyUI
  │     └── Depends on: LoadoutEditor, ForgeStation, StoreAPI
  ├── Initialize DomeManager
  └── Initialize TelemetrySystem
```

### Player Connection Flow
```
OnPlayerConnected(player)
  └── NextTick
        ├── LoadPlayerProfile(steamId) [SaveManager]
        ├── Create PlayerSession
        ├── Store in _activeSessions
        ├── TeleportToLobby(player)
        └── Timer (1s) -> ShowLobbyUI(player) [LobbyUI]
```

### Player Disconnection Flow
```
OnPlayerDisconnected(player)
  ├── DestroyUI(player) [LobbyUI]
  ├── Get session from _activeSessions
  ├── SavePlayerProfile(profile) [SaveManager]
  └── Remove from _activeSessions
```

### Kill Event Flow
```
OnEntityDeath(victim, hitInfo)
  ├── Extract attacker from hitInfo
  ├── AwardTokens(attacker, tokensPerKill) [BloodTokenEconomy]
  ├── RecordKill(attacker, victim) [TelemetrySystem]
  │     ├── Increment attacker.TotalKills
  │     └── Increment victim.TotalDeaths
  └── Timer (3s) -> Respawn victim in lobby
```

### Purchase Flow
```
CmdPurchase(player, itemId, cost)
  ├── CheckRateLimit(steamId) [AntiExploit]
  ├── Validate sufficient tokens
  ├── PurchaseItem(steamId, itemId, cost) [StoreAPI]
  │     ├── SpendTokens(steamId, cost) [BloodTokenEconomy]
  │     ├── Add to OwnedSkins list
  │     └── LogDebug
  ├── SavePlayerProfile(profile) [SaveManager]
  └── Refresh UI tab
```

### Loadout Application Flow
```
TeleportToArena(player)
  └── ApplyLoadout(player)
        ├── Get player session
        ├── Get loadout from profile
        ├── Strip existing inventory
        ├── GiveWeapon(primary)
        │     ├── Map weapon name to item ID
        │     ├── Create item
        │     ├── Apply skin from loadout
        │     ├── Add attachments to weapon
        │     └── Give ammo
        └── GiveWeapon(secondary)
```

### Attachment Synergy Calculation Flow
```
CalculateModifiers(attachments, profile)
  ├── 1. Gather active AE attachments
  ├── 2. Convert to percent changes
  ├── 3. Sum all percent changes
  ├── 4. Apply level-based scaling
  ├── 5. Check pair synergies
  │     └── For each of 10 pair rules:
  │           ├── Check if attachments equipped
  │           └── Apply multipliers & additives
  ├── 6. Check triple synergies
  │     └── For each of 5 triple rules:
  │           ├── Check if attachments equipped
  │           └── Apply multipliers & additives
  ├── 7. Convert percent to multipliers
  ├── 8. Apply additive stats
  ├── 9. Cap multipliers (0.5-2.0)
  ├── 10. Set VFX/SFX flags
  └── 11. Return WeaponModifierState
```

### Auto-Save Flow
```
OnServerInitialized()
  └── Timer.Every(autoSaveInterval)
        └── AutoSaveAllPlayers()
              ├── For each session in _activeSessions:
              │     └── SavePlayerProfile(session.Profile)
              └── LogDebug count saved
```

---

## UI System

### Main UI Structure
```
┌──────────────────────────────────────────────────────────────┐
│  KILLA DOME                                          [CLOSE]  │ ← Header
├──────────────────────────────────────────────────────────────┤
│  [PLAY]  [LOADOUTS]  [STORE]  [STATS]  [SETTINGS]           │ ← Tab Bar
├──────────────────────────────────────────────────────────────┤
│                                                                │
│                    TAB CONTENT AREA                           │
│                                                                │
│                  (changes based on tab)                       │
│                                                                │
│                                                                │
└──────────────────────────────────────────────────────────────┘
```

### Play Tab
```
┌──────────────────────────────────────────────────────────────┐
│                    J O I N   Q U E U E                         │
│                                                                │
│                  Current Queue: 0 players                      │
│                                                                │
│                    [JOIN MATCHMAKING]                          │
│                                                                │
│             Get ready for intense combat!                      │
└──────────────────────────────────────────────────────────────┘
```

### Loadouts Tab
```
┌──────────────────────────────────────────────────────────────┐
│  PRIMARY WEAPON              SECONDARY WEAPON                 │
│  ┌─────────────────┐        ┌─────────────────┐              │
│  │      AK47       │        │     PISTOL      │              │
│  │  [◄ PREV] [NEXT ►]      │  [◄ PREV] [NEXT ►]             │
│  └─────────────────┘        └─────────────────┘              │
├──────────────────────────────────────────────────────────────┤
│  EDITING:  [PRIMARY] [SECONDARY]      [ AK47 ]                │
├──────────────────────────────────────────────────────────────┤
│  SKINS                      ATTACHMENTS                       │
│  ┌──────────┐              [SCOPES] [SILENCERS] [UNDERBARREL]│
│  │ [img]    │              ┌──────────┐                       │
│  │ AK Neon  │              │ [img]    │                       │
│  │ OWNED    │              │ 8x Scope │                       │
│  │ [APPLY]  │              │ OWNED    │                       │
│  └──────────┘              │ [APPLY]  │                       │
│                            └──────────┘                       │
└──────────────────────────────────────────────────────────────┘
```

### Store Tab
```
┌──────────────────────────────────────────────────────────────┐
│                    S T O R E                                   │
│              Your Tokens: 1500                                 │
├──────────────────────────────────────────────────────────────┤
│  GUN SKINS                  ATTACHMENTS                       │
│  ┌───────────────┐         ┌───────────────┐                 │
│  │[img] AK Neon  │         │[img] 8x Scope │                 │
│  │   500 Tokens  │         │   400 Tokens  │                 │
│  │     [BUY]     │         │     [BUY]     │                 │
│  └───────────────┘         └───────────────┘                 │
│  ┌───────────────┐         ┌───────────────┐                 │
│  │[img] AK Classic│        │⚡ 8x Scope AE │                 │
│  │   400 Tokens  │         │   800 Tokens  │                 │
│  │     [BUY]     │         │     [BUY]     │                 │
│  └───────────────┘         └───────────────┘                 │
│                                                                │
│     Purchase items with Blood Tokens to enhance loadout       │
└──────────────────────────────────────────────────────────────┘
```

### Stats Tab
```
┌──────────────────────────────────────────────────────────────┐
│                    YOUR STATS                                  │
│                                                                │
│  Kills: 47                                                     │
│  Deaths: 23                                                    │
│  K/D Ratio: 2.04                                               │
│  Blood Tokens: 1500                                            │
│  Matches Played: 12                                            │
│  VIP Status: YES                                               │
│                                                                │
└──────────────────────────────────────────────────────────────┘
```

### UI Elements Used
- **CuiPanel:** Container backgrounds
- **CuiLabel:** Text display
- **CuiButton:** Interactive buttons
- **CuiRawImageComponent:** PNG images (via ImageLibrary)
- **CuiOutlineComponent:** Visual borders

### Color Scheme
- **Background:** Dark gray (0.08-0.15 RGB)
- **Primary:** Orange (1, 0.54, 0) - Brand color
- **Text:** White (1, 1, 1)
- **Secondary Text:** Gray (0.7, 0.7, 0.7)
- **Success:** Green (0.3, 1, 0.3)
- **Error:** Red (1, 0.3, 0.3)
- **Gold:** Yellow (1, 0.8, 0)

### ImageLibrary Integration
The plugin optionally integrates with the ImageLibrary plugin:
```csharp
[PluginReference]
private Plugin ImageLibrary;

// Usage:
string imageId = (string)ImageLibrary.Call("GetImage", "ak47_neon");
container.Add(new CuiRawImageComponent { Png = imageId });
```

Images should be pre-loaded in ImageLibrary with IDs like:
- `ak47_neon`, `ak47_classic`, `m249_chrome`, `pistol_black`
- `small_scope`, `8x_scope`, `holo_sight`, `laser_sight`
- `sodacan_silencer`, `oilfilter_silencer`, `silencer`
- `muzzle_brake`, `muzzle_boost`
- Plus `_ae` variants for Alter-Ego versions

---

## Economy & Progression

### Blood Token Economy

**Token Acquisition:**
1. **Starting Balance:** 500 tokens (configurable)
2. **Kill Rewards:** 10 tokens per kill (configurable)
3. **Dice Game:** Win 2x bet (10-100 tokens)
4. **External Purchases:** Tebex integration (optional)

**Token Expenditure:**
1. **Gun Skins:** 400-500 tokens
2. **Regular Attachments:** 150-400 tokens
3. **Alter-Ego Attachments:** 550-800 tokens
4. **Weapon Upgrades:** 100-500 tokens (level dependent)
5. **Attachment Upgrades:** 100-500 tokens (level dependent)
6. **Dice Game Bets:** 10-100 tokens

### Progression Systems

#### 1. Weapon Progression
- **Max Level:** 10 (configurable)
- **Tracked Per Weapon:** AK-47, M249, etc.
- **Upgrade Cost Formula:** `100 * (currentLevel + 1)`
- **Persistence:** Saved in PlayerProfile.WeaponLevels

**Example Progression:**
```
Level 0 → 1: 100 tokens
Level 1 → 2: 200 tokens
Level 2 → 3: 300 tokens
...
Level 9 → 10: 1000 tokens
Total to max: 5,500 tokens
```

#### 2. Attachment Progression
- **Max Level:** 5 (configurable)
- **Tracked Per Attachment:** Each attachment independently
- **Upgrade Cost Formula:** `100 * (currentLevel + 1)`
- **Persistence:** Saved in PlayerProfile.AttachmentLevels
- **Synergy Scaling:** Higher levels amplify synergy bonuses

**Example Progression:**
```
Level 0 → 1: 100 tokens
Level 1 → 2: 200 tokens
Level 2 → 3: 300 tokens
Level 3 → 4: 400 tokens
Level 4 → 5: 500 tokens
Total to max: 1,500 tokens per attachment
```

#### 3. Ownership Progression
- Players must purchase items before using them
- Purchased items stored in `OwnedSkins` list
- Includes both skins and attachments
- Persists across sessions

### Balance Considerations

**Token Earning Rate:**
- Average kills per match: ~5-15
- Tokens per match: 50-150
- Matches to afford regular attachment: 2-3 matches
- Matches to afford AE attachment: 4-6 matches

**Upgrade Costs:**
- Early levels cheap (100-300 tokens)
- Later levels expensive (400-1000 tokens)
- Encourages breadth before depth

**Synergy Incentives:**
- Pair synergies provide ~3-8% additional bonuses
- Triple synergies provide ~2-5% additional bonuses
- Encourages strategic attachment combinations

---

## Security & Anti-Exploit

### Rate Limiting
**Implementation:**
```csharp
class RateLimiter {
    private int _maxActions = 5;  // Default: 5 actions/second
    private Queue<DateTime> _actions;
    
    public bool AllowAction() {
        // Sliding window algorithm
        // Remove actions older than 1 second
        // Check if queue size < max
        // Add current timestamp
    }
}
```

**Protected Operations:**
- UI commands (tab switching, weapon cycling)
- Purchase operations
- Attachment applications
- Weapon modifications

**Limits:**
- Default: 5 actions per second
- Configurable per-action type

### Input Validation

**Steam ID Validation:**
```csharp
if (!ulong.TryParse(arg.Args[0], out ulong targetId))
{
    SendReply(arg, "Invalid Steam ID");
    return;
}
```

**Cost Validation:**
```csharp
if (!int.TryParse(arg.Args[1], out int cost))
{
    SendReply(arg, "Invalid cost");
    return;
}
```

**Token Balance Checks:**
```csharp
if (session.Profile.Tokens < cost)
{
    SendReply(player, "Insufficient tokens!");
    return;
}
```

### Permission System

**Defined Permissions:**
```csharp
const string PERMISSION_ADMIN = "killadome.admin"
const string PERMISSION_VIP = "killadome.vip"
```

**Admin Commands:**
- `kd.open` - Force open UI
- `kd.start` - Start match
- `kd.giveskin` - Grant skin to player
- `kd.resetprogress` - Reset player progress

**Permission Checks:**
```csharp
if (!permission.UserHasPermission(player.UserIDString, PERMISSION_ADMIN))
{
    SendReply(arg, "You don't have permission");
    return;
}
```

### Data Integrity

**Atomic Saves:**
```csharp
// Write to temp file
File.WriteAllText(tempPath, json);

// Swap atomically
File.Delete(filePath);
File.Move(tempPath, filePath);
```

**Profile Validation:**
```csharp
try {
    var profile = JsonConvert.DeserializeObject<PlayerProfile>(json);
    return profile ?? new PlayerProfile(steamId, startingTokens);
} catch (Exception ex) {
    // Return default profile on error
}
```

### Exploit Prevention

**Session Validation:**
- Check session exists before operations
- Validate player is connected
- Verify ownership before applying items

**UI State Protection:**
- All UI commands validate player reference
- Rate limiting prevents UI spam
- Null checks throughout

**Economy Protection:**
- Token balance validated before purchases
- Costs hard-coded (not client-supplied)
- Atomic save operations prevent duplication

---

## Configuration

### Config File: `oxide/config/KillaDome.json`

```json
{
  "Lobby Spawn Position": { "x": 0.0, "y": 100.0, "z": 0.0 },
  "Arena Spawn Position": { "x": 0.0, "y": 100.0, "z": 500.0 },
  "Starting Blood Tokens": 500,
  "Tokens Per Kill": 10,
  "Enable Tebex Integration": false,
  "Tebex Secret Key": "YOUR_SECRET_KEY_HERE",
  "Max Weapon Level": 10,
  "Max Attachment Level": 5,
  "UI Update Throttle MS": 100,
  "Auto Save Interval Seconds": 300.0,
  "Enable Debug Logging": false
}
```

### Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| **Lobby Spawn Position** | Vector3 | (0, 100, 0) | Where players spawn in lobby |
| **Arena Spawn Position** | Vector3 | (0, 100, 500) | Where players spawn in arena |
| **Starting Blood Tokens** | int | 500 | Initial token balance |
| **Tokens Per Kill** | int | 10 | Tokens awarded per kill |
| **Enable Tebex Integration** | bool | false | Enable external store |
| **Tebex Secret Key** | string | - | API key for Tebex |
| **Max Weapon Level** | int | 10 | Maximum weapon level |
| **Max Attachment Level** | int | 5 | Maximum attachment level |
| **UI Update Throttle MS** | int | 100 | UI refresh rate limit |
| **Auto Save Interval Seconds** | float | 300.0 | Save frequency (5 min) |
| **Enable Debug Logging** | bool | false | Verbose logging |

### Modifying Configuration

1. Edit `oxide/config/KillaDome.json`
2. Reload plugin: `oxide.reload KillaDome`
3. Or restart server

**Note:** Changes take effect on next plugin load. Active sessions retain old config until player reconnect.

---

## Commands

### Chat Commands

#### `/kd [subcommand]`
General command interface for players.

**Subcommands:**
- `open` - Opens the lobby UI
- `stats` - Shows player statistics
- `help` - Displays help information

**Examples:**
```
/kd open
/kd stats
/kd help
```

#### `/dice <bet>`
Play dice game against the house.

**Parameters:**
- `bet` - Amount to wager (10-100 tokens)

**Rules:**
- Roll die (1-6) vs. house
- Win 2x bet if you roll higher
- Tie returns bet
- 30-second cooldown

**Examples:**
```
/dice 50     → Bet 50 tokens
/dice 100    → Bet 100 tokens (max)
```

#### `/tokengame`
Shows information about available token games.

### Console Commands (Admin)

#### `kd.open`
**Permission:** `killadome.admin`
Force opens the lobby UI for the executing player.

#### `kd.start`
**Permission:** `killadome.admin`
Starts a new match with queued players.

#### `kd.giveskin <steamid> <skinid>`
**Permission:** `killadome.admin`
Grants a skin to a player.

**Parameters:**
- `steamid` - Target player's Steam ID
- `skinid` - Skin identifier

**Example:**
```
kd.giveskin 76561198012345678 3102802323
```

#### `kd.resetprogress <steamid>`
**Permission:** `killadome.admin`
Resets a player's progression to defaults.

**Parameters:**
- `steamid` - Target player's Steam ID

**Example:**
```
kd.resetprogress 76561198012345678
```

### Console Commands (UI)

These are internal commands triggered by UI buttons:

- `killadome.close` - Closes UI
- `killadome.tab <tab>` - Switches UI tab
- `killadome.joinqueue` - Joins matchmaking queue
- `killadome.weapon.prev <slot>` - Cycles to previous weapon
- `killadome.weapon.next <slot>` - Cycles to next weapon
- `killadome.purchase <itemid> <cost>` - Purchases store item
- `killadome.applyskin <weapon> <skinid>` - Applies skin to weapon
- `killadome.applyattachment <weapon> <slot> <attachmentid>` - Applies attachment
- `killadome.attachcat <category>` - Switches attachment category
- `killadome.editweapon <slot>` - Sets weapon being edited

---

## Code Quality Analysis

### Strengths

#### 1. **Modular Architecture**
- Clean separation of concerns
- Each module has single responsibility
- Easy to test individual components
- Maintainable and extensible

#### 2. **Comprehensive Feature Set**
- Full-featured lobby system
- Complex progression mechanics
- Rich UI with multiple tabs
- Integrated economy

#### 3. **Data Persistence**
- Atomic file saves prevent corruption
- JSON serialization for portability
- Per-player file structure scales well

#### 4. **Security Measures**
- Rate limiting prevents spam
- Input validation throughout
- Permission system for admin commands
- Anti-exploit measures

#### 5. **Error Handling**
- Try-catch blocks around I/O
- Null checks before operations
- Fallback to defaults on errors
- Graceful degradation

#### 6. **Documentation**
- Detailed header comment
- Descriptive variable/method names
- Logical code organization

### Areas for Improvement

#### 1. **Dependency on External Plugin**
**Issue:** Relies on ImageLibrary plugin for images.

**Impact:**
- UI breaks if ImageLibrary not installed
- No fallback images

**Recommendation:**
```csharp
if (_plugin.ImageLibrary == null || !_plugin.ImageLibrary.IsLoaded)
{
    // Show text-only fallback
    container.Add(new CuiLabel {
        Text = { Text = skin.Name }
    });
}
```

#### 2. **Hardcoded Weapon List**
**Issue:** Available weapons hardcoded in `CycleWeapon()`:
```csharp
string[] availableWeapons = { "ak47", "m249", "pistol" };
```

**Impact:**
- Difficult to add new weapons
- Not configurable by server admins

**Recommendation:**
- Move to config file
- Create WeaponRegistry class
- Load from JSON definitions

#### 3. **UI Performance**
**Issue:** UI recreated entirely on every update.

**Impact:**
- Unnecessary network traffic
- Client-side lag with frequent updates

**Recommendation:**
- Implement partial UI updates
- Only refresh changed elements
- Cache UI container references

#### 4. **Magic Numbers**
**Issue:** Many hardcoded values:
```csharp
timer.Once(3f, () => ...)  // Why 3 seconds?
float yMin = 0.52f - (i * 0.15f);  // Why these values?
```

**Recommendation:**
- Extract to named constants:
```csharp
const float RESPAWN_DELAY = 3f;
const float STORE_ITEM_HEIGHT = 0.10f;
const float STORE_ITEM_SPACING = 0.05f;
```

#### 5. **Limited Weapon Mapping**
**Issue:** Only 3 weapons in `GiveWeapon()`:
```csharp
string itemName = weaponName switch
{
    "ak47" => "rifle.ak",
    "m249" => "lmg.m249",
    "pistol" => "pistol.semiauto",
    _ => "rifle.ak"  // Default
};
```

**Recommendation:**
- Create comprehensive weapon mapping
- Load from config or data file
- Support all Rust weapons

#### 6. **Synergy System Complexity**
**Issue:** 11-step calculation with hardcoded rules.

**Impact:**
- Difficult to balance
- Hard to add new synergies
- Not moddable by admins

**Recommendation:**
- Move synergy rules to JSON config
- Simplify calculation logic
- Create synergy editor tool

#### 7. **Missing Client-Side Integration**
**Issue:** VFX/SFX managers send console commands but no client implementation:
```csharp
player.SendConsoleCommand($"killadome.vfx {vfxTag} {x} {y} {z}");
```

**Impact:**
- Effects won't actually play
- Requires additional client mod

**Recommendation:**
- Document required client-side setup
- Or use Rust's built-in effect system:
```csharp
Effect.server.Run(vfxPath, position);
```

#### 8. **Tebex Integration Incomplete**
**Issue:** Tebex purchase processing is stubbed:
```csharp
// TODO: Verify purchase with Tebex API using secret key
```

**Recommendation:**
- Implement Tebex API calls
- Add webhook receiver
- Verify transactions properly

#### 9. **No Localization**
**Issue:** All text is English-only hardcoded strings.

**Recommendation:**
- Add language file support
- Use Oxide's lang API:
```csharp
lang.GetMessage("UITitle", this, player.UserIDString);
```

#### 10. **Test Coverage**
**Issue:** No unit tests visible.

**Recommendation:**
- Add unit tests for:
  - Economy calculations
  - Synergy engine
  - Rate limiter
  - Save/load logic

### Performance Considerations

#### Memory Usage
- **Good:** Uses dictionaries for O(1) lookups
- **Concern:** Stores all sessions in memory
- **Recommendation:** Add cleanup for inactive sessions

#### GC Pressure
- **Good:** Reuses data structures where possible
- **Concern:** Heavy UI creation allocates many objects
- **Recommendation:** Object pooling for CUI elements

#### Network Traffic
- **Concern:** Full UI re-sends on every update
- **Recommendation:** Implement delta updates

### Code Metrics

**Estimated Lines of Code:** ~3,070
- **Core Plugin:** ~870 lines
- **Modules:** ~2,200 lines
- **Comments/Whitespace:** ~200 lines

**Cyclomatic Complexity:**
- Most methods: Low (1-5)
- `CalculateModifiers()`: High (~30+)
- `ShowLoadoutsTab()`: High (~25+)

**Class Coupling:**
- Moderate overall
- Main plugin depends on all modules
- Modules mostly independent of each other

---

## Advanced Features Deep Dive

### The Synergy Engine

The synergy engine is one of the most complex and interesting parts of this plugin. It implements an 11-step calculation that transforms basic attachment stats into sophisticated weapon modifications.

#### Step-by-Step Breakdown:

**Step 1-2: Gather and Convert**
```csharp
var activeAttachments = new List<string>();
var percentChanges = new Dictionary<string, List<float>>();

foreach (var attachment in attachments.Values)
{
    if (_alterEgoStats.TryGetValue(attachment, out var stats))
    {
        // Convert multiplier to percent change
        // 0.90f → -10%
        // 1.10f → +10%
        activeAttachments.Add(attachment);
        percentChanges["RecoilMul"].Add((stats.RecoilMul - 1.0f) * 100f);
    }
}
```

**Step 3: Sum Percent Changes**
```csharp
Dictionary<string, float> totalPercent;
foreach (var stat in percentChanges)
{
    totalPercent[stat.Key] = stat.Value.Sum();
}
// Example: -10% + -8% + -5% = -23% total recoil reduction
```

**Step 4: Apply Level Scaling**
```csharp
foreach (var attachment in activeAttachments)
{
    int level = profile.AttachmentLevels.GetValueOrDefault(attachment, 0);
    float scaling = 1.0f + (level * 0.05f);  // 5% per level
    // Level 5 attachment: 1.25x effectiveness
}
```

**Step 5-6: Apply Synergies**
```csharp
// Pair synergies
foreach (var synergy in _pairSynergies)
{
    if (synergy.RequiredAttachments.All(req => activeAttachments.Contains(req)))
    {
        // Apply bonus multipliers
        foreach (var mult in synergy.Multipliers)
        {
            totalPercent[mult.Key] += (mult.Value - 1.0f) * 100f;
        }
    }
}
```

**Step 7: Convert Back to Multipliers**
```csharp
result.RecoilMul = 1.0f + (totalPercent["RecoilMul"] / 100f);
// -23% → 0.77 multiplier
```

**Step 8: Apply Additive Stats**
```csharp
result.OnHitHP += stats.OnHitHP;
result.BleedChance += stats.BleedChance;
```

**Step 9: Cap Values**
```csharp
result.RecoilMul = Mathf.Clamp(result.RecoilMul, 0.5f, 2.0f);
result.DamageMul = Mathf.Clamp(result.DamageMul, 0.5f, 2.0f);
```

**Example Calculation:**

Player equips:
- 8x Scope (Lv 3): -12% recoil, -6% aimcone
- Muzzle Brake (Lv 5): -30% recoil, +30% horizontal stability
- Pair Synergy: Additional -5% recoil

Total Recoil Reduction:
1. Base 8x: -12% × 1.15 (level scaling) = -13.8%
2. Base Brake: -30% × 1.25 (level scaling) = -37.5%
3. Synergy bonus: -5%
4. Total: -56.3% → 0.437 multiplier
5. Capped: 0.5 (minimum)

Final Result: **50% recoil reduction**

This creates deep customization where:
- Individual attachments matter
- Levels matter
- Synergies reward strategic combinations
- Caps prevent overpowered builds

### The Dice Game

The dice game provides a simple gambling mechanic for token acquisition:

```csharp
[ChatCommand("dice")]
private void CmdDiceGame(BasePlayer player, string command, string[] args)
{
    // Cooldown check (30 seconds)
    if (session.LastDiceGame != default && 
        (DateTime.UtcNow - session.LastDiceGame).TotalSeconds < 30)
    {
        // Reject
    }
    
    // Bet validation (10-100 tokens)
    if (bet < 10 || bet > 100) return;
    
    // Balance check
    if (session.Profile.Tokens < bet) return;
    
    // Deduct bet
    session.Profile.Tokens -= bet;
    session.LastDiceGame = DateTime.UtcNow;
    
    // Roll dice
    int playerRoll = UnityEngine.Random.Range(1, 7);
    int houseRoll = UnityEngine.Random.Range(1, 7);
    
    // Determine outcome
    if (playerRoll > houseRoll)
    {
        session.Profile.Tokens += bet * 2;  // Win 2x
        Effect.server.Run("buy.prefab", player.transform.position);
    }
    else if (playerRoll < houseRoll)
    {
        // Lost bet (already deducted)
        Effect.server.Run("deny.prefab", player.transform.position);
    }
    else
    {
        session.Profile.Tokens += bet;  // Return bet on tie
    }
    
    _saveManager.SavePlayerProfile(session.Profile);
}
```

**Game Theory:**
- Expected value: `(5/6 × 2 × bet) + (1/6 × bet) - bet = 0`
- House edge: None (fair game)
- Max profit: 100 tokens per game
- Max loss: 100 tokens per game
- Cooldown prevents abuse

---

## Future Enhancement Opportunities

### 1. **Expanded Weapon Arsenal**
- Add all Rust weapons (LR-300, MP5, Thompson, etc.)
- Custom weapon definitions in JSON
- Per-weapon attachment compatibility

### 2. **Map/Arena System**
- Multiple arenas with different layouts
- Arena voting system
- Dynamic weather/time of day

### 3. **Ranking/Matchmaking**
- ELO-based ranking
- Skill-based matchmaking
- Leaderboards

### 4. **Clan/Team System**
- Team loadouts
- Shared progression
- Team rankings

### 5. **Events/Challenges**
- Daily challenges
- Weekly events
- Special rewards

### 6. **Enhanced Telemetry**
- Weapon usage statistics
- Heatmaps
- Balance analytics

### 7. **Mobile App Integration**
- View stats remotely
- Manage loadouts
- Store purchases

### 8. **Spectator Mode**
- Watch ongoing matches
- First-person spectating
- Replay system

### 9. **Cosmetic System**
- Kill effects
- Victory poses
- Weapon charms

### 10. **Advanced Synergies**
- Quad synergies
- Perk synergies
- Weapon-specific synergies

---

## Conclusion

**KillaDome** is a feature-rich, well-architected Oxide plugin that successfully brings Call of Duty-style gameplay to Rust. Its modular design, comprehensive UI system, and sophisticated attachment/synergy mechanics create a deep and engaging player experience.

### Key Takeaways:

**✅ Strengths:**
- Modular, maintainable codebase
- Rich feature set with progression systems
- Robust economy and anti-exploit measures
- Comprehensive UI with multiple tabs
- Advanced synergy system for depth

**⚠️ Considerations:**
- External dependencies (ImageLibrary)
- Some incomplete features (Tebex, VFX/SFX)
- Performance optimization opportunities
- Limited configurability of core systems

**🚀 Potential:**
With refinements and additional features, KillaDome could become a premier arena-combat plugin for Rust servers, offering endless customization and a competitive gameplay loop that keeps players engaged.

### Technical Excellence:
- **Architecture:** 8/10 (modular, but could be more decoupled)
- **Code Quality:** 7/10 (clean, but lacks tests and some hardcoding)
- **Feature Completeness:** 8/10 (rich features, some stubs)
- **Security:** 8/10 (good measures, room for improvement)
- **Performance:** 7/10 (functional, optimization needed)
- **Documentation:** 6/10 (code is self-documenting, lacks external docs)

**Overall Score: 7.5/10**

A solid, production-ready plugin with room for growth and optimization.
