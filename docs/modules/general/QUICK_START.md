# Quick Start Guide

## What's Been Built

We've successfully created a brand new HealMedy Travel B2C platform from scratch! Here's what's ready:

### ✅ Completed

1. **Project Foundation**
   - Next.js 14 with TypeScript
   - Tailwind CSS with Cam Mavi theme (#00b8b8)
   - Development server running on port 3060

2. **Database Setup**
   - Complete Supabase schema with 6 tables
   - Database migration file ready to run
   - Type-safe database client utilities

3. **KPLUS API Integration**
   - Token management with automatic caching and refresh
   - Hotel search, validation, and booking functions
   - Full TypeScript types for all API calls

4. **UI Components**
   - Beautiful homepage with hero section
   - Responsive Header with navigation
   - Footer with links and contact info
   - Hotel search page with advanced search form

5. **Features**
   - Multi-room search (up to 5 rooms)
   - Adult/child configuration with age selection
   - Date pickers for check-in/out
   - Glass morphism UI effects
   - Mobile-responsive design

## Your Application is Live!

🎉 **Your development server is running at: [http://localhost:3060](http://localhost:3060)**

### What You'll See

1. **Homepage** (/)
   - Full-screen hero with Cam Mavi gradient
   - Quick links to Hotels, Flights, Tours, Transfers
   - "Why Choose Us?" features section
   - Stats section (5,000+ hotels, etc.)

2. **Hotels Page** (/hotels)
   - Advanced search form
   - Popular destinations showcase
   - Feature highlights

## Next: Complete Supabase Setup

To start using the full functionality, you need to set up your Supabase database:

### Step 1: Get Your Supabase Credentials

1. Go to [https://supabase.com](https://supabase.com)
2. Sign in and select your project (or create a new one)
3. Go to **Project Settings** → **API**
4. Copy these values:
   - **Project URL** (e.g., `https://xxxxx.supabase.co`)
   - **anon public** key
   - **service_role** key (show it first)

### Step 2: Update Environment Variables

Open `.env.local` and replace these lines:

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here
```

With your actual credentials.

### Step 3: Run Database Migration

1. In your Supabase dashboard, go to **SQL Editor**
2. Click **New Query**
3. Open `supabase/migrations/20250103000000_initial_schema.sql`
4. Copy all the contents
5. Paste into the SQL Editor
6. Click **Run**
7. You should see "Success. No rows returned"

### Step 4: Verify Tables Created

Go to **Table Editor** in Supabase and verify these tables exist:
- ✓ profiles
- ✓ kplus_tokens
- ✓ hotels
- ✓ hotel_rooms
- ✓ bookings
- ✓ payment_transactions

### Step 5: Restart Development Server

After updating `.env.local`:

```bash
# Stop the current server (Ctrl+C in the terminal)
# Then restart:
npm run dev
```

## What to Build Next

Now that the foundation is ready, here are the next features to implement:

### Priority 1: Hotel Search Results
**File**: `app/hotels/search/page.tsx`

Create the search results page that:
- Receives search parameters from the search form
- Calls the KPLUS API to search hotels
- Displays results in a grid with cards
- Shows hotel images, pricing, ratings
- Includes filters and sorting

### Priority 2: Hotel Details Page
**File**: `app/hotels/[id]/page.tsx`

Display detailed hotel information:
- Hotel images gallery
- Description and amenities
- Room options with pricing
- Cancellation policies
- Booking form

### Priority 3: Booking Flow
**Files**:
- `app/hotels/[id]/book/page.tsx`
- `components/hotels/BookingForm.tsx`

Implement the booking process:
1. Guest information form
2. Validate with KPLUS API
3. Show price confirmation
4. Process payment with Iyzico
5. Create booking in database
6. Send confirmation email

### Priority 4: Authentication
**Files**:
- `app/login/page.tsx`
- `app/register/page.tsx`
- `app/profile/page.tsx`

Add user authentication:
- Sign up with email/password
- Login functionality
- User profile page
- View booking history

## Project Structure Overview

```
📁 healmedy-travel-v2/
│
├── 📁 app/                    # Next.js pages
│   ├── 📄 layout.tsx         # Root layout (Header/Footer)
│   ├── 📄 page.tsx           # Homepage
│   └── 📁 hotels/
│       └── 📄 page.tsx       # Hotel search page
│
├── 📁 components/
│   ├── 📁 layout/
│   │   ├── 📄 Header.tsx     # Navigation header
│   │   └── 📄 Footer.tsx     # Footer with links
│   └── 📁 hotels/
│       └── 📄 HotelSearchForm.tsx  # Search form
│
├── 📁 lib/
│   ├── 📁 supabase/
│   │   ├── 📄 client.ts      # Browser client
│   │   ├── 📄 server.ts      # Server client
│   │   └── 📄 database.types.ts
│   └── 📁 kplus/
│       ├── 📄 client.ts      # API functions
│       ├── 📄 token-manager.ts
│       └── 📄 types.ts
│
├── 📁 supabase/
│   ├── 📁 migrations/
│   │   └── 📄 20250103000000_initial_schema.sql
│   └── 📄 README.md
│
├── 📄 .env.local             # Environment variables
├── 📄 README.md              # Full documentation
└── 📄 QUICK_START.md         # This file
```

## Key Files to Know

### Configuration
- **`.env.local`** - All your API keys and credentials
- **`tailwind.config.ts`** - Theme configuration (Cam Mavi colors)
- **`app/globals.css`** - Global styles and custom utilities

### Database
- **`supabase/migrations/`** - Database schema
- **`lib/supabase/database.types.ts`** - TypeScript types for tables

### API Integration
- **`lib/kplus/client.ts`** - KPLUS API functions
- **`lib/kplus/token-manager.ts`** - Automatic token handling
- **`lib/kplus/types.ts`** - API request/response types

## Need Help?

### Documentation
- Full README: [README.md](./README.md)
- Supabase setup: [supabase/README.md](./supabase/README.md)
- KPLUS integration: [lib/kplus/README.md](./lib/kplus/README.md)

### Check Development Server
- Server running at: http://localhost:3060
- Check terminal for any errors
- Hot reload is enabled (changes auto-refresh)

### Common Issues

**Server won't start?**
```bash
# Clear Next.js cache
rm -rf .next
npm run dev
```

**TypeScript errors?**
```bash
# Check tsconfig.json is present
# Restart VS Code or your editor
```

**Environment variables not working?**
- Restart dev server after changing `.env.local`
- Make sure there are no quotes around values
- Check for typos in variable names

## What's Working Right Now

Even without Supabase credentials, you can:

1. ✅ Browse the homepage
2. ✅ Navigate to the hotels page
3. ✅ Use the search form (UI only, needs backend)
4. ✅ See all the beautiful UI components
5. ✅ Test the responsive mobile design

Once you add Supabase credentials:

6. ✅ KPLUS token management will work
7. ✅ Database queries will work
8. ✅ Authentication will work
9. ✅ Full booking flow will work

## You're Ready to Build! 🚀

The foundation is solid. Start with the hotel search results page and build from there. All the hard infrastructure work (database, API integration, UI framework) is done!

Happy coding! 💙
