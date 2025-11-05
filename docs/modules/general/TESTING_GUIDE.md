# HealMedy Travel - Testing Guide

## 🧪 Comprehensive Testing Checklist

This guide provides step-by-step testing procedures for all platform features.

---

## Prerequisites

### Environment Setup
- [ ] Development server running on http://localhost:3060
- [ ] Supabase database accessible
- [ ] Test user accounts created
- [ ] Admin account configured (admin@healmedy.com)

### Test Data
- [ ] Mock data loaded for all services
- [ ] Sample bookings in database
- [ ] Test user profiles created

---

## 1️⃣ Hotels System Testing

### Test Case 1.1: Search Functionality
**Steps:**
1. Navigate to http://localhost:3060/hotels
2. Enter destination: "Istanbul"
3. Select check-in: Tomorrow
4. Select check-out: +7 days
5. Guests: 2 adults, 1 child
6. Click "Search Hotels"

**Expected Results:**
- ✅ Search results page loads
- ✅ Hotels in Istanbul displayed
- ✅ Filters available (price, rating, amenities)
- ✅ Total count shows correct number

**Status:** [ ] Pass [ ] Fail

---

### Test Case 1.2: Hotel Booking Flow
**Steps:**
1. From search results, click any hotel
2. Review hotel details page
3. Click "Book Now"
4. Fill guest information form
5. Fill contact details
6. Add special requests
7. Click "Continue to Payment"

**Expected Results:**
- ✅ Details page shows all hotel info
- ✅ Booking form validation works
- ✅ Price calculation correct
- ✅ Redirect to payment page
- ✅ Payment page shows correct total

**Status:** [ ] Pass [ ] Fail

---

### Test Case 1.3: Hotel Filters
**Steps:**
1. On search results page
2. Set price range: $50-$150
3. Select rating: 4+ stars
4. Filter by amenities: WiFi, Pool
5. Sort by: Price (Low to High)

**Expected Results:**
- ✅ Results filtered correctly
- ✅ Count updates dynamically
- ✅ Sorting works as expected
- ✅ Clear filters resets all

**Status:** [ ] Pass [ ] Fail

---

## 2️⃣ Flights System Testing

### Test Case 2.1: Round-Trip Search
**Steps:**
1. Navigate to http://localhost:3060/flights
2. Select "Round Trip"
3. Origin: IST, Destination: JFK
4. Departure: Tomorrow, Return: +7 days
5. Adults: 2, Children: 1, Infants: 0
6. Cabin: Economy
7. Click "Search Flights"

**Expected Results:**
- ✅ Search results show both directions
- ✅ Price calculated for all passengers
- ✅ Airline filters work
- ✅ Cabin class filter works
- ✅ Duration sorting works

**Status:** [ ] Pass [ ] Fail

---

### Test Case 2.2: Flight Booking - Multi Passenger
**Steps:**
1. Click on any flight from results
2. Review flight details
3. Click "Book Now"
4. Fill passenger 1 (adult) details + passport
5. Fill passenger 2 (adult) details + passport
6. Fill passenger 3 (child) details + passport
7. Select meal preferences
8. Select seat preferences
9. Click "Continue to Payment"

**Expected Results:**
- ✅ 3 passenger forms displayed
- ✅ Child age validation (2-11 years)
- ✅ Passport expiry validation
- ✅ Price breakdown shows:
  - Adult × 2 = full price × 2
  - Child × 1 = reduced price
- ✅ Total calculated correctly

**Status:** [ ] Pass [ ] Fail

---

### Test Case 2.3: One-Way Flight with Infant
**Steps:**
1. Select "One Way"
2. Adults: 1, Infants: 1
3. Search and book
4. Verify infant pricing (10% of adult)

**Expected Results:**
- ✅ Infant form requires birth date (under 2 years)
- ✅ Infant price = adult price × 0.1
- ✅ No seat selection for infant

**Status:** [ ] Pass [ ] Fail

---

## 3️⃣ Tours System Testing

### Test Case 3.1: Tour Search & Filters
**Steps:**
1. Navigate to http://localhost:3060/tours
2. Search text: "Cappadocia"
3. Filter by category: "Adventure"
4. Set price range: $50-$200
5. Select city: "Cappadocia"

**Expected Results:**
- ✅ Tours filtered correctly
- ✅ Search matches name/description/city
- ✅ Price slider works
- ✅ Reset filters clears all
- ✅ Sort options work

**Status:** [ ] Pass [ ] Fail

---

### Test Case 3.2: Tour Booking
**Steps:**
1. Click "Cappadocia Hot Air Balloon"
2. Select date: Tomorrow
3. Set travelers: 4
4. Verify price: base_price × 4 × 1.1 (service fee)
5. Click "Book Now"
6. Fill 4 traveler forms
7. Add dietary: "Vegetarian"
8. Add accessibility: "Wheelchair access needed"
9. Click "Continue to Payment"

**Expected Results:**
- ✅ Date selection required to enable booking
- ✅ Traveler count updates price dynamically
- ✅ Service fee (10%) applied
- ✅ All traveler forms validate
- ✅ Special requests captured

**Status:** [ ] Pass [ ] Fail

---

### Test Case 3.3: Tour Categories
**Steps:**
1. Test each category filter:
   - Cultural
   - Adventure
   - Nature
   - Historical
   - Food & Wine

**Expected Results:**
- ✅ Each category shows relevant tours only
- ✅ Multiple categories can be tested
- ✅ Category badges display correctly

**Status:** [ ] Pass [ ] Fail

---

## 4️⃣ Transfers System Testing

### Test Case 4.1: Airport Transfer Search
**Steps:**
1. Navigate to http://localhost:3060/transfers
2. Pickup: "Istanbul Airport (IST)"
3. Dropoff: "Sultanahmet Hotels"
4. Date: Tomorrow, Time: 14:00
5. Passengers: 3
6. Click "Search Transfers"

**Expected Results:**
- ✅ Results show vehicles with capacity ≥ 3
- ✅ Vehicle types displayed (Sedan, SUV, Van)
- ✅ Features list shown
- ✅ Duration and price visible

**Status:** [ ] Pass [ ] Fail

---

### Test Case 4.2: Transfer Booking with Flight Details
**Steps:**
1. Select "Mercedes E-Class"
2. Fill passenger details
3. Add flight number: "TK001"
4. Add arrival time: "13:30"
5. Request 2 child seats
6. Check "Wheelchair accessible"
7. Add note: "Driver speaks English"
8. Click "Continue to Payment"

**Expected Results:**
- ✅ Flight details section visible
- ✅ Child seat counter works (0-4)
- ✅ Wheelchair checkbox functional
- ✅ Additional requests text captured
- ✅ Total price includes all options

**Status:** [ ] Pass [ ] Fail

---

### Test Case 4.3: Vehicle Type Filters
**Steps:**
1. Filter by "Luxury"
2. Filter by "Van"
3. Set price range
4. Test capacity filter

**Expected Results:**
- ✅ Luxury vehicles show premium options
- ✅ Vans show high capacity (7-8 passengers)
- ✅ Price filter works correctly
- ✅ Capacity matches passengers needed

**Status:** [ ] Pass [ ] Fail

---

## 5️⃣ Authentication Testing

### Test Case 5.1: User Registration
**Steps:**
1. Navigate to http://localhost:3060/register
2. Email: test@example.com
3. Password: TestPass123!
4. Confirm password: TestPass123!
5. Full name: Test User
6. Phone: +905551234567
7. Click "Register"

**Expected Results:**
- ✅ Email validation works
- ✅ Password strength requirements enforced
- ✅ Password match validation
- ✅ Phone format validation
- ✅ Successful registration redirects to dashboard
- ✅ User record created in database

**Status:** [ ] Pass [ ] Fail

---

### Test Case 5.2: User Login
**Steps:**
1. Navigate to http://localhost:3060/login
2. Email: test@example.com
3. Password: TestPass123!
4. Click "Login"

**Expected Results:**
- ✅ Successful login redirects to dashboard
- ✅ Invalid credentials show error
- ✅ Session persists on refresh
- ✅ Protected routes accessible

**Status:** [ ] Pass [ ] Fail

---

### Test Case 5.3: Protected Routes
**Steps:**
1. Logout
2. Try to access http://localhost:3060/dashboard
3. Try to book any service

**Expected Results:**
- ✅ Redirects to login page
- ✅ Redirect parameter preserved (?redirect=/dashboard)
- ✅ After login, returns to original page
- ✅ Booking flows require authentication

**Status:** [ ] Pass [ ] Fail

---

## 6️⃣ Payment Integration Testing

### Test Case 6.1: Payment Page
**Steps:**
1. Complete any booking flow
2. Reach payment page
3. Verify booking summary
4. Check Iyzico form loads

**Expected Results:**
- ✅ Booking reference displayed
- ✅ Service type shown correctly
- ✅ Price breakdown accurate
- ✅ Total matches booking amount
- ✅ Iyzico payment form visible
- ✅ 3D Secure ready

**Status:** [ ] Pass [ ] Fail

---

### Test Case 6.2: Payment Success Flow
**Steps:**
1. Complete test payment (sandbox mode)
2. Verify success page
3. Check booking status in database
4. Check user dashboard

**Expected Results:**
- ✅ Success page shows booking reference
- ✅ Booking status = "confirmed"
- ✅ Payment status = "completed"
- ✅ Booking appears in user dashboard
- ✅ Email confirmation sent (if enabled)

**Status:** [ ] Pass [ ] Fail

---

## 7️⃣ User Dashboard Testing

### Test Case 7.1: Profile Management
**Steps:**
1. Login and go to http://localhost:3060/dashboard
2. Update full name
3. Update phone number
4. Save changes

**Expected Results:**
- ✅ Profile data loads correctly
- ✅ Form validation works
- ✅ Updates save to database
- ✅ Success message shown
- ✅ Data persists on refresh

**Status:** [ ] Pass [ ] Fail

---

### Test Case 7.2: Bookings History
**Steps:**
1. Go to "My Bookings" tab
2. Review all bookings
3. Filter by service type
4. Click on booking to view details

**Expected Results:**
- ✅ All user bookings displayed
- ✅ Correct service type badges
- ✅ Status colors accurate
- ✅ Booking details modal works
- ✅ Dates formatted correctly

**Status:** [ ] Pass [ ] Fail

---

## 8️⃣ Admin Panel Testing

### Test Case 8.1: Admin Access Control
**Steps:**
1. Try to access /admin without login
2. Login as regular user, try /admin
3. Login as admin@healmedy.com
4. Access /admin

**Expected Results:**
- ✅ No login → redirect to /login
- ✅ Regular user → redirect to /dashboard
- ✅ Admin user → access granted
- ✅ Admin layout displays

**Status:** [ ] Pass [ ] Fail

---

### Test Case 8.2: Admin Dashboard
**Steps:**
1. Navigate to http://localhost:3060/admin
2. Review all statistics
3. Check recent bookings
4. Test quick actions

**Expected Results:**
- ✅ Total revenue calculated correctly
- ✅ Booking counts accurate
- ✅ Service breakdown correct
- ✅ Recent bookings list populated
- ✅ Total users count accurate
- ✅ Quick action links work

**Status:** [ ] Pass [ ] Fail

---

### Test Case 8.3: Bookings Management
**Steps:**
1. Go to http://localhost:3060/admin/bookings
2. Search by booking reference
3. Filter by type: "flight"
4. Filter by status: "pending"
5. Click on a booking
6. Change status to "confirmed"
7. Export CSV

**Expected Results:**
- ✅ Search filters results
- ✅ Type filter works
- ✅ Status filter works
- ✅ Statistics update dynamically
- ✅ Booking details modal opens
- ✅ Status update saves to database
- ✅ CSV download works

**Status:** [ ] Pass [ ] Fail

---

### Test Case 8.4: Users Management
**Steps:**
1. Go to http://localhost:3060/admin/users
2. Search for user by email
3. Click on user to view details
4. Review booking history
5. Check statistics
6. Export CSV

**Expected Results:**
- ✅ All users listed
- ✅ Search filters correctly
- ✅ User stats calculated (bookings, spent)
- ✅ User details modal shows complete info
- ✅ Booking history per user accurate
- ✅ CSV export works

**Status:** [ ] Pass [ ] Fail

---

### Test Case 8.5: Settings Management
**Steps:**
1. Go to http://localhost:3060/admin/settings
2. Update API keys (dummy values)
3. Change currency
4. Toggle notifications
5. Modify payment settings
6. Click "Save Changes"

**Expected Results:**
- ✅ All settings sections visible
- ✅ Form fields editable
- ✅ Toggles work
- ✅ Save button triggers action
- ✅ Success message displays

**Status:** [ ] Pass [ ] Fail

---

## 9️⃣ Mobile Responsiveness Testing

### Test Case 9.1: Mobile Layout - All Pages
**Device Sizes to Test:**
- iPhone SE (375×667)
- iPhone 12 Pro (390×844)
- iPad (768×1024)
- Desktop (1920×1080)

**Pages to Test:**
- [ ] Homepage
- [ ] Hotels search & details
- [ ] Flights search & details
- [ ] Tours listing & details
- [ ] Transfers search
- [ ] Booking forms
- [ ] Payment page
- [ ] Dashboard
- [ ] Admin panel

**Expected Results:**
- ✅ All content readable
- ✅ Forms usable on mobile
- ✅ Navigation accessible
- ✅ Images scale properly
- ✅ No horizontal scroll
- ✅ Touch targets adequate size

**Status:** [ ] Pass [ ] Fail

---

## 🔟 Cross-Browser Testing

### Browsers to Test:
- [ ] Chrome (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest)
- [ ] Edge (latest)

### Key Flows:
- [ ] Search and booking
- [ ] Authentication
- [ ] Payment flow
- [ ] Admin panel

**Status:** [ ] Pass [ ] Fail

---

## 🔴 Critical Issues Found

| Issue # | Description | Severity | Status | Resolution |
|---------|-------------|----------|--------|------------|
| | | | | |

---

## 🟡 Minor Issues Found

| Issue # | Description | Priority | Status | Resolution |
|---------|-------------|----------|--------|------------|
| | | | | |

---

## 📊 Testing Summary

### Overall Results
- **Total Test Cases:** 30+
- **Passed:** _____
- **Failed:** _____
- **Blocked:** _____
- **Not Tested:** _____

### Pass Rate
- **Target:** 95%
- **Actual:** _____%

### Critical Paths Status
- [ ] Hotel booking end-to-end
- [ ] Flight booking end-to-end
- [ ] Tour booking end-to-end
- [ ] Transfer booking end-to-end
- [ ] Payment processing
- [ ] Admin management

---

## 🚀 Sign-Off

### Tester Information
- **Name:** _______________
- **Date:** _______________
- **Environment:** Development / Staging / Production
- **Build Version:** v1.0.0

### Approval
- [ ] All critical paths tested
- [ ] No blocking issues
- [ ] Ready for production

**Signature:** _______________

---

## 📝 Notes

Additional observations, edge cases, or recommendations:

```
[Add notes here]
```

---

**Last Updated:** 2025-11-03
**Version:** 1.0.0
