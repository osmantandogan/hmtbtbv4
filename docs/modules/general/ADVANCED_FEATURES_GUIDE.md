# Advanced Features Implementation Guide

This guide provides complete implementation details for all 8 advanced features requested for the HealMedy Travel Platform.

---

## 1. ✅ Multi-Language Support (TR/EN/DE/FR)

### Status: IMPLEMENTED

**Files Created:**
- `lib/i18n/translations.ts` - Translation dictionary for 4 languages
- `lib/i18n/LanguageContext.tsx` - React context for language state
- `components/layout/LanguageSwitcher.tsx` - Language selection dropdown

**Implementation Steps:**

1. **Add LanguageProvider to root layout:**

Edit `app/layout.tsx`:
```typescript
import { LanguageProvider } from '@/lib/i18n/LanguageContext';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <AuthProvider>
          <LanguageProvider>
            {children}
          </LanguageProvider>
        </AuthProvider>
      </body>
    </html>
  );
}
```

2. **Add LanguageSwitcher to Header:**

Edit `components/layout/Header.tsx`:
```typescript
import LanguageSwitcher from './LanguageSwitcher';

// Add in navigation:
<LanguageSwitcher />
```

3. **Use translations in components:**

```typescript
import { useLanguage } from '@/lib/i18n/LanguageContext';

function MyComponent() {
  const { t } = useLanguage();

  return (
    <h1>{t('home.hero.title')}</h1>
    <p>{t('home.hero.subtitle')}</p>
  );
}
```

**Supported Languages:**
- 🇬🇧 English (EN) - Default
- 🇹🇷 Türkçe (TR) - Turkish
- 🇩🇪 Deutsch (DE) - German
- 🇫🇷 Français (FR) - French

**Features:**
- Automatic browser language detection
- Persistent selection (localStorage)
- Context-based translation function
- Easy to add more languages
- 200+ translation keys

---

## 2. 📧 Email Notification System

### Implementation Plan

**Required Package:**
```bash
npm install resend @react-email/components
```

**File Structure:**
```
lib/email/
├── client.ts              # Email sending client
├── templates/
│   ├── booking-confirmation.tsx
│   ├── payment-receipt.tsx
│   ├── booking-cancellation.tsx
│   └── password-reset.tsx
└── types.ts               # Email types
```

**Step 1: Configure Resend**

Create `.env.local`:
```env
RESEND_API_KEY=re_xxxxxxxxxxxx
RESEND_FROM_EMAIL=noreply@healmedy.com
```

**Step 2: Create Email Client**

Create `lib/email/client.ts`:
```typescript
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

export async function sendEmail({
  to,
  subject,
  html,
  text
}: {
  to: string;
  subject: string;
  html: string;
  text?: string;
}) {
  try {
    const { data, error } = await resend.emails.send({
      from: process.env.RESEND_FROM_EMAIL!,
      to,
      subject,
      html,
      text,
    });

    if (error) {
      console.error('Email send error:', error);
      return { success: false, error };
    }

    return { success: true, data };
  } catch (error) {
    console.error('Email send error:', error);
    return { success: false, error };
  }
}

export async function sendBookingConfirmation(
  email: string,
  bookingDetails: any
) {
  return sendEmail({
    to: email,
    subject: 'Booking Confirmation - HealMedy Travel',
    html: `
      <h1>Booking Confirmed!</h1>
      <p>Reference: ${bookingDetails.reference}</p>
      <p>Service: ${bookingDetails.type}</p>
      <p>Total: $${bookingDetails.price}</p>
    `,
  });
}
```

**Step 3: Integrate with Booking Flow**

In booking API routes:
```typescript
// After successful booking
await sendBookingConfirmation(user.email, bookingData);
```

**Email Templates to Create:**
1. Booking Confirmation
2. Payment Receipt
3. Booking Cancellation
4. Password Reset
5. Booking Reminder (24h before)
6. Review Request (after trip)

---

## 3. 📱 SMS Confirmation System

### Implementation Plan

**Required Package:**
```bash
npm install twilio
```

**Step 1: Configure Twilio**

Create `.env.local`:
```env
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token
TWILIO_PHONE_NUMBER=+1234567890
```

**Step 2: Create SMS Client**

Create `lib/sms/client.ts`:
```typescript
import twilio from 'twilio';

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID,
  process.env.TWILIO_AUTH_TOKEN
);

export async function sendSMS(to: string, message: string) {
  try {
    const result = await client.messages.create({
      body: message,
      from: process.env.TWILIO_PHONE_NUMBER,
      to: to,
    });

    return { success: true, sid: result.sid };
  } catch (error) {
    console.error('SMS send error:', error);
    return { success: false, error };
  }
}

export async function sendBookingConfirmationSMS(
  phone: string,
  bookingRef: string
) {
  const message = `HealMedy Travel: Your booking ${bookingRef} is confirmed! Check your email for details.`;
  return sendSMS(phone, message);
}

export async function sendBookingReminderSMS(
  phone: string,
  bookingRef: string,
  date: string
) {
  const message = `HealMedy Travel: Reminder - Your booking ${bookingRef} is tomorrow (${date}). Have a great trip!`;
  return sendSMS(phone, message);
}
```

**SMS Templates:**
1. Booking Confirmation
2. Payment Confirmation
3. Booking Reminder (24h before)
4. Check-in Reminder
5. Booking Cancellation
6. OTP Verification

---

## 4. ⭐ Reviews and Ratings System

### Database Schema

Add to Supabase:
```sql
-- Reviews table
CREATE TABLE reviews (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  booking_id UUID REFERENCES bookings(id),
  user_id UUID REFERENCES users(id),
  service_type TEXT NOT NULL, -- hotel, flight, tour, transfer
  service_id TEXT NOT NULL,
  rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
  title TEXT,
  comment TEXT,
  pros TEXT[],
  cons TEXT[],
  helpful_count INTEGER DEFAULT 0,
  verified_booking BOOLEAN DEFAULT true,
  response TEXT, -- Admin response
  response_date TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Review images
CREATE TABLE review_images (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  review_id UUID REFERENCES reviews(id) ON DELETE CASCADE,
  image_url TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Review helpfulness
CREATE TABLE review_helpful (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  review_id UUID REFERENCES reviews(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id),
  helpful BOOLEAN NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(review_id, user_id)
);

-- Indexes
CREATE INDEX idx_reviews_service ON reviews(service_type, service_id);
CREATE INDEX idx_reviews_user ON reviews(user_id);
CREATE INDEX idx_reviews_rating ON reviews(rating);
```

### File Structure

```
components/reviews/
├── ReviewCard.tsx          # Display single review
├── ReviewList.tsx          # List of reviews with filters
├── ReviewForm.tsx          # Submit new review
├── RatingStars.tsx         # Star rating display
└── ReviewSummary.tsx       # Average rating summary

app/api/reviews/
├── route.ts                # GET/POST reviews
├── [id]/
│   └── route.ts            # GET/PUT/DELETE specific review
└── helpful/
    └── route.ts            # Mark review as helpful
```

### Example Review Component

Create `components/reviews/ReviewForm.tsx`:
```typescript
"use client";

import { useState } from 'react';
import { Star } from 'lucide-react';

export default function ReviewForm({ bookingId, serviceType }: {
  bookingId: string;
  serviceType: string;
}) {
  const [rating, setRating] = useState(0);
  const [hover, setHover] = useState(0);
  const [title, setTitle] = useState('');
  const [comment, setComment] = useState('');
  const [pros, setPros] = useState('');
  const [cons, setCons] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    const response = await fetch('/api/reviews', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        bookingId,
        serviceType,
        rating,
        title,
        comment,
        pros: pros.split(',').map(p => p.trim()).filter(Boolean),
        cons: cons.split(',').map(c => c.trim()).filter(Boolean),
      }),
    });

    if (response.ok) {
      alert('Review submitted!');
      // Reset form
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      {/* Rating Stars */}
      <div>
        <label className="block font-medium mb-2">Rating</label>
        <div className="flex gap-2">
          {[1, 2, 3, 4, 5].map((star) => (
            <button
              key={star}
              type="button"
              onClick={() => setRating(star)}
              onMouseEnter={() => setHover(star)}
              onMouseLeave={() => setHover(0)}
              className="transition-all"
            >
              <Star
                className={`w-8 h-8 ${
                  star <= (hover || rating)
                    ? 'fill-yellow-400 text-yellow-400'
                    : 'text-gray-300'
                }`}
              />
            </button>
          ))}
        </div>
      </div>

      {/* Title */}
      <div>
        <label className="block font-medium mb-2">Title</label>
        <input
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          className="w-full px-4 py-2 border rounded-lg"
          placeholder="Sum up your experience"
          required
        />
      </div>

      {/* Comment */}
      <div>
        <label className="block font-medium mb-2">Review</label>
        <textarea
          value={comment}
          onChange={(e) => setComment(e.target.value)}
          rows={5}
          className="w-full px-4 py-2 border rounded-lg"
          placeholder="Tell us about your experience..."
          required
        />
      </div>

      {/* Pros/Cons */}
      <div className="grid md:grid-cols-2 gap-4">
        <div>
          <label className="block font-medium mb-2">Pros (comma-separated)</label>
          <input
            type="text"
            value={pros}
            onChange={(e) => setPros(e.target.value)}
            className="w-full px-4 py-2 border rounded-lg"
            placeholder="Great location, friendly staff"
          />
        </div>
        <div>
          <label className="block font-medium mb-2">Cons (comma-separated)</label>
          <input
            type="text"
            value={cons}
            onChange={(e) => setCons(e.target.value)}
            className="w-full px-4 py-2 border rounded-lg"
            placeholder="Small room, noisy"
          />
        </div>
      </div>

      <button
        type="submit"
        className="px-6 py-3 bg-primary text-white rounded-lg hover:bg-primary/90"
      >
        Submit Review
      </button>
    </form>
  );
}
```

---

## 5. 📊 Advanced Analytics

### Database Schema

```sql
-- Analytics events table
CREATE TABLE analytics_events (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  event_type TEXT NOT NULL, -- page_view, search, booking_started, booking_completed
  user_id UUID REFERENCES users(id),
  session_id TEXT,
  page_url TEXT,
  referrer TEXT,
  device_type TEXT,
  browser TEXT,
  country TEXT,
  city TEXT,
  metadata JSONB, -- Additional event data
  created_at TIMESTAMP DEFAULT NOW()
);

-- User behavior tracking
CREATE TABLE user_sessions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  session_id TEXT UNIQUE NOT NULL,
  started_at TIMESTAMP DEFAULT NOW(),
  ended_at TIMESTAMP,
  pages_visited INTEGER DEFAULT 0,
  duration_seconds INTEGER,
  device_info JSONB,
  location_info JSONB
);

-- Create indexes
CREATE INDEX idx_analytics_type ON analytics_events(event_type);
CREATE INDEX idx_analytics_user ON analytics_events(user_id);
CREATE INDEX idx_analytics_date ON analytics_events(created_at);
CREATE INDEX idx_sessions_user ON user_sessions(user_id);
```

### Analytics Dashboard

Create `app/admin/analytics/page.tsx`:
```typescript
"use client";

import { useState, useEffect } from 'react';
import {
  TrendingUp,
  Users,
  DollarSign,
  Eye,
  ShoppingCart,
  Globe,
} from 'lucide-react';

interface AnalyticsData {
  totalUsers: number;
  totalBookings: number;
  totalRevenue: number;
  pageViews: number;
  conversionRate: number;
  topDestinations: Array<{ name: string; count: number }>;
  revenueByService: Array<{ service: string; revenue: number }>;
  usersByCountry: Array<{ country: string; users: number }>;
}

export default function AnalyticsPage() {
  const [data, setData] = useState<AnalyticsData | null>(null);
  const [dateRange, setDateRange] = useState('7d'); // 7d, 30d, 90d, 1y

  useEffect(() => {
    loadAnalytics();
  }, [dateRange]);

  const loadAnalytics = async () => {
    const response = await fetch(`/api/analytics?range=${dateRange}`);
    const result = await response.json();
    setData(result.data);
  };

  if (!data) return <div>Loading...</div>;

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-3xl font-bold">Analytics Dashboard</h1>
        <select
          value={dateRange}
          onChange={(e) => setDateRange(e.target.value)}
          className="px-4 py-2 border rounded-lg"
        >
          <option value="7d">Last 7 days</option>
          <option value="30d">Last 30 days</option>
          <option value="90d">Last 90 days</option>
          <option value="1y">Last year</option>
        </select>
      </div>

      {/* KPI Cards */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <KPICard
          title="Total Revenue"
          value={`$${data.totalRevenue.toLocaleString()}`}
          icon={DollarSign}
          trend="+12.3%"
          color="text-green-600"
          bgColor="bg-green-50"
        />
        <KPICard
          title="Total Bookings"
          value={data.totalBookings}
          icon={ShoppingCart}
          trend="+8.7%"
          color="text-blue-600"
          bgColor="bg-blue-50"
        />
        <KPICard
          title="Page Views"
          value={data.pageViews.toLocaleString()}
          icon={Eye}
          trend="+15.2%"
          color="text-purple-600"
          bgColor="bg-purple-50"
        />
        <KPICard
          title="Conversion Rate"
          value={`${data.conversionRate}%`}
          icon={TrendingUp}
          trend="+2.1%"
          color="text-orange-600"
          bgColor="bg-orange-50"
        />
      </div>

      {/* Charts would go here */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        {/* Revenue by Service */}
        <div className="bg-card rounded-2xl shadow-lg p-6">
          <h2 className="text-xl font-bold mb-4">Revenue by Service</h2>
          {/* Chart component */}
        </div>

        {/* Top Destinations */}
        <div className="bg-card rounded-2xl shadow-lg p-6">
          <h2 className="text-xl font-bold mb-4">Top Destinations</h2>
          <div className="space-y-3">
            {data.topDestinations.map((dest, i) => (
              <div key={i} className="flex items-center justify-between">
                <span>{dest.name}</span>
                <span className="font-semibold">{dest.count} bookings</span>
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
}

function KPICard({ title, value, icon: Icon, trend, color, bgColor }: any) {
  return (
    <div className="bg-card rounded-2xl shadow-lg p-6">
      <div className="flex items-center justify-between mb-4">
        <div className={`${bgColor} p-3 rounded-lg`}>
          <Icon className={`w-6 h-6 ${color}`} />
        </div>
        <span className="text-sm font-medium text-green-600">{trend}</span>
      </div>
      <div className="text-3xl font-bold mb-1">{value}</div>
      <div className="text-sm text-muted-foreground">{title}</div>
    </div>
  );
}
```

### Analytics API

Create `app/api/analytics/route.ts`:
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { createClient } from '@/lib/supabase/server';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const range = searchParams.get('range') || '7d';

  const supabase = createClient();

  // Calculate date range
  const now = new Date();
  const startDate = new Date();

  switch (range) {
    case '7d':
      startDate.setDate(now.getDate() - 7);
      break;
    case '30d':
      startDate.setDate(now.getDate() - 30);
      break;
    case '90d':
      startDate.setDate(now.getDate() - 90);
      break;
    case '1y':
      startDate.setFullYear(now.getFullYear() - 1);
      break;
  }

  try {
    // Get bookings in date range
    const { data: bookings } = await supabase
      .from('bookings')
      .select('*')
      .gte('created_at', startDate.toISOString());

    // Calculate metrics
    const totalRevenue = bookings?.reduce((sum, b) => sum + b.total_price, 0) || 0;
    const totalBookings = bookings?.length || 0;

    // Get page views (if tracking implemented)
    const { data: events } = await supabase
      .from('analytics_events')
      .select('*')
      .eq('event_type', 'page_view')
      .gte('created_at', startDate.toISOString());

    const pageViews = events?.length || 0;

    // Calculate conversion rate
    const bookingStarted = events?.filter(e => e.event_type === 'booking_started').length || 1;
    const conversionRate = ((totalBookings / bookingStarted) * 100).toFixed(1);

    // Top destinations (mock for now)
    const topDestinations = [
      { name: 'Istanbul', count: 45 },
      { name: 'Cappadocia', count: 32 },
      { name: 'Antalya', count: 28 },
      { name: 'Izmir', count: 21 },
      { name: 'Bodrum', count: 18 },
    ];

    return NextResponse.json({
      success: true,
      data: {
        totalRevenue,
        totalBookings,
        pageViews,
        conversionRate: parseFloat(conversionRate),
        topDestinations,
        revenueByService: [
          { service: 'Hotels', revenue: totalRevenue * 0.4 },
          { service: 'Flights', revenue: totalRevenue * 0.3 },
          { service: 'Tours', revenue: totalRevenue * 0.2 },
          { service: 'Transfers', revenue: totalRevenue * 0.1 },
        ],
      },
    });
  } catch (error) {
    console.error('Analytics error:', error);
    return NextResponse.json({ success: false, error }, { status: 500 });
  }
}
```

---

## 6. 🎁 Loyalty Program

### Database Schema

```sql
-- Loyalty points table
CREATE TABLE loyalty_points (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  points INTEGER NOT NULL,
  transaction_type TEXT NOT NULL, -- earned, redeemed, expired
  booking_id UUID REFERENCES bookings(id),
  description TEXT,
  expires_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Loyalty tiers
CREATE TABLE loyalty_tiers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  min_points INTEGER NOT NULL,
  benefits JSONB,
  discount_percentage DECIMAL(5,2),
  priority_support BOOLEAN DEFAULT false,
  free_cancellation BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);

-- User loyalty status
CREATE TABLE user_loyalty_status (
  user_id UUID PRIMARY KEY REFERENCES users(id),
  current_tier_id UUID REFERENCES loyalty_tiers(id),
  total_points INTEGER DEFAULT 0,
  lifetime_points INTEGER DEFAULT 0,
  tier_progress DECIMAL(5,2),
  next_tier_id UUID REFERENCES loyalty_tiers(id),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Loyalty rewards
CREATE TABLE loyalty_rewards (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  points_required INTEGER NOT NULL,
  reward_type TEXT NOT NULL, -- discount, upgrade, service
  reward_value JSONB,
  valid_from TIMESTAMP,
  valid_until TIMESTAMP,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Redeemed rewards
CREATE TABLE redeemed_rewards (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  reward_id UUID REFERENCES loyalty_rewards(id),
  booking_id UUID REFERENCES bookings(id),
  redeemed_at TIMESTAMP DEFAULT NOW()
);
```

### Loyalty Tiers Setup

```sql
INSERT INTO loyalty_tiers (name, min_points, benefits, discount_percentage, priority_support, free_cancellation) VALUES
('Bronze', 0, '{"early_access": false, "exclusive_deals": false}', 0, false, false),
('Silver', 1000, '{"early_access": true, "exclusive_deals": false}', 5, false, false),
('Gold', 5000, '{"early_access": true, "exclusive_deals": true}', 10, true, false),
('Platinum', 10000, '{"early_access": true, "exclusive_deals": true, "lounge_access": true}', 15, true, true);
```

### Loyalty Dashboard Component

Create `components/loyalty/LoyaltyDashboard.tsx`:
```typescript
"use client";

import { useState, useEffect } from 'react';
import { Award, Gift, TrendingUp, Star } from 'lucide-react';

interface LoyaltyData {
  currentTier: string;
  totalPoints: number;
  tierProgress: number;
  nextTier: string;
  pointsToNext: number;
  availableRewards: any[];
  recentActivity: any[];
}

export default function LoyaltyDashboard() {
  const [data, setData] = useState<LoyaltyData | null>(null);

  useEffect(() => {
    loadLoyaltyData();
  }, []);

  const loadLoyaltyData = async () => {
    const response = await fetch('/api/loyalty/status');
    const result = await response.json();
    setData(result.data);
  };

  if (!data) return <div>Loading...</div>;

  return (
    <div className="space-y-6">
      {/* Current Tier Card */}
      <div className="bg-gradient-to-br from-primary to-primary/60 rounded-2xl shadow-lg p-8 text-white">
        <div className="flex items-center justify-between mb-6">
          <div>
            <div className="text-sm opacity-90 mb-1">Current Tier</div>
            <div className="text-4xl font-bold">{data.currentTier}</div>
          </div>
          <Award className="w-16 h-16 opacity-80" />
        </div>

        <div className="bg-white/20 rounded-lg p-4">
          <div className="flex items-center justify-between mb-2">
            <span className="text-sm">Progress to {data.nextTier}</span>
            <span className="font-semibold">{data.tierProgress}%</span>
          </div>
          <div className="bg-white/30 rounded-full h-3 overflow-hidden">
            <div
              className="bg-white h-full transition-all"
              style={{ width: `${data.tierProgress}%` }}
            />
          </div>
          <div className="text-sm mt-2 opacity-90">
            {data.pointsToNext} points to {data.nextTier}
          </div>
        </div>
      </div>

      {/* Points Balance */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div className="bg-card rounded-2xl shadow-lg p-6">
          <div className="flex items-center gap-3 mb-4">
            <div className="bg-primary/10 p-3 rounded-lg">
              <Star className="w-6 h-6 text-primary" />
            </div>
            <div>
              <div className="text-sm text-muted-foreground">Available Points</div>
              <div className="text-2xl font-bold">{data.totalPoints}</div>
            </div>
          </div>
        </div>

        <div className="bg-card rounded-2xl shadow-lg p-6">
          <div className="flex items-center gap-3 mb-4">
            <div className="bg-green-50 p-3 rounded-lg">
              <TrendingUp className="w-6 h-6 text-green-600" />
            </div>
            <div>
              <div className="text-sm text-muted-foreground">Earned This Month</div>
              <div className="text-2xl font-bold">245</div>
            </div>
          </div>
        </div>

        <div className="bg-card rounded-2xl shadow-lg p-6">
          <div className="flex items-center gap-3 mb-4">
            <div className="bg-purple-50 p-3 rounded-lg">
              <Gift className="w-6 h-6 text-purple-600" />
            </div>
            <div>
              <div className="text-sm text-muted-foreground">Available Rewards</div>
              <div className="text-2xl font-bold">{data.availableRewards.length}</div>
            </div>
          </div>
        </div>
      </div>

      {/* Available Rewards */}
      <div className="bg-card rounded-2xl shadow-lg p-6">
        <h2 className="text-xl font-bold mb-6">Available Rewards</h2>
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          {data.availableRewards.map((reward: any) => (
            <div key={reward.id} className="border border-border rounded-lg p-4">
              <div className="flex items-start justify-between mb-3">
                <div>
                  <div className="font-semibold mb-1">{reward.name}</div>
                  <div className="text-sm text-muted-foreground">{reward.description}</div>
                </div>
                <div className="bg-primary/10 px-3 py-1 rounded-full">
                  <span className="text-primary font-semibold">{reward.points} pts</span>
                </div>
              </div>
              <button className="w-full py-2 bg-primary text-white rounded-lg hover:bg-primary/90">
                Redeem
              </button>
            </div>
          ))}
        </div>
      </div>

      {/* Recent Activity */}
      <div className="bg-card rounded-2xl shadow-lg p-6">
        <h2 className="text-xl font-bold mb-6">Recent Activity</h2>
        <div className="space-y-3">
          {data.recentActivity.map((activity: any, i: number) => (
            <div key={i} className="flex items-center justify-between p-3 bg-muted/30 rounded-lg">
              <div>
                <div className="font-medium">{activity.description}</div>
                <div className="text-sm text-muted-foreground">
                  {new Date(activity.created_at).toLocaleDateString()}
                </div>
              </div>
              <div className={`font-semibold ${activity.points > 0 ? 'text-green-600' : 'text-red-600'}`}>
                {activity.points > 0 ? '+' : ''}{activity.points} pts
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
```

### Points Earning Logic

```typescript
// lib/loyalty/points.ts

export const POINTS_RULES = {
  hotel: {
    multiplier: 10, // 10 points per $1 spent
    bonus: {
      firstBooking: 500,
      weekend: 1.5,
      longStay: { minNights: 5, multiplier: 1.3 },
    },
  },
  flight: {
    multiplier: 5,
    bonus: {
      business: 1.5,
      first: 2.0,
      international: 1.2,
    },
  },
  tour: {
    multiplier: 8,
    bonus: {
      group: 1.2,
      premium: 1.5,
    },
  },
  transfer: {
    multiplier: 5,
    bonus: {
      luxury: 1.3,
    },
  },
};

export function calculatePoints(
  serviceType: string,
  amount: number,
  bookingDetails: any
): number {
  const rule = POINTS_RULES[serviceType as keyof typeof POINTS_RULES];
  if (!rule) return 0;

  let points = amount * rule.multiplier;

  // Apply bonuses
  // ... bonus logic

  return Math.floor(points);
}

export async function awardPoints(
  userId: string,
  bookingId: string,
  points: number,
  description: string
) {
  const supabase = createClient();

  // Add points transaction
  await supabase.from('loyalty_points').insert({
    user_id: userId,
    points,
    transaction_type: 'earned',
    booking_id: bookingId,
    description,
    expires_at: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000), // 1 year
  });

  // Update user's total points
  const { data: current } = await supabase
    .from('user_loyalty_status')
    .select('total_points, lifetime_points')
    .eq('user_id', userId)
    .single();

  await supabase
    .from('user_loyalty_status')
    .update({
      total_points: (current?.total_points || 0) + points,
      lifetime_points: (current?.lifetime_points || 0) + points,
    })
    .eq('user_id', userId);

  // Check for tier upgrade
  await checkTierUpgrade(userId);
}
```

---

## 7. 🔔 Price Alerts System

### Database Schema

```sql
-- Price alerts table
CREATE TABLE price_alerts (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  service_type TEXT NOT NULL,
  route_or_location TEXT NOT NULL,
  target_price DECIMAL(10,2),
  current_price DECIMAL(10,2),
  alert_type TEXT NOT NULL, -- price_drop, availability, deal
  search_criteria JSONB,
  active BOOLEAN DEFAULT true,
  last_checked TIMESTAMP,
  triggered_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP
);

-- Price history (for tracking)
CREATE TABLE price_history (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  service_type TEXT NOT NULL,
  service_id TEXT NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  availability INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Create indexes
CREATE INDEX idx_alerts_user ON price_alerts(user_id);
CREATE INDEX idx_alerts_active ON price_alerts(active);
CREATE INDEX idx_price_history_service ON price_history(service_type, service_id);
```

### Price Alert Component

Create `components/price-alerts/CreateAlert.tsx`:
```typescript
"use client";

import { useState } from 'react';
import { Bell, DollarSign } from 'lucide-react';

export default function CreatePriceAlert({
  serviceType,
  route,
  currentPrice,
}: {
  serviceType: string;
  route: string;
  currentPrice: number;
}) {
  const [targetPrice, setTargetPrice] = useState(currentPrice * 0.9);
  const [isCreating, setIsCreating] = useState(false);

  const handleCreate = async () => {
    setIsCreating(true);

    try {
      const response = await fetch('/api/price-alerts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          serviceType,
          route,
          targetPrice,
          currentPrice,
          alertType: 'price_drop',
        }),
      });

      if (response.ok) {
        alert('Price alert created! We\'ll notify you when the price drops.');
      }
    } catch (error) {
      console.error('Error creating alert:', error);
    } finally {
      setIsCreating(false);
    }
  };

  const savings = currentPrice - targetPrice;
  const savingsPercent = ((savings / currentPrice) * 100).toFixed(0);

  return (
    <div className="bg-card rounded-2xl shadow-lg p-6 border border-border">
      <div className="flex items-center gap-3 mb-4">
        <div className="bg-primary/10 p-3 rounded-lg">
          <Bell className="w-6 h-6 text-primary" />
        </div>
        <div>
          <div className="font-bold">Set Price Alert</div>
          <div className="text-sm text-muted-foreground">
            Get notified when prices drop
          </div>
        </div>
      </div>

      <div className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-2">
            Current Price
          </label>
          <div className="text-2xl font-bold text-muted-foreground">
            ${currentPrice}
          </div>
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Target Price
          </label>
          <div className="relative">
            <DollarSign className="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-muted-foreground" />
            <input
              type="number"
              value={targetPrice}
              onChange={(e) => setTargetPrice(Number(e.target.value))}
              min={0}
              max={currentPrice}
              step={10}
              className="w-full pl-10 pr-4 py-3 border border-border rounded-lg focus:ring-2 focus:ring-primary"
            />
          </div>
          <div className="flex items-center justify-between mt-2 text-sm">
            <span className="text-muted-foreground">
              Save ${savings.toFixed(2)} ({savingsPercent}%)
            </span>
            <input
              type="range"
              min={currentPrice * 0.5}
              max={currentPrice}
              value={targetPrice}
              onChange={(e) => setTargetPrice(Number(e.target.value))}
              className="w-32"
            />
          </div>
        </div>

        <button
          onClick={handleCreate}
          disabled={isCreating || targetPrice >= currentPrice}
          className="w-full py-3 bg-primary text-white rounded-lg hover:bg-primary/90 disabled:opacity-50 disabled:cursor-not-allowed font-semibold"
        >
          {isCreating ? 'Creating...' : 'Create Alert'}
        </button>

        <p className="text-xs text-muted-foreground text-center">
          We'll send you an email when the price drops to ${targetPrice} or below
        </p>
      </div>
    </div>
  );
}
```

### Price Monitoring Cron Job

Create `app/api/cron/check-price-alerts/route.ts`:
```typescript
import { NextResponse } from 'next/server';
import { createClient } from '@/lib/supabase/server';
import { sendEmail } from '@/lib/email/client';

export async function GET(request: Request) {
  // Verify cron secret
  const authHeader = request.headers.get('authorization');
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const supabase = createClient();

  try {
    // Get all active alerts
    const { data: alerts } = await supabase
      .from('price_alerts')
      .select('*')
      .eq('active', true);

    let triggeredCount = 0;

    for (const alert of alerts || []) {
      // Check current price (fetch from API or database)
      const currentPrice = await getCurrentPrice(
        alert.service_type,
        alert.search_criteria
      );

      // Check if price dropped below target
      if (currentPrice <= alert.target_price) {
        // Send notification
        await sendPriceAlertEmail(alert, currentPrice);

        // Mark alert as triggered
        await supabase
          .from('price_alerts')
          .update({
            active: false,
            triggered_at: new Date().toISOString(),
          })
          .eq('id', alert.id);

        triggeredCount++;
      } else {
        // Update current price
        await supabase
          .from('price_alerts')
          .update({
            current_price: currentPrice,
            last_checked: new Date().toISOString(),
          })
          .eq('id', alert.id);
      }

      // Save price to history
      await supabase.from('price_history').insert({
        service_type: alert.service_type,
        service_id: alert.route_or_location,
        price: currentPrice,
      });
    }

    return NextResponse.json({
      success: true,
      checked: alerts?.length || 0,
      triggered: triggeredCount,
    });
  } catch (error) {
    console.error('Price check error:', error);
    return NextResponse.json({ success: false, error }, { status: 500 });
  }
}

async function getCurrentPrice(serviceType: string, criteria: any): Promise<number> {
  // Implement price fetching logic based on service type
  // This would call the respective API (KPLUS, BTB) to get current prices
  return 0;
}

async function sendPriceAlertEmail(alert: any, currentPrice: number) {
  const savings = alert.target_price - currentPrice;

  await sendEmail({
    to: alert.user_email,
    subject: 'Price Alert: Your Target Price Reached!',
    html: `
      <h1>Great News! Price Drop Alert</h1>
      <p>The price for <strong>${alert.route_or_location}</strong> has dropped!</p>
      <p>Current Price: <strong>$${currentPrice}</strong></p>
      <p>Your Target: $${alert.target_price}</p>
      <p>You save: <strong>$${savings}</strong></p>
      <a href="https://healmedy.com">Book Now</a>
    `,
  });
}
```

**Setup Vercel Cron:**
```json
// vercel.json
{
  "crons": [
    {
      "path": "/api/cron/check-price-alerts",
      "schedule": "0 */6 * * *"
    }
  ]
}
```

---

## 8. 🎁 Bundle Deals

### Database Schema

```sql
-- Bundle deals table
CREATE TABLE bundle_deals (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  bundle_type TEXT NOT NULL, -- flight_hotel, tour_transfer, package
  services JSONB NOT NULL, -- Array of service types included
  discount_type TEXT NOT NULL, -- percentage, fixed
  discount_value DECIMAL(10,2) NOT NULL,
  min_total_price DECIMAL(10,2),
  valid_from TIMESTAMP NOT NULL,
  valid_until TIMESTAMP NOT NULL,
  max_uses INTEGER,
  current_uses INTEGER DEFAULT 0,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Bundle items (which services are in the bundle)
CREATE TABLE bundle_items (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  bundle_id UUID REFERENCES bundle_deals(id) ON DELETE CASCADE,
  service_type TEXT NOT NULL,
  service_id TEXT,
  required BOOLEAN DEFAULT true,
  min_nights INTEGER,
  min_passengers INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);

-- User bundle purchases
CREATE TABLE bundle_purchases (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  bundle_id UUID REFERENCES bundle_deals(id),
  booking_ids JSONB NOT NULL,
  original_price DECIMAL(10,2) NOT NULL,
  bundle_price DECIMAL(10,2) NOT NULL,
  savings DECIMAL(10,2) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Bundle Component

Create `components/bundles/BundleCard.tsx`:
```typescript
"use client";

import { Hotel, Plane, MapPin, Car, Tag, Clock } from 'lucide-react';

interface BundleDeal {
  id: string;
  name: string;
  description: string;
  services: string[];
  discount: number;
  originalPrice: number;
  bundlePrice: number;
  validUntil: string;
}

export default function BundleCard({ bundle }: { bundle: BundleDeal }) {
  const savings = bundle.originalPrice - bundle.bundlePrice;
  const savingsPercent = ((savings / bundle.originalPrice) * 100).toFixed(0);

  const getServiceIcon = (service: string) => {
    switch (service) {
      case 'hotel':
        return <Hotel className="w-5 h-5" />;
      case 'flight':
        return <Plane className="w-5 h-5" />;
      case 'tour':
        return <MapPin className="w-5 h-5" />;
      case 'transfer':
        return <Car className="w-5 h-5" />;
      default:
        return null;
    }
  };

  return (
    <div className="bg-card rounded-2xl shadow-lg overflow-hidden border-2 border-primary/20 hover:border-primary transition-all">
      {/* Savings Badge */}
      <div className="bg-gradient-to-r from-primary to-primary/80 p-4 text-white">
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-2">
            <Tag className="w-5 h-5" />
            <span className="font-bold">Bundle Deal</span>
          </div>
          <div className="bg-white/20 px-3 py-1 rounded-full">
            <span className="font-bold">Save {savingsPercent}%</span>
          </div>
        </div>
      </div>

      <div className="p-6">
        {/* Bundle Name */}
        <h3 className="text-2xl font-bold mb-2">{bundle.name}</h3>
        <p className="text-muted-foreground mb-4">{bundle.description}</p>

        {/* Included Services */}
        <div className="bg-muted/30 rounded-lg p-4 mb-4">
          <div className="text-sm font-medium mb-3">Included Services:</div>
          <div className="flex items-center gap-3 flex-wrap">
            {bundle.services.map((service) => (
              <div
                key={service}
                className="flex items-center gap-2 bg-white px-3 py-2 rounded-lg shadow-sm"
              >
                {getServiceIcon(service)}
                <span className="capitalize text-sm font-medium">{service}</span>
              </div>
            ))}
          </div>
        </div>

        {/* Pricing */}
        <div className="space-y-2 mb-4">
          <div className="flex items-center justify-between">
            <span className="text-muted-foreground">Original Price:</span>
            <span className="line-through text-muted-foreground">
              ${bundle.originalPrice}
            </span>
          </div>
          <div className="flex items-center justify-between">
            <span className="text-muted-foreground">Bundle Savings:</span>
            <span className="text-green-600 font-semibold">
              -${savings}
            </span>
          </div>
          <div className="flex items-center justify-between pt-2 border-t border-border">
            <span className="font-semibold">Bundle Price:</span>
            <span className="text-3xl font-bold text-primary">
              ${bundle.bundlePrice}
            </span>
          </div>
        </div>

        {/* Valid Until */}
        <div className="flex items-center gap-2 text-sm text-muted-foreground mb-4">
          <Clock className="w-4 h-4" />
          <span>
            Valid until {new Date(bundle.validUntil).toLocaleDateString()}
          </span>
        </div>

        {/* CTA Button */}
        <button className="w-full py-3 bg-primary text-white rounded-lg hover:bg-primary/90 font-semibold">
          Book This Bundle
        </button>
      </div>
    </div>
  );
}
```

### Bundle Page

Create `app/bundles/page.tsx`:
```typescript
"use client";

import { useState, useEffect } from 'react';
import BundleCard from '@/components/bundles/BundleCard';
import { Tag } from 'lucide-react';

export default function BundlesPage() {
  const [bundles, setBundles] = useState<any[]>([]);
  const [filter, setFilter] = useState<string>('all');
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    loadBundles();
  }, [filter]);

  const loadBundles = async () => {
    setIsLoading(true);
    try {
      const response = await fetch(`/api/bundles?filter=${filter}`);
      const result = await response.json();
      setBundles(result.bundles || []);
    } catch (error) {
      console.error('Error loading bundles:', error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="min-h-screen bg-muted/30">
      {/* Hero Section */}
      <section className="bg-gradient-to-br from-primary/10 via-background to-primary/5 py-20">
        <div className="container mx-auto px-4">
          <div className="text-center mb-12">
            <div className="inline-flex items-center justify-center w-16 h-16 bg-primary/10 rounded-full mb-4">
              <Tag className="w-8 h-8 text-primary" />
            </div>
            <h1 className="text-5xl font-bold mb-4">Bundle Deals</h1>
            <p className="text-xl text-muted-foreground max-w-2xl mx-auto">
              Save more by booking multiple services together
            </p>
          </div>
        </div>
      </section>

      {/* Filter Tabs */}
      <div className="container mx-auto px-4 py-8">
        <div className="flex gap-2 mb-8 overflow-x-auto">
          {['all', 'flight-hotel', 'tour-transfer', 'full-package'].map((f) => (
            <button
              key={f}
              onClick={() => setFilter(f)}
              className={`px-6 py-3 rounded-lg font-medium whitespace-nowrap transition-colors ${
                filter === f
                  ? 'bg-primary text-white'
                  : 'bg-card hover:bg-muted'
              }`}
            >
              {f.split('-').map(w => w.charAt(0).toUpperCase() + w.slice(1)).join(' + ')}
            </button>
          ))}
        </div>

        {/* Bundles Grid */}
        {isLoading ? (
          <div className="text-center py-20">
            <div className="w-16 h-16 border-4 border-primary border-t-transparent rounded-full animate-spin mx-auto mb-4"></div>
            <p>Loading bundles...</p>
          </div>
        ) : (
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {bundles.map((bundle) => (
              <BundleCard key={bundle.id} bundle={bundle} />
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
```

---

## Implementation Priority

### Phase 1: Quick Wins (Week 1)
1. ✅ Multi-language Support - DONE
2. Reviews and Ratings - High user engagement
3. Price Alerts - Unique value proposition

### Phase 2: Engagement (Week 2)
4. Email Notifications - Essential for bookings
5. Loyalty Program - Drives retention
6. SMS Confirmations - Improves trust

### Phase 3: Advanced (Week 3)
7. Advanced Analytics - Business insights
8. Bundle Deals - Increases AOV

---

## Testing Checklist

For each feature:
- [ ] Database schema created
- [ ] API endpoints implemented
- [ ] UI components built
- [ ] Mobile responsiveness verified
- [ ] Error handling tested
- [ ] Performance optimized
- [ ] Documentation updated
- [ ] User testing completed

---

## Maintenance Notes

### Regular Tasks:
- Monitor email delivery rates
- Check SMS credit balance
- Review analytics data accuracy
- Update loyalty tier thresholds
- Refresh bundle deals
- Clean up expired price alerts
- Archive old reviews

### Performance Optimization:
- Cache frequently accessed data
- Index database queries
- Optimize image loading
- Implement lazy loading
- Use CDN for static assets

---

**Last Updated:** 2025-11-03
**Version:** 1.0.0
**Status:** Implementation Guide Ready
