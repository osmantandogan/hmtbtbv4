# HealMedy Travel Analytics System

A comprehensive analytics dashboard for tracking user behavior, revenue, bookings, and platform performance.

## Features

- Real-time event tracking
- Advanced dashboard with KPIs and visualizations
- Revenue and booking analytics
- User activity tracking
- Geographic analytics
- Device and browser analytics
- Conversion funnel tracking
- Date range filtering (today, week, month, quarter, year)

## Setup Instructions

### 1. Run Database Migration

Execute the SQL migration to create the analytics schema:

```bash
# Using Supabase CLI
supabase db push

# Or run the migration file directly in Supabase SQL Editor
# File: supabase/migrations/20250103_analytics.sql
```

### 2. Install Dependencies

The analytics system uses Recharts for data visualization:

```bash
npm install recharts
# or
yarn add recharts
# or
pnpm add recharts
```

### 3. Configure Environment Variables

Ensure your `.env.local` file has the necessary Supabase credentials:

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
```

## Usage

### Client-Side Event Tracking

Import and use the analytics tracker in your components:

```typescript
import { analytics } from '@/lib/analytics/tracker';

// Track page views (automatic with auto_page_views: true)
analytics.trackPageView('Homepage');

// Track search
analytics.trackSearch('Istanbul hotels', {
  destination: 'Istanbul',
  country: 'Turkey',
  serviceType: 'hotel',
  dateFrom: '2025-06-01',
  dateTo: '2025-06-07',
}, 25);

// Track service view
analytics.trackServiceView(
  'hotel-123',
  'Luxury Hotel Istanbul',
  'hotel',
  299.99
);

// Track booking started
analytics.trackBookingStarted('booking-456', 'hotel', 299.99);

// Track booking completed
analytics.trackBookingCompleted('booking-456', 'hotel', 299.99, 2);

// Track payment
analytics.trackPaymentCompleted(
  'txn-789',
  299.99,
  'USD',
  'credit_card'
);

// Track custom events
analytics.track({
  event: 'filter_applied',
  data: {
    filter_type: 'price',
    min_price: 100,
    max_price: 500,
  },
});
```

### Using the Analytics Dashboard

#### Option 1: Use the Complete Dashboard Component

```typescript
import { AnalyticsDashboard } from '@/components/analytics';

export default function AnalyticsPage() {
  return <AnalyticsDashboard initialDateRange="month" />;
}
```

#### Option 2: Use Individual Components

```typescript
import {
  KPICard,
  RevenueChart,
  BookingsChart,
  UserActivityChart,
  TopDestinations,
} from '@/components/analytics';

export default function CustomDashboard() {
  const [metrics, setMetrics] = useState(null);
  const [revenueData, setRevenueData] = useState([]);

  // Fetch data...

  return (
    <div>
      <KPICard
        title="Total Revenue"
        value={metrics?.total_revenue || 0}
        format="currency"
        icon={<DollarSign />}
      />

      <RevenueChart data={revenueData} />
      <BookingsChart data={bookingsData} />
      <UserActivityChart data={activityData} />
      <TopDestinations data={destinations} />
    </div>
  );
}
```

## API Endpoints

### POST /api/analytics/track
Track analytics events from the client.

**Request Body:**
```json
{
  "events": [
    {
      "session_id": "session-123",
      "event_type": "page_view",
      "event_data": {
        "page_title": "Homepage",
        "page_path": "/"
      }
    }
  ]
}
```

### GET /api/analytics/dashboard
Get dashboard metrics.

**Query Parameters:**
- `start`: ISO date string (default: 30 days ago)
- `end`: ISO date string (default: now)

**Response:**
```json
{
  "success": true,
  "data": {
    "total_users": 1500,
    "total_sessions": 3200,
    "total_pageviews": 12500,
    "total_searches": 850,
    "total_bookings": 125,
    "total_revenue": 37500,
    "avg_order_value": 300,
    "conversion_rate": 14.7,
    "bounce_rate": 35.2,
    "avg_session_duration": 245
  }
}
```

### GET /api/analytics/revenue
Get revenue data over time.

**Query Parameters:**
- `start`: ISO date string
- `end`: ISO date string
- `granularity`: 'hour' | 'day' | 'week' | 'month' (default: 'day')

### GET /api/analytics/bookings
Get bookings by service type.

**Query Parameters:**
- `start`: ISO date string
- `end`: ISO date string

### GET /api/analytics/users
Get user activity timeline.

**Query Parameters:**
- `start`: ISO date string
- `end`: ISO date string
- `granularity`: 'hour' | 'day' | 'week' | 'month' (default: 'day')

### GET /api/analytics/destinations
Get top destinations.

**Query Parameters:**
- `start`: ISO date string
- `end`: ISO date string
- `limit`: number (default: 10)

## Database Schema

### analytics_events Table

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| user_id | UUID | Reference to auth.users (nullable) |
| session_id | TEXT | Session identifier |
| event_type | TEXT | Type of event |
| event_data | JSONB | Event-specific data |
| page_url | TEXT | Page URL |
| referrer | TEXT | Referrer URL |
| ip_address | INET | User IP address |
| user_agent | TEXT | Browser user agent |
| device_type | TEXT | Device type (mobile/tablet/desktop) |
| browser | TEXT | Browser name |
| os | TEXT | Operating system |
| country | TEXT | Country |
| city | TEXT | City |
| created_at | TIMESTAMP | Event timestamp |

### Views

- `analytics_daily_metrics`: Daily aggregated metrics
- `analytics_revenue_by_date`: Revenue by date
- `analytics_top_services`: Top services by views and bookings
- `analytics_top_destinations`: Top destinations by searches
- `analytics_activity_by_hour`: User activity by hour
- `analytics_by_device`: Analytics by device type
- `analytics_by_location`: Analytics by geographic location

### Functions

- `get_dashboard_metrics(start_date, end_date)`: Get all dashboard metrics
- `get_revenue_over_time(start_date, end_date, granularity)`: Get revenue timeline
- `get_bookings_by_service_type(start_date, end_date)`: Get bookings breakdown
- `get_user_activity_timeline(start_date, end_date, granularity)`: Get user activity
- `get_funnel_analytics(start_date, end_date)`: Get conversion funnel data

## Event Types

- `page_view`: User views a page
- `search`: User performs a search
- `service_viewed`: User views a service detail
- `booking_started`: User starts booking process
- `booking_completed`: User completes a booking
- `payment_initiated`: User initiates payment
- `payment_completed`: Payment is completed
- `user_registered`: New user registration
- `user_login`: User logs in
- `filter_applied`: User applies filters
- `share_clicked`: User clicks share
- `contact_submitted`: Contact form submitted
- `error_occurred`: Error tracking

## Configuration

Configure the analytics tracker:

```typescript
import { analytics } from '@/lib/analytics/tracker';

analytics.setConfig({
  enabled: true,
  debug: process.env.NODE_ENV === 'development',
  auto_page_views: true,
  session_timeout: 30, // minutes
  batch_events: true,
  batch_size: 10,
  batch_interval: 5000, // milliseconds
});
```

## Security

- Row Level Security (RLS) policies are enabled
- Anonymous users can insert events (for tracking)
- Only authenticated users can insert events with user_id
- Only admins can view analytics data
- Service role has full access for backend operations

## Performance Optimization

- Indexed columns for fast queries
- JSONB for flexible event data
- Materialized views for aggregated statistics
- Batch event processing
- Efficient database functions

## Best Practices

1. **Track Meaningful Events**: Focus on events that provide business insights
2. **Enrich Event Data**: Include relevant context in event_data
3. **Respect Privacy**: Don't track PII without consent
4. **Monitor Performance**: Keep track of database query performance
5. **Regular Cleanup**: Archive old analytics data periodically

## Troubleshooting

### Events Not Appearing

1. Check browser console for errors
2. Verify API endpoint is accessible
3. Check Supabase RLS policies
4. Ensure user has proper permissions

### Dashboard Not Loading

1. Verify date range parameters
2. Check API responses in Network tab
3. Ensure database functions are created
4. Verify user authentication

### Slow Performance

1. Check database indexes
2. Reduce date range
3. Use appropriate granularity
4. Consider caching frequently accessed data

## Future Enhancements

- Real-time analytics with websockets
- Custom report builder
- Email reports and alerts
- A/B testing integration
- Cohort analysis
- Predictive analytics
- Export to CSV/PDF
- Custom dashboards per user role

## Support

For issues and questions, please refer to the project documentation or contact the development team.
