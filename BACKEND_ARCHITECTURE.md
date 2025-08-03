# LazyWars Backend Architecture Documentation

## Overview

LazyWars is a Solana mobile dApp with a comprehensive backend built on Supabase, featuring game mechanics for OFModels ma### 4. Turns System
- **Maximum:** 200 turns (schema constraint verified and deployed ✅)
- **Regeneration:** +2 turns every 10 minutes (cron job)
- **Usage:** Scouting (yield affected by happiness), Growing (desertion risk)
- **Tracking:** `last_turns_regen_at` timestamp for efficiency
- **Analytics:** ✅ All turn spending tracked in `lazywars_turn_spending_history`
- **Migration:** `fix-turns-constraint-200.sql` ensures consistent 200 limit across all environmentsnt, resource collection, and player progression.

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
- `grow_upgrade_level` (INTEGER, 1-10, default: 1) - Grow facility upgrade level
- `empire_id` (TEXT, FK to empires) - Reference to current Empire
- `empire_attack_bonus` (INTEGER, ≥0, default: 0) - Combat bonus from Empire minions
- `empire_defense_bonus` (INTEGER, ≥0, default: 0) - Combat bonus from Empire minions
- `last_dues_payment_at` (TIMESTAMPTZ) - Last dues payment timestamp for delta tracking
- `credits_at_last_dues` (INTEGER, ≥0, default: 0) - Credits baseline for delta dues calculation
- `total_dues_paid` (INTEGER, ≥0, default: 0) - Lifetime dues paid to Empire

### Support Table: `lazywars_purchase_history`
- `id` (UUID, PK)
- `wallet_address` (TEXT, FK to profiles)
- `item_name` (TEXT) - Name of purchased item (includes "Grow Upgrade: [Name]")
- `quantity` (INTEGER, >0) - Amount purchased
- `unit_cost` (INTEGER, >0) - Cost per unit
- `total_cost` (INTEGER, >0) - Total transaction cost
- `purchased_at` (TIMESTAMPTZ) - Purchase timestamp

### Analytics Table: `lazywars_turn_spending_history` ✅
- `id` (UUID, PK)
- `wallet_address` (TEXT, FK to profiles) - Player identifier
- `username` (TEXT) - Player username (for analytics)
- `action_type` (TEXT) - Activity type ('scout' | 'grow')
- `turns_spent` (INTEGER, >0) - Number of turns used
- `yield_ofmodels` (INTEGER, default: 0) - OFModels gained (scout only)
- `yield_minions` (INTEGER, default: 0) - Minions gained (scout only)
- `yield_weed` (INTEGER, default: 0) - Weed gained (grow only)
- `spent_at` (TIMESTAMPTZ) - Activity timestamp

### Empire System Tables ✅

#### `lazywars_empires`
- `id` (TEXT, PK) - Empire identifier
- `name` (TEXT, 3-30 chars, UNIQUE) - Empire name
- `leader_wallet_address` (TEXT, FK to profiles) - Empire leader
- `leader_username` (TEXT) - Leader username from profiles join (view only)
- `description` (TEXT, optional, max 200 chars) - Empire description
- `dues_rate` (INTEGER, 0-50, default: 0) - Member dues percentage
- `treasury_credits` (INTEGER, ≥0, default: 0) - Shared treasury
- `offensive_minions` (INTEGER, ≥0, default: 0) - Attack bonuses (renamed from thugs)
- `defensive_minions` (INTEGER, ≥0, default: 0) - Defense bonuses (renamed from thugs)
- `empire_score` (INTEGER, ≥0, default: 0) - Sum of member lazy_scores
- `member_count` (INTEGER, ≥1, default: 1) - Current member count
- `total_members_lifetime` (INTEGER, ≥1, default: 1) - Historical member count
- `created_at` / `updated_at` (TIMESTAMPTZ)

#### `lazywars_empire_members`
- `id` (UUID, PK)
- `empire_id` (TEXT, FK to empires)
- `wallet_address` (TEXT, FK to profiles)
- `role` ('leader' | 'member', default: 'member')
- `joined_at` (TIMESTAMPTZ)
- `total_dues_paid` (INTEGER, ≥0, default: 0) - Lifetime dues contributions
- `contribution_score` (INTEGER, ≥0, default: 0) - Member contribution tracking

#### `lazywars_empire_invitations`
- `id` (UUID, PK)
- `empire_id` (TEXT, FK to empires)
- `wallet_address` (TEXT, FK to profiles) - Invited player
- `invited_by` (TEXT, FK to profiles) - Inviting member
- `status` ('pending' | 'accepted' | 'declined', default: 'pending')
- `invited_at` (TIMESTAMPTZ)
- `expires_at` (TIMESTAMPTZ) - 7-day expiration

#### `lazywars_empire_join_requests` ✅ NEW
- `id` (UUID, PK)
- `empire_id` (TEXT, FK to empires) - Target empire
- `requester_wallet_address` (TEXT, FK to profiles) - Requesting player
- `requester_username` (TEXT) - Requester username
- `status` ('pending' | 'approved' | 'denied', default: 'pending') - Request status
- `created_at` (TIMESTAMPTZ) - Request timestamp
- `updated_at` (TIMESTAMPTZ) - Last update timestamp
- **Constraints:**
  - Unique constraint on `empire_id + requester_wallet_address` for pending requests
  - Global unique constraint on `requester_wallet_address` for pending requests (one request per player)
  - Approved/denied requests are deleted from table after processing

#### `lazywars_empire_board`
- `id` (UUID, PK)
- `empire_id` (TEXT, FK to empires)
- `author_wallet_address` (TEXT, FK to profiles)
- `message` (TEXT, max 500 chars)
- `posted_at` (TIMESTAMPTZ)

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

### 2. Complete Happiness System
**Status: ✅ FULLY IMPLEMENTED**
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
- **Maximum:** 200 turns (schema constraint verified and deployed ✅)
- **Regeneration:** +2 turns every 10 minutes (cron job)
- **Usage:** Scouting (yield affected by happiness), Growing (desertion risk)
- **Tracking:** `last_turns_regen_at` timestamp for efficiency
- **Migration:** `fix-turns-constraint-200.sql` ensures consistent 200 limit across all environments

### 4. Experience & Leveling System
**Status: ✅ FULLY IMPLEMENTED**
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
- Growing: 1 EXP per turn spent (same rate as scouting)
- Future actions: Combat, quests, achievements

**Level Up Mechanics:**
- Automatic level and title updates via `add_exp()` function
- Called automatically in scout/grow actions
- Level up notifications included in action responses
- Progress tracking available via `get_exp_progress()` function

### 5. Weed Growing System
**Status: ✅ FULLY IMPLEMENTED**
**Core Mechanics:**
- **Yield Formula:** `minions × (1 + rng_adjustment)` rounded down
- **RNG Adjustment:** 0-1 random value (average +0.5 = 1.5× multiplier)
- **Base Rate:** 1 weed per minion (100 minions = 100-200 weed yield)

**Efficiency Scaling:**
- **15-20 turns:** Optimal efficiency (no penalty)
- **<15 turns:** 30% efficiency penalty (inefficient small batches)
- **>20 turns:** No additional bonus (diminishing returns)

**Happiness Impact:**
- **≥70% minion happiness:** Full yield
- **<70% minion happiness:** 50% yield reduction
- **Desertion risk:** 10% chance to lose 1-5 minions if unhappy

**EXP Rewards:**
- **1 EXP per turn spent** (same rate as scouting)

**Grow Upgrades System:**
- **Status: ✅ FULLY IMPLEMENTED**
- **10 upgrade levels** from "Backyard Grow-Op" (0% bonus) to "Cosmic Kush Kingdom" (100% bonus)
- **Yield bonuses** applied AFTER base calculation but BEFORE database update
- **Requirements:** EXP thresholds + credit costs (Level 2: 0 EXP, 750 credits → Level 10: 250k EXP, 100k credits)
- **Purchase tracking:** All upgrades recorded in `lazywars_purchase_history` table
- **Backend functions:** `get_grow_upgrade_config()`, `get_player_grow_upgrade_info()`, `upgrade_grow_facility()`

### 6. Lazy Score Calculation
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
4. **`empire-dues-collection`** - ✅ Delta-based Empire dues collection every 30 minutes
   - **System:** Credit delta tracking (income since last collection)
   - **Formula:** `(current_credits - credits_at_last_dues) × dues_rate%`
   - **Performance:** Batch processing, atomic transactions per member
   - **Coverage:** Captures all income sources (OFModels, scouting, combat, manual)
   - **Interval:** 30 minutes (balanced performance vs responsiveness)

### Interactive Functions
1. **`purchase`** - Handles shop item purchases
2. **`scout`** - ✅ Scouting with complete happiness integration
   - Yield multipliers based on average happiness
   - 50% reduction if average happiness < 70%
   - Desertion risk during scouting
   - Real-time happiness calculation and updates
   - **✅ Automatic EXP and level up processing (1 EXP per turn)**
   - **✅ Turn spending tracking with yield recording**
3. **`grow`** - ✅ Weed growing with minion-based yield system
   - **Yield formula:** `minions × (1 + rng_adjustment)` (1-2 weed per minion)
   - **Efficiency scaling:** <15 turns = 30% penalty, 15+ turns = optimal
   - **Minion happiness affects yield:** <70% happiness = 50% reduction
   - **Desertion risk:** 10% chance if minions unhappy, lose 1-5 minions
   - **✅ Automatic EXP and level up processing (1 EXP per turn)**
   - **✅ Turn spending tracking with yield recording**
4. **`upgrade-farm`** - Farm equipment upgrades
5. **`cure-fatness`** - ✅ Remove fatness penalty using gym membership
   - Costs 1 gym_membership
   - Removes `is_fat` status
   - Auto-recalculates happiness after cure
6. **`add-exp`** - ✅ Manual EXP addition with level up checks
   - Admin/special event EXP rewards
   - Automatic level and title updates
   - Level up notifications

### Empire System Functions ✅
7. **`create-empire`** - Empire creation with validation
   - Level 3+ requirement check
   - Name uniqueness validation
   - Auto-adds creator as leader
8. **`empire-invite`** - Send invitations to players
   - Permission checks (leaders/members can invite)
   - 7-day invitation expiration
   - Duplicate invitation prevention
9. **`empire-join`** - Accept invitations and join Empires
   - Invitation validation and cleanup
   - Member count limits (max 20)
   - Profile updates with Empire bonuses
10. **`empire-leave`** - ✅ Leave current Empire with member removal
    - Leadership transfer to longest-serving member if leader leaves
    - Empire disbanding when last member leaves
    - Profile reset: empire_id, dues tracking, combat bonuses cleared
    - Member count and empire score updates
11. **`empire-purchase-minions`** - Purchase Empire minions for combat bonuses
    - Treasury balance validation
    - Minion cost: 15,000 credits each (renamed from thugs)
    - Combat bonus updates for all members
11. **`empire-dues-collection`** - ✅ Delta-based dues collection (30-min interval)
    - Credit delta tracking system
    - Atomic transactions with comprehensive error handling
    - Performance monitoring and reporting
12. **`empire-join-requests`** - ✅ NEW: Request-based joining system
    - Request creation with eligibility validation
    - Leader approval/denial workflow
    - Cancel request functionality for requesters
    - Automatic cleanup of processed requests
    - Global constraint: one pending request per player
13. **`kick-empire-member`** - ✅ Remove members from Empire (leaders only)
    - Permission validation and member removal
    - Profile cleanup and combat bonus updates
    - Empire member count and score adjustments

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
- `grow_weed(turns_spent, wallet_address)` - ✅ Weed growing with minion-based yield system + upgrade bonuses
  - Yield formula: minions × (1 + rng_adjustment) × upgrade_multiplier
  - Efficiency scaling for batch sizes (15+ turns optimal)
  - Minion happiness affects productivity
  - Desertion risk management
  - **✅ Upgrade bonus integration:** Applied after base calculation but before database update
- `upgrade_farm(upgrade_type, wallet_address)` - Farm upgrades

### Complete Happiness System Functions
**Status: ✅ IMPLEMENTED**
- `calculate_complete_happiness(ofmodels, minions, payout_pct, nft_costumes, is_fat, moutai, weapons_total)` - Core happiness calculation for both systems
- `update_happiness(wallet_address)` - Update and store happiness for a player
- `check_desertion_risk(ofmodels_happiness, minions_happiness, ofmodels, minions)` - Check desertion risk and calculate losses
- `cure_fatness(wallet_address)` - Cure fatness using gym membership (costs 1, removes penalty)

### EXP and Leveling Functions
**Status: ✅ IMPLEMENTED**
- `add_exp(wallet_address, exp_amount)` - Add EXP and automatically check for level up with title update
- `get_exp_progress(wallet_address)` - Get detailed level progress information for frontend
- `get_title_from_level(level)` - Get player title based on level
- `calculate_level_from_exp(exp)` - Calculate level from EXP amount (existing function)

### Grow Upgrades System Functions
**Status: ✅ IMPLEMENTED**
- `get_grow_upgrade_config(upgrade_level)` - Get upgrade configuration for specific level (name, cost, EXP req, yield bonus)
- `get_player_grow_upgrade_info(wallet_address)` - Get current upgrade level, next upgrade info, and upgrade eligibility
- `upgrade_grow_facility(wallet_address)` - Purchase next upgrade level with validation and purchase tracking

### Empire System Functions ✅
**Status: ✅ FULLY IMPLEMENTED**
- `create_empire(name, description, wallet_address)` - Empire creation with level validation
- `invite_to_empire(empire_id, wallet_address, invited_by)` - Send Empire invitations
- `join_empire(invitation_id, wallet_address)` - Accept invitations and join Empire
- `leave_empire(wallet_address)` - ✅ Leave current Empire with comprehensive cleanup
  - **Leadership transfer:** Auto-transfers to longest-serving member if leader leaves
  - **Empire disbanding:** Deletes empire when last member leaves
  - **Profile reset:** Clears empire_id, last_dues_payment_at, combat bonuses, credits_at_last_dues
  - **Member management:** Removes from empire_members table and updates counts
- `kick_empire_member(empire_id, member_wallet_address, kicker_wallet_address)` - Remove members (leaders only)
- `purchase_empire_minions(leader_wallet_address, minion_type, quantity)` - Buy offensive/defensive minions (renamed from thugs)
- `collect_empire_dues_delta()` - ✅ Delta-based dues collection with credit tracking
- `migrate_to_delta_dues_system()` - Migration function for deploying new dues system
- `update_empire_member_bonuses(empire_id)` - Recalculate combat bonuses for all members
- `get_empire_members(empire_id)` - Get Empire member list with profiles
- `get_empire_invitations(wallet_address)` - Get pending invitations for player

### Empire Join Request Functions ✅ NEW
**Status: ✅ NEWLY IMPLEMENTED**
- `can_request_to_join_empire(empire_id, wallet_address)` - Check join eligibility
- `create_empire_join_request(empire_id, wallet_address, username)` - Create join request
- `get_empire_join_requests(empire_id)` - Get pending requests for empire leaders
- `process_empire_join_request(request_id, action, leader_wallet_address)` - Approve/deny requests
- `cancel_empire_join_request(wallet_address)` - Cancel own pending request
- **Key Features:**
  - Global constraint: Only one pending request per player at a time
  - Automatic eligibility validation (not already member, empire has space)
  - Processed requests (approved/denied) are deleted from table
  - Leader-only approval/denial with permission validation

### Delta-Based Dues Collection Functions ✅
**Status: ✅ NEWLY IMPLEMENTED**
- `collect_empire_dues_delta()` - Main delta-based collection function
  - **Algorithm:** Credit delta tracking between collection cycles
  - **Performance:** Batch processing with atomic member transactions
  - **Error Handling:** Individual member failure isolation
  - **Monitoring:** Comprehensive result tracking and reporting
- `migrate_to_delta_dues_system()` - Safe migration from legacy system
- **Performance View:** `empire_dues_performance` - Real-time dues projections

### Analytics Functions
- `get_total_credits_spent(wallet_address)` - Purchase analytics
- `get_player_purchase_stats(wallet_address)` - Detailed purchase stats
- `get_upgrade_purchase_history(wallet_address)` - ✅ Upgrade-specific purchase history
- `get_total_upgrade_spending(wallet_address)` - ✅ Total credits spent on upgrades

### Turn Spending Analytics Functions ✅
- `get_total_turns_spent(wallet_address)` - Total lifetime turns spent across all activities
- `get_turns_spent_by_action(wallet_address, action_type)` - Turns spent by specific action ('scout'|'grow')
- `get_player_turn_stats(wallet_address)` - Comprehensive turn spending statistics:
  - Total turns spent, scout turns, grow turns
  - Total activities performed, first/last activity timestamps
  - Total yields (ofmodels, minions, weed) across all activities
- `get_player_turn_history(wallet_address, limit)` - Recent turn spending history with yields
- `lazywars_turn_analytics` - View combining turn spending with current player stats

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
  MAX_TURNS: 200,
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

GROW_UPGRADE_CONSTANTS = {
  MAX_LEVEL: 10,
  MIN_LEVEL: 1,
  YIELD_BONUSES: [0, 5, 10, 20, 30, 40, 55, 70, 85, 100], // % bonuses by level
  UPGRADE_NAMES: [
    "Backyard Grow-Op",
    "Closet Kush Corner", 
    "Basement Bud Bunker",
    "Garage Ganja Garden",
    "Attic Herb Haven",
    "Warehouse Weed Wonderland",
    "Industrial Dank Depot",
    "Mega Mary Jane Manor",
    "Epic Endo Empire",
    "Cosmic Kush Kingdom"
  ]
}

EMPIRE_CONSTANTS = {
  MIN_LEVEL_TO_CREATE: 3, // Block Besieger level required
  MAX_MEMBERS: 20,
  MIN_NAME_LENGTH: 3,
  MAX_NAME_LENGTH: 30,
  MAX_DESCRIPTION_LENGTH: 200,
  MAX_DUES_RATE: 50, // 50% maximum
  MINION_COST: 15000, // 15,000 credits per minion
  MINION_ATTACK_BONUS: 10, // +10 attack per offensive minion
  MINION_DEFENSE_BONUS: 10, // +10 defense per defensive minion
  INVITATION_EXPIRY_DAYS: 7,
  MAX_MESSAGE_LENGTH: 500,
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
- **✅ Weed growing with minion-based yield system + upgrade bonuses**
- **✅ Purchase system with analytics + upgrade purchase tracking**
- **✅ Turn spending tracking and analytics for scout/grow activities**
- **✅ Lazy score calculation and rankings**
- **✅ Grow upgrades system with 10 levels and yield bonuses**
- **✅ Empire system with alliance mechanics, treasury, and combat bonuses**
- **✅ UI/UX consistency and design system (August 2025)**

### ⚠️ Partially Implemented
- Attack/defense mechanics (basic framework exists, Empire bonuses implemented)

### ❌ Not Implemented
- Advanced combat mechanics with happiness effects
- Achievement and quest systems

### 🔄 NEXT PRIORITIES:
1. Advanced combat mechanics with happiness effects on battle outcomes
2. Achievement and quest systems with EXP rewards  
3. Enhanced animation system and user experience polish
4. Advanced leaderboards with filtering and search capabilities
