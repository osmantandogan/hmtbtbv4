# BTB API Integration Plan

## Overview

We're integrating the BTB (Tourbridge) API to provide Tours, Flights, and Transfers alongside the existing KPLUS Hotels integration.

## API Structure

### Two APIs Working Together:

1. **KPLUS API** → Hotels
   - Base URL: `http://sandbox.kplus.com.tr/kplus/v0`
   - Already integrated ✅

2. **BTB/Tourbridge API** → Tours, Flights, Transfers
   - Base URL: `https://tourbridge.preview.emergentagent.com/api`
   - New integration 🆕

## Implementation Plan

### Phase 4: Flights System (Week 1)

**Pages to Build:**
1. `/flights` - Flight search form
2. `/flights/search` - Flight results with filters
3. `/flights/[id]` - Flight details
4. `/flights/[id]/book` - Flight booking

**API Integration:**
- `GET /api/flights/search` → BTB API
- Mock data fallback for development
- Toggle: `USE_REAL_API` (same pattern as hotels)

**Features:**
- One-way and round-trip flights
- Multi-passenger support
- Cabin class selection (Economy/Business/First)
- Filter by price, airline, stops, duration
- Sorting options

---

### Phase 5: Tours System (Week 2)

**Pages to Build:**
1. `/tours` - Browse all tours
2. `/tours/[slug]` - Tour details with gallery
3. `/tours/[slug]/book` - Tour booking

**API Integration:**
- `GET /api/tours/search` → BTB API
- Date range filtering
- Search by destination/text
- Mock data fallback

**Features:**
- Tour categories
- Date availability
- Inclusions/Exclusions
- Image galleries
- Reviews (future)
- Multi-traveler support

---

### Phase 6: Transfers System (Week 3)

**Pages to Build:**
1. `/transfers` - Transfer search form
2. `/transfers/search` - Vehicle options
3. `/transfers/book` - Transfer booking

**API Integration:**
- `GET /api/transfers/search` → BTB API
- Airport/Hotel transfers
- Mock data fallback

**Features:**
- Vehicle types (Sedan, SUV, Van, Luxury)
- Passenger and luggage capacity
- Point-to-point or airport transfers
- Flight info integration

---

## Database Updates Needed

### New Booking Types

Update `bookings` table to support all types:
```sql
ALTER TABLE bookings
ADD COLUMN booking_type TEXT DEFAULT 'hotel', -- hotel, flight, tour, transfer
ADD COLUMN btb_booking_id TEXT, -- For BTB bookings
ADD COLUMN flight_details JSONB, -- Flight-specific data
ADD COLUMN tour_details JSONB, -- Tour-specific data
ADD COLUMN transfer_details JSONB; -- Transfer-specific data
```

### Separate Tables (Optional - for advanced features)

```sql
-- Flight bookings
CREATE TABLE flight_bookings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  booking_id UUID REFERENCES bookings(id),
  flight_id TEXT NOT NULL,
  airline TEXT,
  flight_number TEXT,
  origin TEXT,
  destination TEXT,
  departure_time TIMESTAMP,
  arrival_time TIMESTAMP,
  passengers JSONB,
  cabin_class TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Tour bookings
CREATE TABLE tour_bookings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  booking_id UUID REFERENCES bookings(id),
  tour_id TEXT NOT NULL,
  tour_name TEXT,
  tour_date DATE,
  travelers JSONB,
  pickup_location TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Transfer bookings
CREATE TABLE transfer_bookings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  booking_id UUID REFERENCES bookings(id),
  transfer_id TEXT NOT NULL,
  vehicle_type TEXT,
  pickup_location TEXT,
  dropoff_location TEXT,
  pickup_datetime TIMESTAMP,
  passengers INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Toggle System

Following the same pattern as hotels:

```typescript
// In each page
const USE_REAL_API = false; // Set to true for production

if (USE_REAL_API) {
  // Call BTB API
  const response = await fetch('/api/flights/search', {...});
} else {
  // Use mock data
  const mockFlights = [...];
}
```

---

## API Client Usage

### Tours Example:
```typescript
import { searchTours } from '@/lib/btb/client';

const tours = await searchTours({
  searchText: 'Istanbul',
  startDate: '2025-07-01',
  endDate: '2025-07-10',
});
```

### Flights Example:
```typescript
import { searchFlights } from '@/lib/btb/client';

const flights = await searchFlights({
  origin: 'IST',
  destination: 'JFK',
  departureDate: '2025-12-15',
  adults: 2,
  cabinClass: 1, // Economy
});
```

### Transfers Example:
```typescript
import { searchTransfers } from '@/lib/btb/client';

const transfers = await searchTransfers({
  originCode: 'IST',
  destinationCode: 'HOTEL',
  transferDate: '2025-12-01',
  passengerCount: 2,
});
```

---

## Environment Variables

Add to `.env.local`:
```env
BTB_BASE_URL=https://tourbridge.preview.emergentagent.com/api
BTB_API_KEY=your_api_key_here  # Optional - for B2B partners
```

---

## Build Order

1. ✅ Hotels (Complete)
2. ✅ Authentication (Complete)
3. ✅ Dashboard (Complete)
4. 🔄 **Flights** (Start here)
5. ⏭️ Tours
6. ⏭️ Transfers
7. ⏭️ Admin Panel

---

## Testing Strategy

### Development:
- Use mock data with `USE_REAL_API = false`
- Test UI/UX without API dependency
- Fast iteration

### Staging:
- Enable `USE_REAL_API = true`
- Test with BTB sandbox/preview environment
- Verify data transformation

### Production:
- Use production BTB API endpoint
- Monitor API responses
- Error handling and fallbacks

---

## Benefits of This Approach

1. **Unified Platform**: Single platform for all travel needs
2. **Consistent UX**: Same design patterns across all services
3. **Shared Auth**: One login for everything
4. **Integrated Bookings**: View all bookings in one dashboard
5. **Easy Maintenance**: Same code patterns throughout

---

**Next Step**: Start building Flights search page! 🚀
