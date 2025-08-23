# LazyWars Backend System

## Overview

LazyWars is a blockchain-based idle gaming dApp built on the Solana network with a comprehensive backend system powered by Supabase. This document provides developers with a clear understanding of the backend architecture, game mechanics, and system design patterns.

It was built publicly using AI agents and modern web technologies, featuring a unique terminal-style UI aesthetic with draggable panels for each game function.

Public X Thread:
https://x.com/adidogCEO/status/1949623428618580463

## Architecture Overview

### Technology Stack
- **Database**: PostgreSQL (via Supabase)
- **API Layer**: Supabase Edge Functions (Deno-based serverless functions)  
- **Real-time**: Supabase Realtime subscriptions
- **Authentication**: Solana wallet-based authentication
- **Scheduled Tasks**: Supabase Cron for automated game mechanics
- **Frontend**: Next.js 14 App Router with TypeScript

### Core Design Principles
- **Panel-based architecture** with draggable, resizable terminal windows
- **Event-driven architecture** with database functions for game mechanics
- **Atomic transactions** for all game state changes
- **Row-level security** for data access control
- **Terminal UI aesthetic** with matrix-green theme throughout
- **Comprehensive audit trails** for all player actions

## Database Schema

### Core Tables

The game utilizes several core database tables for player management, empire systems, and game mechanics. Due to security considerations, detailed schema information is not publicly documented. However, the general structure includes:

#### Player Profiles
Central player data containing:
- **Identity & Authentication**: Unique identifiers and wallet addresses
- **Game Resources**: Credits, turns, units (OFmodels and minions)
- **Progression System**: Experience points, levels, rankings, and titles
- **Game Mechanics State**: Happiness levels, status effects, location
- **Inventory System**: Various items including NFT costumes, equipment, and consumables
- **Empire Integration**: Empire membership and bonus tracking

#### Empire System
Guild functionality with:
- **Empire Management**: Name, leadership, treasury, and member limits (20 max)
- **Financial System**: Dues collection, shared treasury, and member contributions
- **Combat Bonuses**: Offensive and defensive minion systems
- **Recruitment**: Invitation and join request workflows

#### Combat & Activity Logs
- **Combat History**: Complete battle logs with outcomes and resource transfers
- **Purchase Tracking**: Transaction history for economic analysis  
- **Activity Analytics**: Turn spending and performance metrics

## Game Mechanics Architecture

### 1. Panel System
**Draggable Terminal Windows**
- Each game function is contained in a draggable panel
- Panels can be collapsed, resized, and repositioned
- Layout state persists in localStorage
- Terminal aesthetic with matrix-green theme

**Available Panels**:
- **Dashboard**: Core game stats, scout/grow actions
- **Shop**: Buy/sell items and equipment 
- **Attack**: Combat system with 5 attack types
- **Empire**: Guild management and recruitment
- **Profile**: Player statistics and progression
- **Activity Log**: Real-time game events

### 2. Income Generation System

**Hourly Income Calculation**
```
Base Income = ofmodels × 10 credits/hour
Happiness Modifier = ofmodels_happiness / 100
Payout Adjustment = Based on payout percentage setting
Final Income = Base × Happiness × Payout Adjustment
```

**Automated Processing**
- Cron job runs hourly income calculation
- Database functions maintain happiness levels
- Desertion mechanics for low happiness units

### 3. Happiness System

**Factors Affecting Happiness**
- Payout percentage (higher payout = happier units)
- NFT costumes (happiness boost)
- Minion-to-model ratio (protection factor)
- Status effects (fatness penalty)
- Time-based decay (gradual reduction)

### 4. Combat System

**Attack Types**
1. **Drive-By Shooting**: Uses cybertrucks and guns for intel gathering
2. **Home Invasion**: Steals credits and weed from targets
3. **Jack Cybertrucks**: Steals enemy vehicles
4. **Smokeup Models**: Inflicts fatness status with weed
5. **Steal OFmodels**: Uses weed bribes to steal units

**Combat Resolution**
- Attack/Defense calculation based on units and equipment
- Success probability determination with RNG outcome
- Resource transfer and unit losses
- Happiness impact on both parties
- Experience point rewards

**Combat Requirements**
- Minimum 2 turns required for attacks
- Target must be within 50-200% of attacker's lazy score
- Specific resource requirements per attack type

### 5. Turns System

**Turn Management**
- Maximum: 200 turns per player
- Regeneration: +2 turns every 10 minutes
- Activities consume turns (scouting: 1 turn, growing: 3+ turns, attacks: 2 turns)
- Growth activities affected by happiness and upgrade levels

**Turn Activities**
- **Scout**: Recruit new minions and OFmodels
- **Grow**: Generate weed and gain experience
- **Attack**: Combat with other players

### 6. Empire System

**Empire Economics**
- Member dues based on income percentage (0-50%)
- Shared treasury for empire purchases
- Collective bonuses (attack/defense modifiers)
- Empire scoring based on member performance

**Empire Features**
- Maximum 20 members per empire
- Leadership roles with permissions
- Offensive/Defensive minion purchases
- Join request and invitation systems
- Empire vs empire competition through rankings

**Leadership Requirements**
- Must be Level 3+ (Block Besieger) to create empires
- Leaders can manage dues, kick members, purchase bonuses
- Leadership transfer system for succession

### 7. Shop System

**Item Categories**
- **Consumables**: Moutai, Weed
- **Equipment**: Various firearms (Sig P365, Beretta 1301, Vector SMG)
- **Vehicles**: Cybertrucks for combat
- **Upgrades**: NFT Costumes, Gym Memberships, Boomboxes
- **Special**: Rocket Launchers, future lootbox system

**Buy/Sell System**
- Dynamic pricing for all items
- Sell-back at reduced prices
- Inventory tracking per player

## Edge Functions Architecture

### Function Categories

**Game Action Functions**
- `scout/`: Minion and OFmodel recruitment
- `grow/`: Weed production and experience gain
- `attack/`: Combat execution with outcome calculation
- `purchase/`: Shop item transactions

**Empire Functions**  
- `create-empire/`: New empire establishment
- `empire-join/`: Membership management
- `empire-invite/`: Recruitment workflows
- `empire-purchase-minions/`: Empire-level purchases

**Automated Functions**
- `hourly-income/`: Scheduled income processing
- `turns-regeneration/`: Turn point restoration
- `happiness-decay/`: Happiness degradation over time

**Utility Functions**
- Profile management and updates
- Experience point calculations
- Status effect management

### Function Design Patterns

**Input Validation**
- Comprehensive parameter checking
- Range validation for numerical inputs
- Player state verification

**Transaction Safety**
- Database transactions for atomic operations
- Rollback on failure conditions
- Consistent error handling

## Integration Points

### Solana Blockchain
- Wallet authentication via @solana/wallet-adapter-react
- Message signing for secure operations
- Future: Complete on-chain migration for data sovereignty

### Panel-Based Frontend
- Next.js 14 with TypeScript
- Draggable panels using custom hooks
- Terminal UI with matrix-green theme
- Radix UI + shadcn/ui components

## Game Balance & Economics

### Resource Economy
- Credits as primary currency
- Turn-based action economy limiting spam
- Happiness system creating strategic depth
- Empire dues creating group economic dynamics

### Progression System
- Level-based title progression (Stake Slacker → Block Besieger → etc.)
- Experience gain through grow activities and combat
- Lazy score ranking system for competitive play

### Combat Balance
- Range-based targeting (50-200% of lazy score)
- Resource requirements for different attack types
- Risk/reward mechanics with unit losses
- Happiness penalties for failed attacks

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
1. **Phase 1**: Parallel transaction logging alongside current Supabase system
2. **Phase 2**: Hybrid state management with primary data onchain
3. **Phase 3**: Full onchain migration with community-run infrastructure

This represents the ultimate vision: a truly decentralized, community-owned, and infinitely extensible gaming platform where the data is the game, and the game is the data.

## Conclusion

This backend system provides a robust, scalable foundation for the LazyWars gaming experience. The panel-based architecture creates an engaging terminal-style interface while the comprehensive backend handles complex game mechanics, empire systems, and player progression. The future blockchain migration will ensure long-term decentralization and community ownership.
