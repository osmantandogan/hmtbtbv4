# 🚀 Complete System Build Plan - HealMedy Travel Platform

## 📋 MASTER PROJECT OVERVIEW

Building a **complete B2C + B2B travel booking platform** with:
- ✅ Hotels (95% complete)
- 🔄 Flights (to build)
- 🔄 Tours (to build)
- 🔄 Transfers (to build)
- 🔄 User Authentication & Dashboard
- 🔄 Admin Panel

---

## 🎯 BUILD PHASES

### **Phase 1: Complete Hotels System** ✅ (Current - 95%)
**Status**: Nearly complete, needs payment frontend
**Time**: 1 hour
**Priority**: CRITICAL

#### Remaining:
1. Update payment page with Iyzico integration
2. Add demo mode toggle

---

### **Phase 2: User Authentication System** 🔄
**Status**: Not started (0%)
**Time**: 1 day (8 hours)
**Priority**: HIGH

#### Components to Build:
1. **Login Page** (`/login`)
   - Email/password login
   - Social login (Google, Facebook) optional
   - Remember me checkbox
   - Forgot password link

2. **Register Page** (`/register`)
   - Email/password registration
   - Email verification
   - Terms acceptance
   - Auto-login after registration

3. **Password Reset** (`/reset-password`)
   - Email verification
   - Reset token validation
   - New password form

4. **Supabase Auth Integration**
   - Auth context provider
   - Protected routes middleware
   - Session management
   - Automatic token refresh

#### Database:
- ✅ `profiles` table already exists
- Need: User preferences, settings

---

### **Phase 3: User Dashboard** 🔄
**Status**: Not started (0%)
**Time**: 2 days (16 hours)
**Priority**: HIGH

#### Pages to Build:
1. **Dashboard Home** (`/dashboard`)
   - Overview of bookings
   - Quick stats
   - Recent activity

2. **My Bookings** (`/dashboard/bookings`)
   - All bookings list (hotels, flights, tours, transfers)
   - Filter by status (upcoming, past, cancelled)
   - Booking details modal
   - Download voucher
   - Cancel booking (with rules)

3. **Profile Settings** (`/dashboard/profile`)
   - Personal information
   - Contact details
   - Password change
   - Email preferences

4. **Payment Methods** (`/dashboard/payment-methods`)
   - Saved cards
   - Add/remove cards
   - Default payment method

#### Features:
- Booking history with pagination
- Booking status tracking
- Email notifications toggle
- Profile photo upload

---

### **Phase 4: Flights System** 🔄
**Status**: Not started (0%)
**Time**: 1 week (40 hours)
**Priority**: MEDIUM

#### API Integration Options:
1. **Amadeus API** (Recommended)
   - Self-service API
   - Good documentation
   - Free tier available
   - REST API

2. **Sabre API**
   - More complex
   - Enterprise-focused
   - Better rates

3. **Travelport API**
   - GDS integration
   - Comprehensive

**Decision**: Start with Amadeus (easier to integrate)

#### Pages to Build:
1. **Flight Search** (`/flights`)
   - Origin/destination airports
   - Departure/return dates
   - Passengers (adults/children/infants)
   - Class selection (economy/business/first)
   - Multi-city option

2. **Flight Results** (`/flights/search`)
   - Flight cards with details
   - Filters (price, airlines, stops, duration)
   - Sorting options
   - Fare comparison

3. **Flight Details** (`/flights/[id]`)
   - Full itinerary
   - Baggage info
   - Seat map (optional)
   - Fare rules

4. **Flight Booking** (`/flights/[id]/book`)
   - Passenger information
   - Seat selection
   - Extras (baggage, meals)
   - Payment

#### Database Tables:
```sql
-- Flight bookings
CREATE TABLE flight_bookings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id),
  booking_reference TEXT UNIQUE NOT NULL,
  pnr_code TEXT, -- Airline PNR
  origin TEXT NOT NULL,
  destination TEXT NOT NULL,
  departure_date TIMESTAMP NOT NULL,
  return_date TIMESTAMP,
  passengers JSONB NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL,
  currency TEXT DEFAULT 'TRY',
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW()
);

-- Flight segments
CREATE TABLE flight_segments (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  booking_id UUID REFERENCES flight_bookings(id),
  airline_code TEXT NOT NULL,
  flight_number TEXT NOT NULL,
  origin TEXT NOT NULL,
  destination TEXT NOT NULL,
  departure_time TIMESTAMP NOT NULL,
  arrival_time TIMESTAMP NOT NULL,
  cabin_class TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

### **Phase 5: Tours System** 🔄
**Status**: Not started (0%)
**Time**: 1 week (40 hours)
**Priority**: MEDIUM

#### Approach Options:
1. **Manual Tour Management** (Recommended initially)
   - Admin creates tours
   - No external API needed
   - Full control

2. **Tour Operator API Integration**
   - GetYourGuide API
   - Viator API
   - TourRadar API

**Decision**: Start with manual, add API later

#### Pages to Build:
1. **Tours Listing** (`/tours`)
   - Browse all tours
   - Filters (destination, price, duration, category)
   - Search functionality
   - Featured tours

2. **Tour Details** (`/tours/[slug]`)
   - Photo gallery
   - Detailed description
   - Itinerary
   - Inclusions/exclusions
   - Reviews
   - Available dates
   - Pricing

3. **Tour Booking** (`/tours/[slug]/book`)
   - Date selection
   - Number of travelers
   - Pickup location (if applicable)
   - Special requests
   - Payment

#### Database Tables:
```sql
-- Tours
CREATE TABLE tours (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  slug TEXT UNIQUE NOT NULL,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  destination TEXT NOT NULL,
  duration_days INTEGER NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  currency TEXT DEFAULT 'TRY',
  images JSONB,
  itinerary JSONB,
  inclusions JSONB,
  exclusions JSONB,
  max_capacity INTEGER NOT NULL,
  category TEXT NOT NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Tour bookings
CREATE TABLE tour_bookings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id),
  tour_id UUID REFERENCES tours(id),
  booking_reference TEXT UNIQUE NOT NULL,
  tour_date DATE NOT NULL,
  travelers JSONB NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL,
  currency TEXT DEFAULT 'TRY',
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW()
);

-- Tour availability
CREATE TABLE tour_availability (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  tour_id UUID REFERENCES tours(id),
  date DATE NOT NULL,
  available_slots INTEGER NOT NULL,
  price DECIMAL(10,2),
  UNIQUE(tour_id, date)
);
```

---

### **Phase 6: Transfers System** 🔄
**Status**: Not started (0%)
**Time**: 3-4 days (30 hours)
**Priority**: MEDIUM

#### Types of Transfers:
1. Airport → Hotel
2. Hotel → Airport
3. Point to Point
4. Hourly rental

#### Pages to Build:
1. **Transfer Search** (`/transfers`)
   - Pickup location (autocomplete)
   - Dropoff location
   - Date & time
   - Passengers & luggage
   - Vehicle type

2. **Transfer Results** (`/transfers/search`)
   - Vehicle options (sedan, SUV, van, luxury)
   - Pricing comparison
   - Features (WiFi, child seat, etc.)
   - Provider ratings

3. **Transfer Booking** (`/transfers/book`)
   - Passenger details
   - Flight information (optional)
   - Special requests
   - Payment

#### Database Tables:
```sql
-- Transfer vehicles
CREATE TABLE transfer_vehicles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  type TEXT NOT NULL, -- sedan, suv, van, luxury
  capacity INTEGER NOT NULL,
  luggage_capacity INTEGER NOT NULL,
  price_per_km DECIMAL(10,2),
  base_price DECIMAL(10,2),
  features JSONB,
  image_url TEXT,
  is_active BOOLEAN DEFAULT true
);

-- Transfer bookings
CREATE TABLE transfer_bookings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id),
  booking_reference TEXT UNIQUE NOT NULL,
  vehicle_id UUID REFERENCES transfer_vehicles(id),
  pickup_location TEXT NOT NULL,
  dropoff_location TEXT NOT NULL,
  pickup_datetime TIMESTAMP NOT NULL,
  passengers INTEGER NOT NULL,
  flight_number TEXT,
  total_amount DECIMAL(10,2) NOT NULL,
  currency TEXT DEFAULT 'TRY',
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

### **Phase 7: Admin Panel** 🔄
**Status**: Not started (0%)
**Time**: 2 weeks (80 hours)
**Priority**: LOW (but important)

#### Admin Dashboard Structure:
```
/admin
├── /dashboard (Overview)
├── /bookings (All bookings management)
│   ├── /hotels
│   ├── /flights
│   ├── /tours
│   └── /transfers
├── /users (User management)
├── /tours (Tour content management)
├── /transfers (Vehicle management)
├── /analytics (Reports & stats)
└── /settings (System settings)
```

#### Features to Build:
1. **Admin Authentication**
   - Separate admin login
   - Role-based access (admin, manager, support)
   - Admin session management

2. **Dashboard Overview**
   - Today's bookings
   - Revenue stats
   - Recent activity
   - Quick actions

3. **Booking Management**
   - View all bookings
   - Filter by type, status, date
   - Booking details
   - Cancel/refund bookings
   - Manual booking creation
   - Export to CSV/Excel

4. **User Management**
   - View all users
   - User details
   - Booking history per user
   - Ban/suspend users
   - Manual user creation

5. **Content Management**
   - **Tours**: Create, edit, delete tours
   - **Vehicles**: Manage transfer vehicles
   - Upload images
   - Set availability

6. **Analytics & Reports**
   - Revenue reports (daily, monthly, yearly)
   - Booking trends
   - Top destinations
   - User growth
   - Payment success rate

7. **Settings**
   - System configuration
   - Email templates
   - Payment gateway settings
   - API keys management

#### Database Tables:
```sql
-- Admin users
CREATE TABLE admin_users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'admin', -- admin, manager, support
  is_active BOOLEAN DEFAULT true,
  last_login TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Admin activity log
CREATE TABLE admin_activity_log (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  admin_id UUID REFERENCES admin_users(id),
  action TEXT NOT NULL,
  entity_type TEXT NOT NULL,
  entity_id UUID,
  details JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 📊 COMPLETE DATABASE SCHEMA

### Current Tables (Already Created):
1. ✅ profiles
2. ✅ kplus_tokens
3. ✅ hotels
4. ✅ hotel_rooms
5. ✅ bookings (hotels)
6. ✅ payment_transactions

### New Tables Needed:
7. 🔄 flight_bookings
8. 🔄 flight_segments
9. 🔄 tours
10. 🔄 tour_bookings
11. 🔄 tour_availability
12. 🔄 transfer_vehicles
13. 🔄 transfer_bookings
14. 🔄 admin_users
15. 🔄 admin_activity_log
16. 🔄 user_preferences
17. 🔄 saved_payment_methods

**Total Tables**: 17

---

## 🔧 TECHNICAL REQUIREMENTS

### APIs to Integrate:
1. ✅ **KPLUS** - Hotels (done)
2. ✅ **Iyzico** - Payments (backend done)
3. 🔄 **Amadeus** - Flights
4. ✅ **Resend** - Emails (done)
5. 🔄 **Cloudinary/S3** - Image uploads (for tours)

### NPM Packages to Install:
```bash
# Flight integration
npm install amadeus

# Image uploads
npm install cloudinary
# or
npm install @aws-sdk/client-s3

# Admin dashboard
npm install recharts  # Charts
npm install date-fns  # Date utilities
npm install react-table  # Data tables

# Excel export
npm install xlsx
```

---

## ⏱️ TIME ESTIMATES

| Phase | Task | Hours | Days |
|-------|------|-------|------|
| 1 | Complete Hotels Payment | 1 | 0.5 |
| 2 | User Authentication | 8 | 1 |
| 3 | User Dashboard | 16 | 2 |
| 4 | Flights System | 40 | 5 |
| 5 | Tours System | 40 | 5 |
| 6 | Transfers System | 30 | 4 |
| 7 | Admin Panel | 80 | 10 |
| **TOTAL** | **215 hours** | **27.5 days** |

**Realistic Timeline**: 5-6 weeks (working 5-6 hours/day)

---

## 💰 COST ESTIMATES

### API Subscriptions:
1. **KPLUS** - Already have
2. **Iyzico** - Free sandbox, 2-3% per transaction in production
3. **Amadeus** - Free tier (limited), ~$0.25 per search in production
4. **Resend** - Free tier 100 emails/day, $20/month for 50k
5. **Supabase** - Free tier, $25/month for production
6. **Hosting (Vercel)** - Free tier, $20/month for pro

**Monthly Estimate**: $65-100 + transaction fees

---

## 🎯 BUILD ORDER (Recommended)

### Week 1:
- ✅ Day 1: Complete hotels payment page
- ✅ Day 2: User authentication system
- ✅ Day 3-4: User dashboard basics
- ✅ Day 5: My Bookings page

### Week 2-3:
- 🔄 Flights system complete
- 🔄 Amadeus API integration
- 🔄 Flight booking flow

### Week 4:
- 🔄 Tours system
- 🔄 Tour management
- 🔄 Tour booking

### Week 5:
- 🔄 Transfers system
- 🔄 Vehicle management
- 🔄 Transfer booking

### Week 6+:
- 🔄 Admin panel
- 🔄 Analytics
- 🔄 Polish & testing

---

## 🚀 DEPLOYMENT PLAN

### Staging Environment:
- Deploy to Vercel staging
- Use sandbox APIs
- Test all flows

### Production:
1. Get production API keys
2. Enable production mode toggles
3. Update environment variables
4. Deploy to production
5. Monitor for issues

---

## 📋 NEXT STEPS

**Right Now, I will:**

1. ✅ Complete payment page (1 hour)
2. ✅ Build user authentication (4-6 hours)
3. ✅ Start user dashboard (2 hours)

**Then we'll decide:**
- Continue with flights?
- Or build tours/transfers first?
- Or jump to admin panel?

---

**Let's start building! 🎉**

I'll begin with completing the payment page, then move to authentication.

**Ready?** Let me know and I'll start coding! 🚀
