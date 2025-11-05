# Price Alerts System - Complete Guide

## Overview

The HealMedy Travel Price Alerts system allows users to track prices for hotels, flights, tours, and transfers. Users receive email notifications when prices drop to their target levels.

## File Structure

```
healmedy-travel-v2/
├── supabase/migrations/
│   └── 20250103_price_alerts.sql         # Database schema
├── lib/
│   ├── types/
│   │   └── price-alerts.ts               # TypeScript types and utilities
│   └── price-alerts/
│       └── checker.ts                    # Price checking service
├── components/price-alerts/
│   ├── CreateAlertButton.tsx             # Button to create alerts
│   ├── CreateAlertModal.tsx              # Modal form for alert creation
│   ├── AlertCard.tsx                     # Single alert display card
│   └── AlertsList.tsx                    # List view of all alerts
└── app/api/price-alerts/
    ├── route.ts                          # GET (list), POST (create)
    ├── [id]/route.ts                     # GET, PUT, DELETE single alert
    └── check/route.ts                    # Cron job for price checking
```

## Database Schema

### Tables

#### `price_alerts`
Main table storing price alert configurations.

**Columns:**
- `id` (UUID) - Primary key
- `user_id` (UUID) - References auth.users
- `service_type` (TEXT) - hotel/flight/tour/transfer
- `service_id` (TEXT) - External service identifier
- `service_name` (TEXT) - Display name
- `destination` (TEXT) - Destination location
- `origin` (TEXT) - Origin location (for flights)
- `check_in_date` (DATE) - For hotels
- `check_out_date` (DATE) - For hotels
- `departure_date` (DATE) - For flights
- `return_date` (DATE) - For flights
- `guests` (INTEGER) - Number of guests
- `target_price` (NUMERIC) - User's target price
- `current_price` (NUMERIC) - Last checked price
- `currency` (TEXT) - Default 'USD'
- `alert_frequency` (TEXT) - once/daily/weekly
- `is_active` (BOOLEAN) - Alert status
- `last_checked_at` (TIMESTAMPTZ) - Last price check
- `last_notified_at` (TIMESTAMPTZ) - Last notification sent
- `notification_count` (INTEGER) - Total notifications sent
- `metadata` (JSONB) - Additional data
- `created_at`, `updated_at` (TIMESTAMPTZ)

**Indexes:**
- `idx_price_alerts_user_id` - Fast user lookup
- `idx_price_alerts_service_type` - Filter by service
- `idx_price_alerts_is_active` - Active alerts
- `idx_price_alerts_last_checked` - Scheduled checks
- `idx_price_alerts_destination` - Location search
- `idx_price_alerts_user_active` - User's active alerts

#### `price_alert_notifications`
History of all notifications sent.

**Columns:**
- `id` (UUID) - Primary key
- `alert_id` (UUID) - References price_alerts
- `user_id` (UUID) - References auth.users
- `previous_price` (NUMERIC) - Price before change
- `new_price` (NUMERIC) - Current price
- `price_drop` (NUMERIC) - Amount saved
- `notification_type` (TEXT) - price_drop/target_reached/price_increase
- `sent_at` (TIMESTAMPTZ) - When notification was sent
- `email_sent` (BOOLEAN) - Email delivery status
- `read_at` (TIMESTAMPTZ) - When user viewed notification

### Views

#### `active_price_alerts_to_check`
Filtered view of alerts that need price checking based on frequency.

### Functions

#### `deactivate_expired_alerts()`
Automatically deactivates alerts for past dates.

### Row Level Security (RLS)

All tables have RLS enabled:
- Users can only view/manage their own alerts
- Service role has full access (for cron jobs)

## API Endpoints

### GET `/api/price-alerts`
List user's price alerts with filtering.

**Query Parameters:**
- `service_type` - Filter by service type
- `is_active` - Filter by active status
- `destination` - Search by destination
- `limit` - Number of results (default: 50)
- `offset` - Pagination offset (default: 0)

**Response:**
```json
{
  "alerts": [...],
  "count": 10,
  "limit": 50,
  "offset": 0
}
```

### POST `/api/price-alerts`
Create a new price alert.

**Request Body:**
```json
{
  "service_type": "hotel",
  "service_id": "hotel-123",
  "service_name": "Grand Hotel Istanbul",
  "destination": "Istanbul, Turkey",
  "check_in_date": "2025-06-01",
  "check_out_date": "2025-06-05",
  "guests": 2,
  "target_price": 150.00,
  "current_price": 180.00,
  "currency": "USD",
  "alert_frequency": "daily"
}
```

**Response:**
```json
{
  "alert": { ... }
}
```

### GET `/api/price-alerts/[id]`
Get a specific price alert.

**Response:**
```json
{
  "alert": { ... }
}
```

### PUT `/api/price-alerts/[id]`
Update a price alert.

**Request Body:**
```json
{
  "target_price": 140.00,
  "alert_frequency": "weekly",
  "is_active": false
}
```

### DELETE `/api/price-alerts/[id]`
Delete a price alert.

**Response:**
```json
{
  "success": true,
  "message": "Price alert deleted successfully"
}
```

### POST `/api/price-alerts/check`
Cron job endpoint to check all active alerts.

**Headers:**
```
Authorization: Bearer <CRON_SECRET>
```

**Response:**
```json
{
  "success": true,
  "results": {
    "total_checked": 45,
    "alerts_with_changes": 12,
    "notifications_sent": 8,
    "errors": []
  },
  "timestamp": "2025-01-03T10:00:00Z"
}
```

## React Components

### CreateAlertButton
Button component to trigger alert creation.

**Usage:**
```tsx
import CreateAlertButton from '@/components/price-alerts/CreateAlertButton';

<CreateAlertButton
  data={{
    service_type: 'hotel',
    service_id: 'hotel-123',
    service_name: 'Grand Hotel',
    destination: 'Istanbul',
    price: 180,
    currency: 'USD',
    dates: {
      checkIn: '2025-06-01',
      checkOut: '2025-06-05'
    },
    guests: 2
  }}
  variant="primary"
  size="md"
/>
```

### CreateAlertModal
Modal form for creating price alerts.

**Features:**
- Pre-fills service information
- Suggests target price (10% below current)
- Shows potential savings
- Frequency selection (once/daily/weekly)

### AlertCard
Displays a single price alert with actions.

**Features:**
- Service type icon and name
- Current vs target price comparison
- Savings calculation
- Toggle active/inactive
- Delete functionality
- "Book Now" button when target is reached

### AlertsList
Complete list view with filtering and search.

**Features:**
- Search by destination/service name
- Filter by status (active/inactive)
- Filter by service type
- Stats display (active/inactive counts)
- Empty state handling
- Loading states

**Usage:**
```tsx
import AlertsList from '@/components/price-alerts/AlertsList';

// Show all alerts with filters
<AlertsList showFilters={true} />

// Show limited alerts (e.g., dashboard widget)
<AlertsList showFilters={false} limit={5} />
```

## Price Checking Service

### Core Functions

#### `getCurrentPrice(alert: PriceAlert): Promise<number | null>`
Fetches current price from external APIs based on service type.

**TODO:** Implement actual API integrations for:
- Hotels: `/api/hotels/search`
- Flights: `/api/flights/search`
- Tours: `/api/tours/search`
- Transfers: `/api/transfers/search`

#### `checkPriceAlert(alert: PriceAlert): Promise<PriceCheckResult>`
Checks a single alert and determines if notification is needed.

**Logic:**
1. Verify alert should be checked (based on frequency)
2. Fetch current price from API
3. Compare with previous price
4. Check if target price is reached
5. Update alert in database

#### `processAlert(alert: PriceAlert): Promise<{ success, notified, error }>`
Complete processing including notification sending.

#### `checkAllAlerts(): Promise<BulkCheckResponse>`
Checks all active alerts that need checking.

**Process:**
1. Deactivate expired alerts
2. Fetch alerts from `active_price_alerts_to_check` view
3. Process alerts sequentially (with 1s delay)
4. Send email notifications for price changes
5. Return summary statistics

### Email Notifications

Emails are sent using Resend API when:
- Target price is reached
- Price drops below previous check
- Frequency conditions are met

**Email includes:**
- Service details (name, destination, dates)
- Price comparison (old vs new)
- Savings amount and percentage
- "Book Now" call-to-action
- Alert management link

## Setup Instructions

### 1. Database Migration

Run the migration to create tables:

```bash
# Using Supabase CLI
supabase migration up

# Or apply directly in Supabase Studio
# Copy contents of 20250103_price_alerts.sql
```

### 2. Environment Variables

Add to `.env.local`:

```env
# Supabase (already configured)
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key

# Email (already configured)
RESEND_API_KEY=your_resend_api_key
EMAIL_FROM=HealMedy Travel <noreply@healmedy.com>

# Cron Job Security (NEW)
CRON_SECRET=generate_a_random_secure_token_here
```

Generate CRON_SECRET:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### 3. Implement API Integrations

Update `lib/price-alerts/checker.ts` to call your actual search APIs:

```typescript
// Example for hotel price checking
async function checkHotelPrice(alert: PriceAlert): Promise<number | null> {
  const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/hotels/search`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      hotelId: alert.service_id,
      checkIn: alert.check_in_date,
      checkOut: alert.check_out_date,
      guests: alert.guests,
    }),
  });

  const data = await response.json();
  return data.price;
}
```

### 4. Set Up Cron Job

#### Option A: Vercel Cron (Recommended)

Create `vercel.json`:
```json
{
  "crons": [
    {
      "path": "/api/price-alerts/check",
      "schedule": "0 */6 * * *"
    }
  ]
}
```

#### Option B: External Cron Service

Use services like:
- Cron-job.org
- EasyCron
- GitHub Actions

Example GitHub Actions workflow:

```yaml
# .github/workflows/price-check.yml
name: Price Alerts Check
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours
  workflow_dispatch:  # Manual trigger

jobs:
  check-prices:
    runs-on: ubuntu-latest
    steps:
      - name: Check Price Alerts
        run: |
          curl -X POST https://your-domain.com/api/price-alerts/check \
            -H "Authorization: Bearer ${{ secrets.CRON_SECRET }}"
```

### 5. Integration Examples

#### Add to Hotel Detail Page

```tsx
// app/hotels/[id]/page.tsx
import CreateAlertButton from '@/components/price-alerts/CreateAlertButton';

export default function HotelDetailPage({ hotel, searchParams }) {
  return (
    <div>
      {/* Hotel details */}

      <CreateAlertButton
        data={{
          service_type: 'hotel',
          service_id: hotel.id,
          service_name: hotel.name,
          destination: hotel.location,
          price: hotel.price,
          currency: 'USD',
          dates: {
            checkIn: searchParams.checkIn,
            checkOut: searchParams.checkOut
          },
          guests: searchParams.guests
        }}
      />
    </div>
  );
}
```

#### Add to Dashboard

```tsx
// app/dashboard/page.tsx
import AlertsList from '@/components/price-alerts/AlertsList';

export default function DashboardPage() {
  return (
    <div>
      <h1>My Dashboard</h1>

      {/* Show recent alerts */}
      <AlertsList showFilters={false} limit={5} />

      <a href="/dashboard/price-alerts">View All Alerts</a>
    </div>
  );
}
```

#### Create Dedicated Alerts Page

```tsx
// app/dashboard/price-alerts/page.tsx
import AlertsList from '@/components/price-alerts/AlertsList';

export default function PriceAlertsPage() {
  return (
    <div className="container mx-auto py-8">
      <AlertsList showFilters={true} />
    </div>
  );
}
```

## Testing

### Manual Testing

1. **Create an Alert:**
   - Navigate to hotel/flight detail page
   - Click "Set Price Alert"
   - Fill in target price
   - Submit form

2. **View Alerts:**
   - Go to `/dashboard/price-alerts`
   - Verify alert appears in list

3. **Test Price Check:**
   ```bash
   curl -X POST http://localhost:3000/api/price-alerts/check \
     -H "Authorization: Bearer YOUR_CRON_SECRET"
   ```

4. **Check Email:**
   - Verify email delivery in Resend dashboard
   - Check spam folder if not received

### Database Queries

Check alerts:
```sql
SELECT * FROM price_alerts WHERE user_id = 'your-user-id';
```

Check notifications:
```sql
SELECT * FROM price_alert_notifications ORDER BY sent_at DESC LIMIT 10;
```

View alerts to check:
```sql
SELECT * FROM active_price_alerts_to_check WHERE should_check = true;
```

## Monitoring and Maintenance

### Key Metrics to Monitor

1. **Alert Statistics:**
   - Total active alerts
   - Alerts checked per day
   - Average response time

2. **Notification Metrics:**
   - Notifications sent per day
   - Email delivery rate
   - Click-through rate

3. **Error Tracking:**
   - Failed API calls
   - Email sending failures
   - Database errors

### Maintenance Tasks

1. **Weekly:**
   - Review error logs
   - Check email delivery rates
   - Monitor API performance

2. **Monthly:**
   - Clean up old notifications (>90 days)
   - Analyze user engagement
   - Update price checking intervals

3. **Quarterly:**
   - Review and optimize database indexes
   - Update email templates
   - Refine notification logic

## Troubleshooting

### Common Issues

**1. Alerts Not Being Checked**
- Verify cron job is running
- Check `CRON_SECRET` configuration
- Review `last_checked_at` timestamps

**2. No Email Notifications**
- Verify Resend API key
- Check email delivery logs
- Ensure `EMAIL_FROM` is verified

**3. Authentication Errors**
- Verify Supabase configuration
- Check RLS policies
- Ensure service role key is correct

**4. Price Fetching Fails**
- Implement error handling in API calls
- Add retry logic
- Log failed requests

## Future Enhancements

### Planned Features

1. **Push Notifications**
   - Browser push notifications
   - Mobile app notifications

2. **Advanced Filtering**
   - Price range alerts
   - Percentage-based targets
   - Flexible date ranges

3. **Analytics Dashboard**
   - Historical price charts
   - Best time to book insights
   - Savings statistics

4. **Social Features**
   - Share alerts with friends
   - Alert recommendations
   - Popular destinations tracking

5. **Smart Alerts**
   - ML-based price predictions
   - Automatic target adjustment
   - Seasonal pricing patterns

## Support

For issues or questions:
- GitHub Issues: [repository-url]
- Email: support@healmedy.com
- Documentation: [docs-url]

## License

Copyright © 2025 HealMedy Travel. All rights reserved.
