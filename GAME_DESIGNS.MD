# LazyWars Backend System

## Overview

LazyWars is a blockchain-based idle gaming dApp built on the Solana network with a comprehensive backend system powered by Supabase. This document provides developers with a clear understanding of the backend architecture, game mechanics, and system design patterns without exposing sensitive security information.

It was built publicly using Grok for research and initial framework, v0 for initial frontend designs, and agents using Sonnet/Opus created the initial codebase/structure.

Public X Thread:
https://x.com/adidogCEO/status/1949623428618580463

## Architecture Overview

### Technology Stack
- **Database**: PostgreSQL (via Supabase)
- **API Layer**: Supabase Edge Functions (Deno-based serverless functions)
- **Real-time**: Supabase Realtime subscriptions
- **Authentication**: Supabase Auth with Solana wallet integration
- **Scheduled Tasks**: Supabase Cron for game mechanics
- **Frontend**: Expo React Native with TypeScript

### Core Design Principles
- **Event-driven architecture** with database triggers for game mechanics
- **Atomic transactions** for all game state changes
- **Row-level security** for data access control
- **Materialized views** for performance optimization
- **Comprehensive audit trails** for all player actions

## Database Schema

### Core Tables

#### `lazywar_profiles` - Player Data
The central player table containing all game state information:

**Identity & Authentication**
- `id`: UUID primary key
- `wallet_address`: Unique Solana wallet identifier
- `username`: Player display name (4-21 characters)
- `created_at` / `updated_at`: Timestamps

**Game Resources**
- `credits`: Primary in-game currency (≥0)
- `turns`: Action points for activities (0-200, regenerates over time)
- `ofmodels`: Primary income-generating units (≥0)
- `minions`: Support units for combat and activities (≥0)

**Progression System**
- `exp`: Experience points (≥0)
- `level`: Player level (1-10, calculated from EXP)
- `lazy_rank`: Ranking position
- `lazy_score`: Calculated ranking metric
- `title`: Level-based title progression

**Game Mechanics State**
- `ofmodels_happiness` / `minions_happiness`: Unit morale (0-100)
- `payout_pct`: Income distribution percentage (0-100)
- `is_fat`: Status effect impacting happiness
- `city`: Player location (affects certain mechanics)

**Inventory System**
- Various item types: `nft_costumes`, `gym_memberships`, `moutai`, `weed`
- Combat items: `sig_p365`, `beretta_1301`, `vector_smg`, `cybertruck`
- Utility items: `boombox` (entertainment)

**Farm System**
- `farm_hydro_level` / `farm_led_level`: Upgrade levels (≥0)
- `grow_upgrade_level`: Growing facility level (1-10)

**Empire Integration**
- `empire_id`: Foreign key to empire membership
- `empire_attack_bonus` / `empire_defense_bonus`: Combat modifiers
- Empire financial tracking fields

#### `lazywars_empires` - Guild System
Player organizations with shared resources and goals:

- `id`: Unique empire identifier
- `name`: Empire display name (unique, 4-21 characters)
- `leader_wallet_address`: Empire leader reference
- `description`: Optional empire description
- `dues_rate`: Member contribution percentage (0-50)
- `treasury_credits`: Shared empire treasury
- `offensive_minions` / `defensive_minions`: Empire combat bonuses
- `empire_score`: Aggregate member score
- `member_count`: Current active members
- `total_members_lifetime`: Historical membership tracking

#### `lazywars_combat` - Battle System
Comprehensive combat logging and mechanics:

- Attack metadata: `attack_wallet_address`, `target_wallet_address`
- Combat details: `attack_type`, `minions_sent`, `cybertruck_used`
- Calculations: `attacker_atk`, `target_def`, `success_probability`
- Results: `attack_success`, various `yield_*` fields
- Unit losses: `attack_minions_loss`, `target_minions_loss`
- Happiness states before/after combat

**Attack Types**
- Drive-By Shooting: Basic minion-based attack
- Home Invasion: Resource theft focused
- Jack Cybertrucks: Vehicle theft
- Smokeup Models: Happiness-based attack
- Steal OFmodels: Direct unit theft

### Analytics & Tracking Tables

#### `lazywars_purchase_history`
Complete transaction logging for all in-game purchases:
- Item details, quantities, costs
- Timestamp tracking for economic analysis

#### `lazywars_turn_spending_history`
Activity tracking for turn-based actions:
- Action types: scouting, growing
- Resource yields by activity
- Performance analytics data

### Empire Management Tables

#### `lazywars_empire_members`
Empire membership tracking with roles and contributions.

#### `lazywars_empire_invitations` / `lazywars_empire_join_requests`
Empire recruitment system with invitation and application workflows.

## Game Mechanics Architecture

### 1. Income Generation System

**Hourly Income Calculation**
```
Base Income = ofmodels × 10 credits/hour
Happiness Modifier = ofmodels_happiness / 100
Payout Adjustment = 1 - ((payout_pct - 40) / 60)
Final Income = Base × Happiness × Payout Adjustment
```

**Automated Processing**
- Cron job runs hourly income calculation
- Database triggers maintain happiness levels
- Desertion mechanics for low happiness units

### 2. Happiness System

**Factors Affecting Happiness**
- Payout percentage (higher payout = happier units)
- NFT costumes (happiness boost)
- Minion-to-model ratio (protection factor)
- Fatness penalty (reduces happiness)
- Time-based decay (gradual reduction)

**Database Triggers**
Happiness calculations are automated through PostgreSQL triggers that fire on relevant data changes, ensuring consistency and performance.

### 3. Combat System

**Combat Resolution**
1. Attack/Defense calculation based on units and equipment
2. Success probability determination
3. Random outcome generation
4. Resource transfer and unit losses
5. Happiness impact on both parties
6. Experience point awards

**Combat Types**
Each attack type has unique mechanics:
- Resource requirements (turns, equipment, consumables)
- Success probability modifiers
- Yield calculations
- Defensive countermeasures

### 4. Turns System

**Turn Management**
- Maximum: 200 turns per player
- Regeneration: +2 turns every 10 minutes
- Activities consume turns (scouting, growing, combat)
- Efficiency affected by happiness levels

### 5. Empire System

**Empire Economics**
- Member dues based on income percentage
- Shared treasury for empire purchases
- Collective bonuses (attack/defense modifiers)
- Empire scoring based on member performance

**Empire Mechanics**
- Leadership structure with role-based permissions
- Invitation and join request systems
- Member contribution tracking
- Empire vs empire competition

## Edge Functions Architecture

### Function Categories

**Game Action Functions**
- `grow/`: Resource generation activity
- `scout/`: Unit acquisition activity
- `attack/`: Combat initiation
- `purchase/`: Shop transactions

**Empire Functions**
- `create-empire/`: New empire establishment
- `empire-join/`: Membership management
- `empire-invite/`: Recruitment system
- `empire-purchase-*`: Empire-level purchases

**Automated Functions**
- `hourly-income/`: Scheduled income processing
- `turns-regeneration/`: Turn point restoration
- `happiness-decay/`: Happiness degradation
- `empire-dues-collection/`: Automated dues collection

**Utility Functions**
- `add-exp/`: Experience point management
- `cure-fatness/`: Status effect removal
- Various upgrade and maintenance functions

### Function Design Patterns

**Input Validation**
- Comprehensive parameter checking
- Range validation for numerical inputs
- Player state verification

**Transaction Safety**
- Database transactions for atomic operations
- Rollback on failure conditions
- Consistent error handling

**Performance Optimization**
- Efficient database queries
- Minimal computational overhead
- Cached calculations where appropriate

## Data Flow Architecture

### Player Actions Flow
1. Client sends request to Edge Function
2. Function validates input and player state
3. Database transaction begins
4. Game mechanics calculations performed
5. Multiple table updates in single transaction
6. Analytics data recorded
7. Response sent to client
8. Real-time updates broadcast to relevant subscribers

### Automated Systems Flow
1. Cron triggers scheduled function
2. Function processes all eligible players
3. Batch updates for efficiency
4. Error handling for individual failures
5. Success/failure logging
6. Performance metrics collection

### Empire Systems Flow
1. Empire actions affect multiple players
2. Treasury and bonus calculations
3. Member notification systems
4. Aggregate scoring updates
5. Leaderboard recalculation

## Performance Considerations

### Database Optimization
- Strategic indexing on frequently queried columns
- Materialized views for complex aggregations
- Query optimization for large datasets
- Connection pooling for Edge Functions

### Caching Strategy
- Client-side caching for static game data
- Database-level caching for computed values
- Edge Function response caching where appropriate

### Scalability Design
- Horizontal scaling capability through Supabase
- Stateless Edge Functions
- Event-driven architecture reducing computational load
- Efficient data structures minimizing storage requirements

## Security Model

### Access Control
- Row-level security policies on all tables
- Function-level authentication requirements
- Player data isolation
- Admin function restrictions

### Data Validation
- Input sanitization at multiple levels
- Business rule enforcement through constraints
- Audit trails for all sensitive operations
- Anti-cheating mechanisms

### Rate Limiting
- Function execution limits
- Player action frequency controls
- Resource consumption monitoring

## Monitoring & Analytics

### Performance Metrics
- Function execution times
- Database query performance
- Player action frequencies
- System resource utilization

### Game Analytics
- Player progression tracking
- Economic system monitoring
- Combat outcome analysis
- Empire performance metrics

### Error Tracking
- Comprehensive error logging
- Player impact assessment
- System health monitoring
- Automated alerting systems

## Development Workflow

### Local Development
- Supabase CLI for local database setup
- Edge Function local testing
- Database migration management
- Environment configuration

### Testing Strategy
- Unit tests for game mechanics
- Integration tests for Edge Functions
- Performance testing for high-load scenarios
- Player experience testing

### Deployment Process
- Staged deployment through environments
- Database migration automation
- Edge Function deployment
- Configuration management

## API Patterns

### Request/Response Format
```typescript
// Standard request format
interface GameActionRequest {
  wallet_address: string
  // Action-specific parameters
}

// Standard response format
interface GameActionResponse {
  success: boolean
  data?: any
  error?: string
  // Action-specific results
}
```

### Error Handling
Consistent error response patterns with appropriate HTTP status codes and descriptive error messages.

### Real-time Updates
Supabase Realtime integration for immediate UI updates on game state changes.

## Integration Points

### Solana Blockchain
- Wallet authentication
- Transaction signing for premium features
- Message signing

### Mobile Application
- Expo React Native client
- TypeScript for type safety
- React Query for state management

### External Services
- Analytics platforms for player behavior
- Forums for player interaction
- Additional leaderboards on hyperlazy platform

## Full Blockchain Migration Plans

### Fully Onchain Data That's Forkable

LazyWars is architected with a long-term vision to become a fully onchain game, transitioning from the current hybrid model (Supabase backend + Solana authentication) to a complete blockchain-based system. This migration will ensure true decentralization, data sovereignty, and enable community-driven game evolution where the game is opensource by default as the data is the game.

Being a game that's uniquely centered around the data and its history. Putting the game onchain by default puts the entire game onchain as an open-source game that anyone can freely fork into a new timeline with new mechanics as they wish.

The data is the game, the game is the data.

### Micro-Transaction Architecture

**Automated Transaction System**
- Every game action will generate a micro-transaction on Solana
- Each player will be assigned a dedicated keypair for automated transaction signing
- Unique wallet address per player

### Data Sovereignty & Reconstruction

**Complete Game State in Memos**
- Every transaction memo contains sufficient data to reconstruct events
- Chronological transaction history serves as history of events
- No reliance on centralized databases for core game data
- Full auditability and transparency of all game history

### Decentralized Game Evolution

**Forkable Game Timeline**
- Complete game history stored onchain enables community forks
- Different rulesets can be applied to the same transaction history
- Community governance for game mechanics changes
- Multiple frontend implementations possible

**Frontend Agnostic Design**
- Any developer can build a LazyWars interface using onchain data
- Mobile apps, web interfaces, CLI tools all supported
- Real-time sync across different frontend implementations
- Open-source data

### Example Implementation Strategy

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

This onchain evolution represents the ultimate vision for LazyWars: a truly decentralized, community-owned, and infinitely extensible gaming platform built on top of a blockchain

## Conclusion

This backend system provides a robust, scalable foundation for the LazyWars gaming experience based on the original pimpwars formula.
