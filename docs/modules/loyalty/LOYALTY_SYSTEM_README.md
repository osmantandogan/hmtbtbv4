# HealMedy Travel Loyalty Program System

## Overview
Complete loyalty rewards program with points earning, tier progression, and reward redemption capabilities.

## System Components

### 1. Database Schema
**File:** `supabase/migrations/20250103_loyalty_program.sql`

#### Tables Created:
- **loyalty_tiers** - Defines Bronze, Silver, Gold, and Platinum tiers with benefits
- **loyalty_accounts** - User accounts with points balances and tier information
- **loyalty_transactions** - All point earning and spending activities
- **loyalty_rewards** - Catalog of available rewards
- **loyalty_redemptions** - Records of user reward redemptions

#### Key Features:
- Row Level Security (RLS) enabled on all tables
- Database functions for point calculations and tier updates
- Automatic tier progression based on points
- Transaction history tracking
- Indexes for performance optimization

### 2. TypeScript Types
**File:** `lib/types/loyalty.ts`

Comprehensive type definitions including:
- Enums for transaction types, reward categories, and redemption statuses
- Interfaces for all database models
- Request/response types for API endpoints
- Utility constants for tier colors, multipliers, and labels

### 3. Loyalty Service
**File:** `lib/loyalty/points.ts`

Core business logic functions:

#### Main Functions:
```typescript
// Calculate points for an amount with tier multiplier
calculatePoints(amount: number, tierMultiplier: number): PointsCalculation

// Award points to user (called after booking confirmation)
awardPoints(request: AwardPointsRequest): Promise<AwardPointsResponse | null>

// Redeem points for a reward
redeemPoints(userId: string, request: RedeemRewardRequest): Promise<RedeemRewardResponse | null>

// Check and update user tier
checkTierEligibility(userId: string): Promise<{ tier_changed: boolean; new_tier?: LoyaltyTier }>

// Get current points balance
getPointsBalance(userId: string): Promise<number>

// Get transaction history with pagination
getTransactionHistory(userId: string, page: number, pageSize: number): Promise<TransactionHistoryResponse>

// Get tier progress information
getTierProgress(userId: string): Promise<TierProgress | null>

// Get available rewards for user
getAvailableRewards(userId: string, category?: string): Promise<LoyaltyReward[]>

// Get loyalty statistics
getLoyaltyStatistics(userId: string): Promise<Statistics>
```

### 4. React Components
**Location:** `components/loyalty/`

#### Components Created:

1. **LoyaltyDashboard.tsx** - Main dashboard component
   - Shows points balance, tier status, and progress
   - Tabbed interface for overview, rewards, and history
   - Statistics cards for monthly/yearly points
   - Integrated reward redemption

2. **TierBadge.tsx** - Visual tier badge
   - Displays tier icon and name
   - Color-coded by tier level
   - Configurable sizes (sm, md, lg)

3. **PointsDisplay.tsx** - Animated points counter
   - Shows current points balance
   - Optional animation on mount
   - Coin icon with formatted numbers

4. **TierProgressCard.tsx** - Progress to next tier
   - Visual progress bar
   - Points needed to next tier
   - Current and next tier badges

5. **LoyaltyBenefits.tsx** - Tier benefits list
   - Displays all benefits for current tier
   - Check mark icons
   - Tier color coding

6. **RewardCard.tsx** - Individual reward display
   - Reward details and image
   - Points required
   - Redeem button with validation
   - Stock and limit tracking

7. **RewardsGrid.tsx** - Grid of available rewards
   - Category filtering
   - Responsive grid layout
   - Empty state handling

8. **TransactionHistory.tsx** - Transaction list
   - Chronological point history
   - Earn/spend differentiation
   - Pagination support
   - Balance tracking

### 5. API Routes
**Location:** `app/api/loyalty/`

#### Endpoints Created:

1. **GET /api/loyalty/account**
   - Returns complete loyalty dashboard data
   - Includes account, tier progress, transactions, rewards, and statistics
   - Authenticated endpoint

2. **POST /api/loyalty/points/award**
   - Award points to user (internal use)
   - Called after booking confirmation
   - Request body: `{ user_id, booking_id, amount, description }`
   - Returns: Transaction details and new balance

3. **GET /api/loyalty/rewards**
   - Get available rewards for user
   - Optional category filter: `?category=voucher`
   - Returns: Rewards with user-specific redemption info

4. **POST /api/loyalty/redeem**
   - Redeem points for a reward
   - Request body: `{ reward_id, booking_id?, notes? }`
   - Returns: Redemption code and confirmation

5. **GET /api/loyalty/transactions**
   - Get transaction history
   - Pagination: `?page=1&page_size=20`
   - Returns: Transactions with pagination info

6. **GET /api/loyalty/tiers**
   - Get all tier information (public)
   - No authentication required
   - Returns: All tiers with benefits

## Tier Configuration

### Tier Levels

| Tier | Points Range | Multiplier | Color |
|------|-------------|------------|-------|
| Bronze | 0 - 1,000 | 1.0x | #CD7F32 |
| Silver | 1,001 - 5,000 | 1.5x | #C0C0C0 |
| Gold | 5,001 - 15,000 | 2.0x | #FFD700 |
| Platinum | 15,001+ | 3.0x | #E5E4E2 |

### Points Calculation
**Base Rate:** $1 spent = 10 points

**Examples:**
- Bronze member books $100 trip: 100 × 10 × 1.0 = 1,000 points
- Silver member books $100 trip: 100 × 10 × 1.5 = 1,500 points
- Gold member books $100 trip: 100 × 10 × 2.0 = 2,000 points
- Platinum member books $100 trip: 100 × 10 × 3.0 = 3,000 points

## Implementation Guide

### Step 1: Run Database Migration
```bash
# Apply the migration to your Supabase project
supabase db push

# Or run directly in Supabase SQL editor:
# Copy contents of 20250103_loyalty_program.sql
```

### Step 2: Award Points on Booking Completion
Add to your booking confirmation flow:

```typescript
// In your booking completion handler
import { awardPoints } from '@/lib/loyalty/points';

async function onBookingConfirmed(booking) {
  // Award points based on booking amount
  await awardPoints({
    user_id: booking.user_id,
    booking_id: booking.id,
    amount: booking.total_amount,
    description: `Booking #${booking.id}`,
  });
}
```

### Step 3: Add Loyalty Dashboard to Your App
Create a loyalty page:

```typescript
// app/loyalty/page.tsx
import { LoyaltyDashboard } from '@/components/loyalty/LoyaltyDashboard';
import { createClient } from '@/lib/supabase/server';
import { redirect } from 'next/navigation';

export default async function LoyaltyPage() {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    redirect('/auth/login');
  }

  return (
    <div className="container mx-auto px-4 py-8">
      <LoyaltyDashboard userId={user.id} />
    </div>
  );
}
```

### Step 4: Display Points in User Profile/Header
```typescript
import { getPointsBalance } from '@/lib/loyalty/points';
import { PointsDisplay } from '@/components/loyalty/PointsDisplay';

async function UserHeader({ userId }) {
  const points = await getPointsBalance(userId);

  return (
    <div className="user-header">
      <PointsDisplay points={points} size="sm" />
    </div>
  );
}
```

### Step 5: Add Rewards Section
```typescript
// app/rewards/page.tsx
import { RewardsGrid } from '@/components/loyalty/RewardsGrid';

export default function RewardsPage() {
  return (
    <div className="container mx-auto px-4 py-8">
      <h1>Redeem Your Rewards</h1>
      <RewardsGrid
        rewards={rewards}
        onRedeem={handleRedeem}
      />
    </div>
  );
}
```

## Sample Rewards

The migration includes 10 sample rewards:
- $50, $100, $250 Travel Vouchers
- 10% and 20% Discount Codes
- Free Room Upgrade
- Airport Lounge Access
- Spa Treatment Voucher
- Private City Tour
- Romantic Dinner Package

## API Usage Examples

### Award Points After Booking
```typescript
const response = await fetch('/api/loyalty/points/award', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    user_id: 'uuid',
    booking_id: 'booking-uuid',
    amount: 500.00,
    description: 'Medical travel booking',
  }),
});

const result = await response.json();
// Returns: { success, transaction_id, points_awarded, new_balance, tier_changed }
```

### Redeem Reward
```typescript
const response = await fetch('/api/loyalty/redeem', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    reward_id: 'reward-uuid',
    notes: 'For upcoming trip',
  }),
});

const result = await response.json();
// Returns: { success, redemption_id, redemption_code, points_spent, new_balance }
```

### Get Dashboard Data
```typescript
const response = await fetch('/api/loyalty/account');
const dashboard = await response.json();
// Returns complete dashboard data including account, tiers, transactions, rewards
```

## Database Functions Reference

### award_loyalty_points
Awards points to a user and creates transaction record.
```sql
SELECT award_loyalty_points(
  account_user_id := 'user-uuid',
  points_amount := 1000,
  transaction_type := 'earn',
  transaction_description := 'Booking points',
  booking_ref_id := 'booking-uuid'
);
```

### redeem_loyalty_points
Deducts points and creates redemption transaction.
```sql
SELECT redeem_loyalty_points(
  account_user_id := 'user-uuid',
  points_amount := 5000,
  transaction_description := 'Redeemed $50 voucher',
  ref_id := 'reward-uuid'
);
```

### update_loyalty_tier
Updates user's tier based on current points.
```sql
SELECT update_loyalty_tier('user-uuid');
```

### calculate_loyalty_points
Calculates points for an amount.
```sql
SELECT calculate_loyalty_points(100.00, 1.5);
-- Returns: 1500 points (100 * 10 * 1.5)
```

## Security Considerations

1. **RLS Policies** - All tables have Row Level Security enabled
2. **User Isolation** - Users can only see their own data
3. **Public Tiers** - Tier information is public (no sensitive data)
4. **Auth Required** - All endpoints require authentication except /tiers
5. **Validation** - Points calculations validated server-side

## Monitoring & Analytics

### Key Metrics to Track:
- Total points issued
- Points redemption rate
- Tier distribution
- Popular rewards
- Average points per booking
- Member engagement rate

### Useful Queries:
```sql
-- Total points in circulation
SELECT SUM(total_points) FROM loyalty_accounts;

-- Tier distribution
SELECT
  lt.name,
  COUNT(la.id) as member_count
FROM loyalty_accounts la
JOIN loyalty_tiers lt ON la.current_tier_id = lt.id
GROUP BY lt.name;

-- Top rewards
SELECT
  lr.name,
  COUNT(lrd.id) as redemption_count
FROM loyalty_redemptions lrd
JOIN loyalty_rewards lr ON lrd.reward_id = lr.id
GROUP BY lr.name
ORDER BY redemption_count DESC;
```

## Customization

### Adding New Tiers
Update the migration or insert directly:
```sql
INSERT INTO loyalty_tiers (name, min_points, max_points, benefits, multiplier, color)
VALUES ('Diamond', 25001, NULL, '["Premium benefits"]'::jsonb, 4.00, '#B9F2FF');
```

### Adding New Rewards
```sql
INSERT INTO loyalty_rewards (name, description, points_required, category, is_active)
VALUES (
  'Luxury Suite Upgrade',
  'Upgrade to our best suite',
  15000,
  'upgrade',
  true
);
```

### Modifying Point Calculation
Edit `calculatePoints` function in `lib/loyalty/points.ts`:
```typescript
export function calculatePoints(amount: number, tierMultiplier: number = 1.0) {
  // Modify base rate or calculation logic here
  const basePoints = Math.floor(amount * 10); // Change multiplier
  const finalPoints = Math.floor(basePoints * tierMultiplier);
  return { base_points: basePoints, final_points: finalPoints, ... };
}
```

## Testing

### Test Scenarios:
1. ✓ Create new user account
2. ✓ Award points after booking
3. ✓ Check tier progression
4. ✓ Redeem reward
5. ✓ View transaction history
6. ✓ Check insufficient points handling
7. ✓ Test tier benefits
8. ✓ Test reward limits

## Troubleshooting

### Points Not Awarded
- Check booking confirmation flow calls `awardPoints()`
- Verify user_id exists in loyalty_accounts
- Check database logs for errors

### Tier Not Updating
- Run `update_loyalty_tier(user_id)` manually
- Check points threshold in loyalty_tiers table
- Verify RLS policies allow updates

### Redemption Failing
- Check user has sufficient points
- Verify reward is active (is_active = true)
- Check max redemptions limit not reached

## Support

For issues or questions:
1. Check database logs in Supabase dashboard
2. Review API error responses
3. Verify RLS policies are correctly set
4. Check TypeScript types match database schema

## Next Steps

1. **Email Notifications** - Add email alerts for tier changes and redemptions
2. **Push Notifications** - Mobile notifications for points earned
3. **Expiration Policy** - Implement point expiration after 12 months
4. **Referral Program** - Bonus points for referring friends
5. **Special Promotions** - Double points events
6. **Partner Rewards** - Integration with external reward partners
7. **Admin Dashboard** - Manage rewards and view analytics
8. **Gamification** - Badges and achievements
