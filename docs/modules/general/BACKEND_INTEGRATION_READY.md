# 🔌 Backend Integration - Ready to Connect!

## ✅ What's Been Prepared

While you're running the Supabase migration, I've prepared the backend integration:

### 1. API Route Handlers Created

Three new API endpoints ready to connect your frontend to KPLUS:

#### **Hotel Search API** - `/api/hotels/search`
**File**: `app/api/hotels/search/route.ts`

**What it does**:
- Receives search parameters from frontend
- Calls KPLUS API to search hotels
- Returns formatted results

**Usage**:
```typescript
const response = await fetch('/api/hotels/search', {
  method: 'POST',
  body: JSON.stringify({
    checkIn: '2025-07-01',
    checkOut: '2025-07-05',
    rooms: [{ adults: 2, childAges: [] }],
    nationality: 'TR',
    destination: 'Istanbul'
  })
});

const data = await response.json();
// data.hotels = array of hotels from KPLUS
```

#### **Hotel Validation API** - `/api/hotels/validate`
**File**: `app/api/hotels/validate/route.ts`

**What it does**:
- Validates hotel availability before booking
- Checks for price changes
- Returns current pricing

**Usage**:
```typescript
const response = await fetch('/api/hotels/validate', {
  method: 'POST',
  body: JSON.stringify({
    searchId: 'search-123',
    hotelCode: 'HOTEL001',
    roomCode: 'ROOM001'
  })
});

const data = await response.json();
// data.isAvailable = true/false
// data.priceChanged = true/false
// data.currentPrice = updated price
```

#### **Hotel Booking API** - `/api/hotels/book`
**File**: `app/api/hotels/book/route.ts`

**What it does**:
- Creates booking via KPLUS API
- Stores booking in Supabase database
- Returns booking reference

**Usage**:
```typescript
const response = await fetch('/api/hotels/book', {
  method: 'POST',
  body: JSON.stringify({
    searchId: 'search-123',
    hotelCode: 'HOTEL001',
    roomCode: 'ROOM001',
    guestInfo: {
      title: 'Mr',
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com',
      phone: '+905551234567',
      country: 'TR'
    },
    checkIn: '2025-07-01',
    checkOut: '2025-07-05'
  })
});

const data = await response.json();
// data.bookingReference = 'HM12345ABC'
// data.kplusBookingCode = KPLUS booking code
```

---

## 🔄 Next: Connect Frontend to Backend

Once your Supabase migration is complete, here's what we'll do:

### Step 1: Update Search Results Page
**File**: `app/hotels/search/page.tsx`

**Change**:
```typescript
// FROM: Using mock data
import { mockHotels } from "@/lib/mock-data/hotels";

// TO: Calling real API
const response = await fetch('/api/hotels/search', {
  method: 'POST',
  body: JSON.stringify(searchParams)
});
const { hotels } = await response.json();
```

### Step 2: Update Booking Page
**File**: `app/hotels/[id]/book/page.tsx`

**Change**:
```typescript
// FROM: Mock booking creation
const bookingRef = "HM" + Math.random().toString(36).substring(2, 10);

// TO: Real KPLUS booking
const response = await fetch('/api/hotels/book', {
  method: 'POST',
  body: JSON.stringify(bookingData)
});
const { bookingReference } = await response.json();
```

### Step 3: Add Validation Before Payment
**File**: `app/hotels/[id]/book/page.tsx`

**Add before payment**:
```typescript
// Validate availability and pricing
const validation = await fetch('/api/hotels/validate', {
  method: 'POST',
  body: JSON.stringify({ searchId, hotelCode, roomCode })
});

const { isAvailable, priceChanged, currentPrice } = await validation.json();

if (!isAvailable) {
  alert('Sorry, this room is no longer available');
  return;
}

if (priceChanged) {
  alert(`Price updated to ${currentPrice}`);
}
```

---

## 📊 What Happens After Migration

### Immediate Benefits:
1. ✅ **Real Hotel Data**: Search results from KPLUS API
2. ✅ **Live Availability**: Real-time room availability
3. ✅ **Accurate Pricing**: Current prices from hotels
4. ✅ **Database Storage**: All bookings saved in Supabase
5. ✅ **Token Caching**: KPLUS tokens cached for performance

### Database Tables Active:
- `kplus_tokens` - Caches API tokens (auto-refresh)
- `hotels` - Can cache static hotel content
- `bookings` - Stores all booking records
- `payment_transactions` - Tracks payments
- `profiles` - User accounts (for auth later)

---

## 🧪 Testing the APIs

### Test 1: Search API
```bash
curl -X POST http://localhost:3060/api/hotels/search \
  -H "Content-Type: application/json" \
  -d '{
    "checkIn": "2025-07-01",
    "checkOut": "2025-07-05",
    "rooms": [{"adults": 2}],
    "nationality": "TR",
    "destination": "Istanbul"
  }'
```

### Test 2: Check KPLUS Connection
Visit: http://localhost:3060/api/hotels/search?checkIn=2025-07-01&checkOut=2025-07-05&rooms=[{"adults":2}]

You should see JSON response with hotels!

---

## ⚡ Performance Optimizations Ready

### 1. Token Caching
- KPLUS tokens cached in Supabase
- Auto-refresh 5 minutes before expiry
- No repeated authentication calls

### 2. Hotel Static Content Caching
Can run a script to cache all hotels:

```typescript
// Script to cache hotels (we'll create this)
const hotels = await getHotelStaticContent();
for (const hotel of hotels) {
  await supabase.from('hotels').insert({
    kplus_hotel_code: hotel.HotelCode,
    name: hotel.Name,
    // ... other fields
  });
}
```

### 3. Search Result Caching
Can cache search results temporarily to improve performance

---

## 🎯 Migration Complete Checklist

After you run the Supabase migration, verify:

- [ ] 6 tables created in Supabase
- [ ] No middleware errors in console
- [ ] Dev server restarted
- [ ] Ready to connect APIs

---

## 📝 Current Status

**Prepared**:
- ✅ 3 API route handlers
- ✅ KPLUS integration code
- ✅ Database schema
- ✅ Token management
- ✅ Error handling

**Waiting For**:
- ⏳ Supabase migration to complete
- ⏳ Your confirmation

**Next Steps** (after migration):
1. Test API endpoints
2. Connect search results to real data
3. Connect booking flow to KPLUS
4. Add authentication pages
5. Integrate Iyzico payment

---

## 🚀 What to Do Now

1. **Complete the Supabase migration** using [MIGRATION_INSTRUCTIONS.md](./MIGRATION_INSTRUCTIONS.md)

2. **Restart your dev server**:
   ```bash
   # Stop with Ctrl+C
   npm run dev
   ```

3. **Test the search API**:
   Visit: http://localhost:3060/api/hotels/search?checkIn=2025-07-01&checkOut=2025-07-05&rooms=[{"adults":2}]

4. **Let me know** - I'll connect everything!

---

**Ready to go live with real data! Just complete the migration and we'll connect it all.** 🔥
