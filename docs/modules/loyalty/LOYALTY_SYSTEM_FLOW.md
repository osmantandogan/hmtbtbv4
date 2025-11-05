# Loyalty Program System Flow Diagrams

## 1. Points Earning Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    User Books Travel                         │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Booking Confirmed & Paid                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Backend calls awardPoints()                                 │
│  - user_id: User UUID                                        │
│  - booking_id: Booking UUID                                  │
│  - amount: $500                                              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Get User's Current Tier                                     │
│  - Query loyalty_accounts table                              │
│  - Get current_tier with multiplier                          │
│  - Example: Silver tier (1.5x multiplier)                    │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Calculate Points                                            │
│  - Base: $500 × 10 = 5,000 points                           │
│  - With multiplier: 5,000 × 1.5 = 7,500 points             │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Database Function: award_loyalty_points()                   │
│  1. Update loyalty_accounts:                                 │
│     - total_points += 7,500                                  │
│     - lifetime_points += 7,500                               │
│  2. Create loyalty_transaction record                        │
│  3. Call update_loyalty_tier()                               │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Check Tier Eligibility                                      │
│  - Current points: 7,500                                     │
│  - Current tier: Silver (1,001-5,000)                        │
│  - New tier: Gold (5,001-15,000) ✓                          │
│  - Tier upgraded!                                            │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Return Result                                               │
│  - success: true                                             │
│  - points_awarded: 7,500                                     │
│  - new_balance: 7,500                                        │
│  - tier_changed: true                                        │
│  - new_tier: Gold                                            │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Optional: Send Notifications                                │
│  - "Congratulations! You've been upgraded to Gold tier!"     │
│  - "You earned 7,500 points from your booking"               │
└─────────────────────────────────────────────────────────────┘
```

## 2. Reward Redemption Flow

```
┌─────────────────────────────────────────────────────────────┐
│  User Browses Available Rewards                              │
│  - GET /api/loyalty/rewards                                  │
│  - Displays rewards with points required                     │
│  - Shows user's current balance                              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  User Selects Reward to Redeem                               │
│  - Example: "$100 Travel Voucher"                            │
│  - Points required: 10,000                                   │
│  - User's balance: 7,500 ❌ (insufficient)                   │
│  OR                                                           │
│  - User's balance: 12,000 ✓ (sufficient)                    │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  User Clicks "Redeem Now"                                    │
│  - POST /api/loyalty/redeem                                  │
│  - Body: { reward_id: "uuid" }                               │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Validate Redemption                                         │
│  1. Check user has enough points                             │
│  2. Check reward is active                                   │
│  3. Check tier requirements (if any)                         │
│  4. Check max redemptions not exceeded                       │
│  5. Check stock available (if limited)                       │
└────────────────────────┬────────────────────────────────────┘
                         │ (All checks pass)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Generate Redemption Code                                    │
│  - Format: HMT-1704326400-ABC123                            │
│  - Unique per redemption                                     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Database Function: redeem_loyalty_points()                  │
│  1. Deduct points from loyalty_accounts                      │
│     - total_points -= 10,000                                 │
│     - new balance: 2,000                                     │
│  2. Create loyalty_transaction (negative points)             │
│     - type: 'redeem'                                         │
│     - points: -10,000                                        │
│  3. Check if tier dropped (unlikely but possible)            │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Create Redemption Record                                    │
│  - loyalty_redemptions table                                 │
│  - status: 'approved'                                        │
│  - redemption_code: HMT-1704326400-ABC123                   │
│  - points_spent: 10,000                                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Return Success Response                                     │
│  - redemption_id: UUID                                       │
│  - redemption_code: HMT-1704326400-ABC123                   │
│  - points_spent: 10,000                                      │
│  - new_balance: 2,000                                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Display to User                                             │
│  - "Success! Your redemption code: HMT-1704326400-ABC123"   │
│  - "Save this code for your next booking"                    │
│  - "New balance: 2,000 points"                               │
└─────────────────────────────────────────────────────────────┘
```

## 3. Tier Progression Flow

```
┌─────────────────────────────────────────────────────────────┐
│                     User Journey                             │
└─────────────────────────────────────────────────────────────┘

New User
  ↓
┌─────────────────────┐
│  BRONZE TIER        │  Points: 0
│  Multiplier: 1.0x   │  Benefits:
│  Color: #CD7F32     │  - Basic earning rate
└──────────┬──────────┘  - Email support
           │             - Member rates
           │ Book $100 trip → earn 1,000 points
           │
           ↓ (1,001 points)
┌─────────────────────┐
│  SILVER TIER        │  Points: 1,001 - 5,000
│  Multiplier: 1.5x   │  Benefits:
│  Color: #C0C0C0     │  - 50% more points
└──────────┬──────────┘  - Room upgrades
           │             - Late checkout
           │             - 10% spa discount
           │ Book $300 trip → earn 4,500 points (300×10×1.5)
           │
           ↓ (5,501 points)
┌─────────────────────┐
│  GOLD TIER          │  Points: 5,001 - 15,000
│  Multiplier: 2.0x   │  Benefits:
│  Color: #FFD700     │  - 2x points earning
└──────────┬──────────┘  - Guaranteed upgrades
           │             - 4PM checkout
           │             - Free breakfast
           │             - 20% spa discount
           │ Book $500 trip → earn 10,000 points (500×10×2.0)
           │
           ↓ (15,501 points)
┌─────────────────────┐
│  PLATINUM TIER      │  Points: 15,001+
│  Multiplier: 3.0x   │  Benefits:
│  Color: #E5E4E2     │  - 3x points earning
└─────────────────────┘  - Suite upgrades
                          - 6PM checkout
                          - Concierge service
                          - 30% spa discount
                          - Exclusive events

Note: Tier is calculated automatically after each transaction
```

## 4. Dashboard Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│  User Visits Loyalty Dashboard                               │
│  - Route: /loyalty                                           │
│  - Component: <LoyaltyDashboard userId={user.id} />         │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  API Call: GET /api/loyalty/account                          │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Backend Fetches Data (Parallel)                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 1. getOrCreateLoyaltyAccount()                       │  │
│  │    - Total points: 7,500                             │  │
│  │    - Lifetime points: 12,500                         │  │
│  │    - Current tier: Gold                              │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ 2. getTierProgress()                                 │  │
│  │    - Current: Gold (5,001-15,000)                    │  │
│  │    - Next: Platinum (15,001+)                        │  │
│  │    - Points needed: 7,501                            │  │
│  │    - Progress: 33%                                   │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ 3. getTransactionHistory()                           │  │
│  │    - Last 10 transactions                            │  │
│  │    - Earn/redeem/bonus records                       │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ 4. getAvailableRewards()                             │  │
│  │    - All active rewards                              │  │
│  │    - With user redemption info                       │  │
│  │    - Can redeem flags                                │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ 5. getLoyaltyStatistics()                            │  │
│  │    - Points this month: 5,000                        │  │
│  │    - Points this year: 12,500                        │  │
│  │    - Total redemptions: 3                            │  │
│  │    - Member since: 2024-01-15                        │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Return LoyaltyDashboardData                                 │
│  {                                                            │
│    account: { ... },                                         │
│    tier_progress: { ... },                                   │
│    recent_transactions: [ ... ],                             │
│    available_rewards: [ ... ],                               │
│    statistics: { ... }                                       │
│  }                                                            │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Render Dashboard Components                                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Header Section                                       │  │
│  │ - <PointsDisplay points={7500} />                    │  │
│  │ - <TierBadge tier={Gold} />                          │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ Statistics Cards                                     │  │
│  │ [This Month] [This Year] [Total Rewards]            │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ Tab: Overview (Active)                               │  │
│  │ - <TierProgressCard progress={...} />                │  │
│  │ - <LoyaltyBenefits tier={Gold} />                    │  │
│  │ - <TransactionHistory transactions={...} />          │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ Tab: Rewards                                         │  │
│  │ - <RewardsGrid rewards={...} />                      │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ Tab: History                                         │  │
│  │ - <TransactionHistory transactions={...} />          │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 5. Database Structure & Relationships

```
┌─────────────────────────────────────────────────────────────┐
│                    Database Tables                           │
└─────────────────────────────────────────────────────────────┘

┌──────────────────┐
│ loyalty_tiers    │
│──────────────────│          ┌─────────────────────┐
│ id (PK)          │◄─────────│ loyalty_accounts    │
│ name             │  current │─────────────────────│
│ min_points       │   tier   │ id (PK)             │
│ max_points       │          │ user_id (UNIQUE)    │
│ benefits (jsonb) │◄─────────│ current_tier_id (FK)│
│ multiplier       │   next   │ next_tier_id (FK)   │
│ color            │   tier   │ total_points        │
│ created_at       │          │ lifetime_points     │
└──────────────────┘          │ tier_expiry_date    │
                              └──────────┬──────────┘
                                         │
                                         │ user_id
                                         │
                  ┌──────────────────────┼──────────────────────┐
                  │                      │                      │
                  ▼                      ▼                      ▼
     ┌────────────────────┐  ┌─────────────────────┐  ┌────────────────────┐
     │loyalty_transactions│  │ loyalty_redemptions  │  │  loyalty_rewards   │
     │────────────────────│  │─────────────────────│  │────────────────────│
     │ id (PK)            │  │ id (PK)             │  │ id (PK)            │
     │ user_id (FK)       │  │ user_id (FK)        │  │ name               │
     │ booking_id         │  │ reward_id (FK)      │──│ description        │
     │ type (enum)        │  │ points_spent        │  │ points_required    │
     │ points             │  │ status (enum)       │  │ category (enum)    │
     │ description        │  │ redemption_code     │  │ discount_%         │
     │ balance_after      │  │ booking_id          │  │ is_active          │
     │ created_at         │  │ created_at          │  │ stock_quantity     │
     └────────────────────┘  └─────────────────────┘  │ created_at         │
                                                       └────────────────────┘

Relationships:
- loyalty_accounts.current_tier_id → loyalty_tiers.id (current tier)
- loyalty_accounts.next_tier_id → loyalty_tiers.id (next tier to achieve)
- loyalty_transactions.user_id → loyalty_accounts.user_id (all transactions)
- loyalty_redemptions.user_id → loyalty_accounts.user_id (all redemptions)
- loyalty_redemptions.reward_id → loyalty_rewards.id (which reward)
```

## 6. Points Calculation Examples

```
┌─────────────────────────────────────────────────────────────┐
│                 Booking Amount: $500                         │
└─────────────────────────────────────────────────────────────┘

Bronze Tier (1.0x)
─────────────────
Base calculation: $500 × 10 = 5,000 base points
Tier multiplier: 5,000 × 1.0 = 5,000 points
Result: 5,000 points earned


Silver Tier (1.5x)
─────────────────
Base calculation: $500 × 10 = 5,000 base points
Tier multiplier: 5,000 × 1.5 = 7,500 points
Result: 7,500 points earned (+50% bonus)


Gold Tier (2.0x)
─────────────────
Base calculation: $500 × 10 = 5,000 base points
Tier multiplier: 5,000 × 2.0 = 10,000 points
Result: 10,000 points earned (+100% bonus)


Platinum Tier (3.0x)
─────────────────
Base calculation: $500 × 10 = 5,000 base points
Tier multiplier: 5,000 × 3.0 = 15,000 points
Result: 15,000 points earned (+200% bonus)


┌─────────────────────────────────────────────────────────────┐
│             Points Required to Reach Each Tier               │
└─────────────────────────────────────────────────────────────┘

Bronze → Silver: 1,001 points needed
Silver → Gold: 5,001 points needed
Gold → Platinum: 15,001 points needed

Example progression for Bronze member:
- Book $100 trip: 1,000 pts → Still Bronze (0-1,000)
- Book $100 trip: 1,000 pts → Total 2,000 → Silver! (1,001-5,000)
- Now earning 1.5x multiplier
- Book $300 trip: 4,500 pts → Total 6,500 → Gold! (5,001-15,000)
- Now earning 2.0x multiplier
- Book $500 trip: 10,000 pts → Total 16,500 → Platinum! (15,001+)
- Now earning 3.0x multiplier
```

## 7. Error Handling Flow

```
┌─────────────────────────────────────────────────────────────┐
│              Common Error Scenarios                          │
└─────────────────────────────────────────────────────────────┘

Scenario 1: Insufficient Points
────────────────────────────────
User balance: 5,000 points
Reward cost: 10,000 points
                │
                ▼
┌───────────────────────────┐
│ Validation in redeemPoints│
│ Check: balance < required │
│ Result: throw Error       │
└──────────┬────────────────┘
           │
           ▼
┌───────────────────────────┐
│ API catches error         │
│ Returns 400 status        │
│ Error: "Insufficient pts" │
└──────────┬────────────────┘
           │
           ▼
┌───────────────────────────┐
│ Frontend shows alert      │
│ "You need 10,000 points"  │
│ "You have 5,000 points"   │
└───────────────────────────┘


Scenario 2: Reward Not Available
─────────────────────────────────
User tries to redeem inactive reward
                │
                ▼
┌───────────────────────────┐
│ Database query            │
│ WHERE is_active = true    │
│ Result: No rows           │
└──────────┬────────────────┘
           │
           ▼
┌───────────────────────────┐
│ API returns 404           │
│ Error: "Reward not found" │
└──────────┬────────────────┘
           │
           ▼
┌───────────────────────────┐
│ Frontend shows message    │
│ "This reward is no longer"│
│ "available"               │
└───────────────────────────┘


Scenario 3: Max Redemptions Reached
────────────────────────────────────
Reward limit: 3 per user
User has redeemed: 3 times
                │
                ▼
┌───────────────────────────┐
│ Check redemption count    │
│ Count >= max_limit        │
│ Result: throw Error       │
└──────────┬────────────────┘
           │
           ▼
┌───────────────────────────┐
│ API returns 400           │
│ Error: "Max reached"      │
└──────────┬────────────────┘
           │
           ▼
┌───────────────────────────┐
│ Frontend shows message    │
│ "You've already redeemed" │
│ "this reward 3 times"     │
└───────────────────────────┘
```

## 8. Security & RLS Flow

```
┌─────────────────────────────────────────────────────────────┐
│        Row Level Security (RLS) Protection                   │
└─────────────────────────────────────────────────────────────┘

Request Flow with RLS:
──────────────────────

1. User A (uuid-111) makes request
                │
                ▼
2. Supabase checks auth.uid()
   - Returns: uuid-111
                │
                ▼
3. Query: SELECT * FROM loyalty_accounts
          WHERE user_id = ?
                │
                ▼
4. RLS Policy Applied:
   - "Users can view their own loyalty account"
   - USING (auth.uid() = user_id)
                │
                ▼
5. Database filters automatically:
   WHERE user_id = 'uuid-111'
   AND auth.uid() = 'uuid-111' ✓
                │
                ▼
6. Returns only User A's data


Attack Prevention:
──────────────────

Attempt: User A tries to access User B's data
                │
                ▼
Request: GET /api/loyalty/account
Auth: User A (uuid-111)
Try to fetch: User B (uuid-222)
                │
                ▼
RLS Policy Check:
- auth.uid() = uuid-111
- Requesting user_id = uuid-222
- Check: uuid-111 = uuid-222 ❌
                │
                ▼
Result: Empty result set / Access denied
User A cannot see User B's data
```

---

These diagrams provide a comprehensive visual understanding of how the loyalty program system works from end to end.
