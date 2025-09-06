# lazywars-gamepaper.md

## Overview

LazyWars is a blockchain-based idle gaming dApp built on the Solana network.

It was built publicly on X.

**Public Development Thread:** https://x.com/adidogCEO/status/1949623428618580463

---

## Full Blockchain Migration Plans

### Vision: Fully Onchain Data That's Forkable

LazyWars is architected with a long-term vision to become a fully onchain game, transitioning from the current hybrid model to a complete blockchain-based system. This migration will ensure true decentralization, data sovereignty, and enable community-driven game evolution.

### Micro-Transaction Architecture
- Every game action will generate a micro-transaction on Solana
- Each player assigned a dedicated keypair for automated transaction signing
- Complete game state stored in transaction memos
- Chronological transaction history serves as complete game history

### Decentralized Game Evolution
- Complete game history stored onchain enables community forks
- Different rulesets can be applied to the same transaction history
- Community governance for game mechanics changes
- Multiple frontend implementations possible (web, mobile, CLI)

### Implementation Strategy

**Phase 1: Parallel Transaction Logging**
- Maintain current Supabase system for performance
- Add parallel micro-transaction creation for every action
- Memo data matches current database schema
- Validate onchain data consistency with database

**Phase 2: Hybrid State Management**
- Primary game state stored onchain
- Database used for caching and performance optimization
- Real-time updates from blockchain events
- Gradual reduction of database dependency

**Phase 3: Full Onchain Migration**
- Complete removal of centralized database dependency
- All game state derived from blockchain transactions
- Community-run infrastructure nodes

*This represents the ultimate vision: a truly decentralized, community-owned, and infinitely extensible gaming platform where the data is the game, and the game is the data.*

---

## Current System Architecture

### Technology Stack

| Component | Technology |
|-----------|------------|
| **Database** | PostgreSQL (via Supabase) |
| **API Layer** | Supabase Edge Functions (Deno-based serverless) |
| **Real-time** | Supabase Realtime subscriptions |
| **Authentication** | Solana wallet-based authentication |
| **Scheduled Tasks** | Supabase Cron for automated game mechanics |

## Database Schema

### Core Data Structure

| Table Category | Purpose |
|----------------|---------|
| **Player Profiles** | Central player data and game state |
| **Empire System** | Guild functionality and management |
| **Combat & Activity Logs** | Battle history and analytics |

#### Player Profiles
*Central player data containing:*
- **Identity & Authentication**: Unique identifiers and wallet addresses
- **Game Resources**: Credits, turns, units (OFmodels and minions)
- **Progression System**: Experience points, levels, rankings, and titles
- **Game Mechanics State**: Happiness levels, status effects, location
- **Inventory System**: Various items including NFT costumes, equipment, and consumables
- **Empire Integration**: Empire membership and bonus tracking

#### Empire System
*Guild functionality with:*
- **Empire Management**: Name, leadership, treasury, and member limits (20 max)
- **Financial System**: Dues collection, shared treasury, and member contributions
- **Combat Bonuses**: Offensive and defensive minion systems
- **Recruitment**: Invitation and join request workflows

#### Combat & Activity Logs
*Comprehensive tracking including:*
- **Combat History**: Complete battle logs with outcomes and resource transfers
- **Purchase Tracking**: Transaction history for economic analysis  
- **Activity Analytics**: Turn spending and performance metrics

---

## Game Mechanics Architecture

### 1. 🎮 Panel System

**Draggable Terminal Windows**
- Each game function is contained in a draggable panel
- Panels can be collapsed, resized, and repositioned
- Layout state persists in localStorage
- Terminal aesthetic with matrix-green theme

| Panel | Function |
|-------|----------|
| **Dashboard** | Core game stats, scout/grow actions |
| **Shop** | Buy/sell items and equipment |
| **Attack** | Combat system with 5 attack types |
| **Empire** | Guild management and recruitment |
| **Profile** | Player statistics and progression |
| **Activity Log** | Real-time game events |

### 2. 💰 Income Generation System

**Hourly Income Calculation**
```
Base Income = ofmodels × 10 credits/hour
Happiness Modifier = ofmodels_happiness / 100
Payout Adjustment = Based on payout percentage setting
Final Income = Base × Happiness × Payout Adjustment
```

**Automated Processing**
- ⏰ Cron job runs hourly income calculation
- 🔧 Database functions maintain happiness levels
- ⚠️ Desertion mechanics for low happiness units

### 3. 😊 Happiness System

**Factors Affecting Happiness**

| Factor | Effect |
|--------|--------|
| Payout percentage | Higher payout = happier units |
| NFT costumes | Happiness boost |
| Minion-to-model ratio | Protection factor |
| Status effects | Fatness penalty |
| Time-based decay | Gradual reduction |

### 4. ⚔️ Combat System

**Attack Types**

| # | Attack Type | Description |
|---|-------------|-------------|
| 1 | **Drive-By Shooting** | Uses cybertrucks and guns for intel gathering |
| 2 | **Home Invasion** | Steals credits and weed from targets |
| 3 | **Jack Cybertrucks** | Steals enemy vehicles |
| 4 | **Smokeup Models** | Inflicts fatness status with weed |
| 5 | **Steal OFmodels** | Uses weed bribes to steal units |

**Combat Resolution**
- 🎲 Attack/Defense calculation based on units and equipment
- 📊 Success probability determination with RNG outcome
- 💸 Resource transfer and unit losses
- 😔 Happiness impact on both parties
- ⭐ Experience point rewards

**Combat Requirements**
- ⚡ Minimum 2 turns required for attacks
- 🎯 Target must be within 50-200% of attacker's lazy score
- 🛠️ Specific resource requirements per attack type

### 5. ⚡ Turns System

**Turn Management**
- 📊 **Maximum**: 200 turns per player
- 🔄 **Regeneration**: +2 turns every 10 minutes
- 💸 **Activities consume turns**: scouting (1 turn), growing (3+ turns), attacks (2 turns)
- 😊 **Growth activities** affected by happiness and upgrade levels

**Turn Activities**

| Activity | Cost | Description |
|----------|------|-------------|
| **Scout** | 1 turn | Recruit new minions and OFmodels |
| **Grow**  | 1 turn | Generate weed and gain experience |
| **Attack** | 2 turns | Combat with other players |

### 6. 🏰 Empire System

**Empire Economics**
- 💰 Member dues based on income percentage (0-50%)
- 🏦 Shared treasury for empire purchases
- 💪 Collective bonuses (attack/defense modifiers)
- 📈 Empire scoring based on member performance

**Empire Features**

| Feature | Description |
|---------|-------------|
| **Max Members** | 20 members per empire |
| **Leadership** | Roles with permissions |
| **Minion Purchases** | Offensive/Defensive upgrades |
| **Recruitment** | Join request and invitation systems |
| **Competition** | Empire vs empire rankings |

**Leadership Requirements**
- 🎖️ Must be Level 3+ (Block Besieger) to create empires
- 👑 Leaders can manage dues, kick members, purchase bonuses
- 🔄 Leadership transfer system for succession

### 7. 🛒 Shop System

**Item Categories**

| Category | Items |
|----------|-------|
| **Consumables** | Moutai, Weed |
| **Equipment** | Various firearms (Sig P365, Beretta 1301, Vector SMG) |
| **Vehicles** | Cybertrucks for combat |
| **Upgrades** | NFT Costumes, Gym Memberships, Boomboxes |
| **Special** | Rocket Launchers, future lootbox system |

**Buy/Sell System**
- 💱 Dynamic pricing for all items
- 📉 Sell-back at reduced prices
- 📦 Inventory tracking per player

---

## Game Balance & Economics

### 💰 Resource Economy
- 💎 **Credits** as primary currency
- ⚡ **Turn-based action economy** limiting spam
- 😊 **Happiness system** creating strategic depth
- 🏰 **Empire dues** creating group economic dynamics

### 📈 Progression System
- 🎖️ **Level-based title progression** (Stake Slacker → Block Besieger → etc.)
- ⭐ **Experience gain** through grow activities and combat
- 🏆 **Lazy score ranking system** for competitive play

### ⚔️ Combat Balance
- 🎯 **Range-based targeting** (50-200% of lazy score)
- 🛠️ **Resource requirements** for different attack types
- ⚖️ **Risk/reward mechanics** with unit losses
- 😔 **Happiness penalties** for failed attacks

---

## Summary

The future blockchain migration will ensure long-term decentralization and community ownership, making LazyWars a truly community-owned gaming platform where **the data is the game, and the game is the data**.
