# LazyWars Backend Architecture Documentation

## Overview
LazyWars is a Solana mobile dApp with a comprehensive backend built on Supabase, featuring game mechanics for OFModels management, resource collection, and player progression.

## Database Schema

### Core Table: `lazywar_profiles`

#### Identity & Basic Info
- `id` (UUID, PK)
- `wallet_address` (TEXT, UNIQUE) - Solana wallet address
- `username` (TEXT, 4-18 chars, alphanumeric + underscore/hyphen)
- `created_at` / `updated_at` (TIMESTAMPTZ)

#### Resources & Game State
- `credits` (INTEGER, ≥0, default: 1000) - Primary currency
- `turns` (INTEGER, 0-200, default: 2) - Action points
- `ofmodels` (INTEGER, ≥0, default: 1) - Primary income generators
- `minions` (INTEGER, ≥0, default: 0) - Support units
- `city` (TEXT) - Player location (LA, NY, Miami, Dubai, Hong Kong)

#### Happiness System
- `ofmodels_happiness` (INTEGER, 0-100, default: 100)
- `minions_happiness` (INTEGER, 0-100, default: 100)

#### Progression System
- `exp` (INTEGER, ≥0, default: 0) - Experience points
- `level` (INTEGER, 1-10, default: 1) - Auto-calculated from EXP
- `title` (TEXT, default: 'Stake Slacker') - Level-based title (Stake Slacker → Solstice Slothlord)
- `lazy_score` (INTEGER, ≥0, default: 100) - Auto-calculated ranking score
- `lazy_rank` (INTEGER, ≥1, default: 1) - Player ranking

#### OFModels Mechanics
- `payout_pct` (INTEGER, 0-99, default: 40) - Income percentage to models
- `is_fat` (BOOLEAN, default: false) - Fatness penalty (-20% happiness)
- `last_income_at` (TIMESTAMPTZ) - Last hourly income calculation
- `last_happiness_decay_at` (TIMESTAMPTZ) - Last happiness decay application

#### Farm System
- `farm_hydro_level` (INTEGER, ≥0, default: 0) - Hydroponic system level
- `farm_led_level` (INTEGER, ≥0, default: 0) - LED lighting system level

#### Inventory (Shop Items)
- `nft_costumes` (INTEGER, ≥0, default: 0) - Protective items
- `gym_memberships` (INTEGER, ≥0, default: 0) - Fitness items
- `moutai` (INTEGER, ≥0, default: 0) - Luxury alcohol
- `weed` (INTEGER, ≥0, default: 0) - Consumable/tradeable
- `sig_p365` (INTEGER, ≥0, default: 0) - Weapon type 1
- `beretta_1301` (INTEGER, ≥0, default: 0) - Weapon type 2
- `vector_smg` (INTEGER, ≥0, default: 0) - Weapon type 3
- `boombox` (INTEGER, ≥0, default: 0) - Entertainment item
- `cybertruck` (INTEGER, ≥0, default: 0) - Vehicle

#### **ADDED COLUMNS** ✅
- `last_turns_regen_at` (TIMESTAMPTZ) - Turn regeneration tracking
- `last_minions_decay_at` (TIMESTAMPTZ) - Minions happiness decay tracking  
- `last_desertion_check_at` (TIMESTAMPTZ) - Desertion risk processing tracking

### Support Table: `lazywars_purchase_history`
- `id` (UUID, PK)
- `wallet_address` (TEXT, FK to profiles)
- `item_name` (TEXT) - Name of purchased item
- `quantity` (INTEGER, >0) - Amount purchased
- `unit_cost` (INTEGER, >0) - Cost per unit
- `total_cost` (INTEGER, >0) - Total transaction cost
- `purchased_at` (TIMESTAMPTZ) - Purchase timestamp

## Game Mechanics

### 1. OFModels Income System
**Constants:**
- Base income per model: 10 credits/hour
- Default payout: 40%
- Happiness threshold: 70% (below = risk of desertion)

**Factors:**
- Payout percentage affects income (higher payout = lower income)
- Happiness affects income multiplier
- NFT costumes provide happiness boost (+10 per costume, max +50)
- Minion ratio provides happiness boost (+5 per ratio point, max +25)
- Fatness penalty reduces happiness by 20%

### 2. Complete Happiness System ✅ IMPLEMENTED
**OFModels Happiness (0-100%):**
- **Formula:** 50 + payout_boost + nft_costumes_boost + minion_ratio_boost - fatness_penalty
- **Payout boost:** (payout_pct - 40) / 2 (60% payout = +10% happiness)
- **NFT Costumes boost:** num_costumes / 10 (max +50%; prevents fatness penalty)
- **Minions ratio boost:** 5 × (minions / ofmodels) (max +25% at 5:1 ratio)
- **Fatness penalty:** -20% (only if no NFT costumes protection)
- **Decay conditions:** -5% hourly if minions < ofmodels AND nft_costumes = 0

**Minions Happiness (0-100%):**
- **Formula:** 100 - moutai_deficit - weapons_deficit
- **Moutai deficit:** -1% per missing moutai bottle (if minions > moutai)
- **Weapons deficit:** -1% per missing weapon (total weapons = sig_p365 + beretta_1301 + vector_smg)
- **Decay conditions:** -5% hourly if moutai < minions OR weapons_total < minions

**Desertion Mechanics:**
- **Trigger:** Happiness < 70% for either system
- **Risk:** 10% chance per hour (automated cron) + 10% chance per grow action
- **Loss:** 1-5 units randomly (limited by current amount)
- **Frequency:** Checked hourly via `happiness-decay` Edge Function

**Protection & Recovery:**
- **OFModels protection:** Maintain minions >= ofmodels OR own NFT costumes
- **Minions protection:** Maintain moutai >= minions AND weapons >= minions
- **Fatness cure:** Costs 1 gym_membership, removes -20% OFModels penalty
- **Auto-updates:** Happiness recalculated in scout/grow/cure actions

### 3. Turns System
- **Maximum:** 100 turns (schema constraint updated from 200)
- **Regeneration:** +2 turns every 10 minutes (cron job)
- **Usage:** Scouting (yield affected by happiness), Growing (desertion risk)
- **Tracking:** `last_turns_regen_at` timestamp for efficiency

### 4. Experience & Leveling System ✅ IMPLEMENTED
**Level Thresholds & Titles:**
- Level 1: "Stake Slacker" (0 EXP)
- Level 2: "Node Napper" (100 EXP)
- Level 3: "Block Besieger" (300 EXP)
- Level 4: "Chain Conqueror" (600 EXP)
- Level 5: "Ledger Lounger" (1,000 EXP)
- Level 6: "Consensus Couch" (2,000 EXP)
- Level 7: "Proof Pillager" (4,000 EXP)
- Level 8: "Epoch Idler" (8,000 EXP)
- Level 9: "Genesis Guerilla" (15,000 EXP)
- Level 10: "Solstice Slothlord" (69,420 EXP)

**EXP Sources:**
- Scouting: 1 EXP per turn spent
- Growing: 2 EXP per turn spent (higher reward)
- Future actions: Combat, quests, achievements

**Level Up Mechanics:**
- Automatic level and title updates via `add_exp()` function
- Called automatically in scout/grow actions
- Level up notifications included in action responses
- Progress tracking available via `get_exp_progress()` function

### 5. Lazy Score Calculation
```sql
lazy_score = credits + (ofmodels * 2000) + (minions * 1000)
```

## Edge Functions (Supabase)

### Active Cron Jobs
1. **`happiness-decay`** - ✅ Complete happiness processing every hour
   - Processes both OFModels and Minions happiness decay
   - Checks desertion risk for both systems (10% chance if <70% happiness)
   - Applies losses (1-5 units) when desertion occurs
   - Updates tracking timestamps for efficiency
2. **`hourly-income`** - Calculates OFModels income every hour
3. **`turns-regeneration`** - Regenerates turns every 10 minutes

### Interactive Functions
1. **`purchase`** - Handles shop item purchases
2. **`scout`** - ✅ Scouting with complete happiness integration
   - Yield multipliers based on average happiness
   - 50% reduction if average happiness < 70%
   - Desertion risk during scouting
   - Real-time happiness calculation and updates
   - **✅ Automatic EXP and level up processing (1 EXP per turn)**
3. **`grow`** - ✅ Weed growing with happiness integration  
   - Minion assistance bonus (10% per minion)
   - Happiness multiplier for minions productivity
   - Desertion risk during growing (10% chance if unhappy)
   - Farm equipment bonuses
   - **✅ Automatic EXP and level up processing (2 EXP per turn)**
4. **`upgrade-farm`** - Farm equipment upgrades
5. **`cure-fatness`** - ✅ Remove fatness penalty using gym membership
   - Costs 1 gym_membership
   - Removes `is_fat` status
   - Auto-recalculates happiness after cure
6. **`add-exp`** - ✅ Manual EXP addition with level up checks
   - Admin/special event EXP rewards
   - Automatic level and title updates
   - Level up notifications

## SQL Functions

### Core Functions
- `calculate_lazy_score(credits, ofmodels, minions)` - Score calculation
- `get_player_rank(wallet_address)` - Player ranking
- `calculate_level_from_exp(exp)` - Level from experience
- `regenerate_turns(cutoff_time)` - Bulk turns regeneration

### Game Mechanics Functions
- `scout_models(turns_spent, wallet_address)` - ✅ Scouting with complete happiness integration
  - Happiness multipliers for yield calculation
  - Risk assessment based on happiness levels
  - Real-time happiness updates
- `grow_weed(turns_spent, wallet_address)` - ✅ Weed growing with happiness integration
  - Minion assistance bonuses
  - Desertion risk during growing
  - Farm equipment bonuses
- `upgrade_farm(upgrade_type, wallet_address)` - Farm upgrades

### ✅ Complete Happiness System Functions
- `calculate_complete_happiness(ofmodels, minions, payout_pct, nft_costumes, is_fat, moutai, weapons_total)` - Core happiness calculation for both systems
- `update_happiness(wallet_address)` - Update and store happiness for a player
- `check_desertion_risk(ofmodels_happiness, minions_happiness, ofmodels, minions)` - Check desertion risk and calculate losses
- `cure_fatness(wallet_address)` - Cure fatness using gym membership (costs 1, removes penalty)

### ✅ EXP and Leveling Functions
- `add_exp(wallet_address, exp_amount)` - Add EXP and automatically check for level up with title update
- `get_exp_progress(wallet_address)` - Get detailed level progress information for frontend
- `get_title_from_level(level)` - Get player title based on level
- `calculate_level_from_exp(exp)` - Calculate level from EXP amount (existing function)

### Analytics Functions
- `get_total_credits_spent(wallet_address)` - Purchase analytics
- `get_player_purchase_stats(wallet_address)` - Detailed purchase stats

## Database Triggers

### Auto-Update Triggers
- `update_updated_at_column()` - Updates timestamp on row changes
- `update_lazy_score()` - Recalculates score when credits/ofmodels/minions change
- `update_level_on_exp_change()` - Updates level when EXP changes
- `update_level_on_insert()` - Sets correct level on profile creation

## Constants & Configurations

### OFModels & Happiness Constants
```typescript
OFMODELS_CONSTANTS = {
  DEFAULT_PAYOUT_PCT: 40,
  BASE_INCOME_PER_MODEL: 10,
  BASE_HAPPINESS: 50,
  HAPPINESS_DECAY_RATE: 5,
  DESERTION_THRESHOLD: 70,
  DESERTION_CHANCE: 0.10, // 10% hourly
  MAX_DESERTION_LOSS: 5,
  NFT_COSTUME_BOOST_FACTOR: 10,
  NFT_COSTUME_MAX_BOOST: 50,
  MINION_RATIO_BOOST: 5,
  MINION_RATIO_MAX_BOOST: 25,
  FATNESS_PENALTY: 20,
  TURNS_REGEN_AMOUNT: 2,
  TURNS_REGEN_INTERVAL_MINUTES: 10,
  MAX_TURNS: 100,
}

MINIONS_CONSTANTS = {
  BASE_HAPPINESS: 100,
  HAPPINESS_DECAY_RATE: 5,
  DESERTION_THRESHOLD: 70,
  DESERTION_CHANCE: 0.10, // 10% hourly
  MAX_DESERTION_LOSS: 5,
  MOUTAI_DEFICIT_PENALTY: 1, // -1% per missing
  WEAPONS_DEFICIT_PENALTY: 1, // -1% per missing
}
```

## Implementation Status

### ✅ Fully Implemented
- **✅ Complete happiness system for both OFModels and Minions**
- **✅ Desertion risk mechanics with automated hourly processing**
- **✅ Fatness penalty system with gym membership cure**
- **✅ Happiness integration in scout and grow functions**
- **✅ Real-time happiness calculation and updates**
- **✅ Automated happiness decay via cron jobs**
- **✅ EXP and Titles system with automatic level ups**
- **✅ Level up integration in game actions (scout/grow)**
- **✅ Progress tracking and title management**
- **✅ OFModels income generation with happiness mechanics**
- **✅ Turns regeneration system with proper tracking**
- **✅ Scouting mechanics with happiness-based yields and risk**
- **✅ Weed growing with minion assistance and happiness effects**
- **✅ Purchase system with analytics**
- **✅ Lazy score calculation and rankings**

### ⚠️ Partially Implemented
- Attack/defense mechanics (basic framework exists)

### ❌ Not Implemented
- Advanced combat mechanics with happiness effects
- Social features and guild systems
- Achievement and quest systems

## File Structure
```
supabase/
├── schema.sql                           # Core database schema
├── add-ofmodels-mechanics.sql          # OFModels system
├── add-level-system.sql                # EXP/Level system
├── add-inventory-columns.sql           # Shop items
├── add-scout-mechanics.sql             # Scouting system
├── add-grow-mechanics.sql              # Farming system
├── add-turns-regeneration-function.sql # Turns system
├── add-purchase-analytics.sql          # Purchase tracking
├── add-complete-happiness-system.sql   # ✅ Complete happiness mechanics
├── update-scout-with-happiness.sql     # ✅ Updated scout integration
├── update-grow-with-happiness.sql      # ✅ Updated grow integration
├── add-exp-title-system.sql            # ✅ EXP and titles system
├── update-scout-grow-with-exp-system.sql # ✅ Scout/grow with automatic level ups
├── functions/
│   ├── hourly-income/index.ts          # Income generation
│   ├── happiness-decay/index.ts        # ✅ Complete happiness processing
│   ├── turns-regeneration/index.ts     # Turn regeneration
│   ├── purchase/index.ts               # Shop purchases
│   ├── scout/index.ts                  # Resource scouting
│   ├── grow/index.ts                   # Weed growing
│   ├── upgrade-farm/index.ts           # Farm upgrades
│   ├── cure-fatness/index.ts           # ✅ Fatness cure function
│   ├── add-exp/index.ts                # ✅ Manual EXP addition with level ups
│   └── _shared/ofmodels-mechanics.ts   # Shared constants
```

## Next Implementation Priorities
1. ✅ **COMPLETED:** Complete happiness system implementation
2. ✅ **COMPLETED:** Minions happiness mechanics with decay and desertion  
3. ✅ **COMPLETED:** Happiness integration in game actions (scout/grow)
4. ✅ **COMPLETED:** Fatness cure system with gym memberships
5. ✅ **COMPLETED:** Automated happiness processing via cron jobs
6. ✅ **COMPLETED:** EXP and Titles system with automatic level ups
7. Advanced combat mechanics with happiness effects on battle outcomes
8. Social features and player interactions
9. Achievement and quest systems with EXP rewards
10. Leaderboards with title-based rankings
