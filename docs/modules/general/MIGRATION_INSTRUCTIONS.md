# 🚀 Quick Database Migration Instructions

## Step 1: Run the Migration

You have 2 options:

### Option A: Via Supabase Dashboard (Recommended - 2 minutes)

1. **Open Supabase Dashboard**:
   - Go to: https://supabase.com/dashboard/project/ytgerojajvgyywamdfjm
   - Or: https://supabase.com/dashboard/projects → Select your project

2. **Open SQL Editor**:
   - Click "SQL Editor" in the left sidebar
   - Click "New Query" button

3. **Copy & Paste Migration**:
   - Open file: `supabase/migrations/20250103000000_initial_schema.sql`
   - Select ALL content (Ctrl+A)
   - Copy (Ctrl+C)
   - Paste in SQL Editor (Ctrl+V)

4. **Run the Migration**:
   - Click "Run" button (or press Ctrl+Enter)
   - Wait for "Success. No rows returned"

5. **Verify Tables Created**:
   - Click "Table Editor" in left sidebar
   - You should see 6 tables:
     - ✓ profiles
     - ✓ kplus_tokens
     - ✓ hotels
     - ✓ hotel_rooms
     - ✓ bookings
     - ✓ payment_transactions

### Option B: Via Command Line (Advanced)

```bash
# Install Supabase CLI
npm install -g supabase

# Login to Supabase
supabase login

# Link to project
supabase link --project-ref ytgerojajvgyywamdfjm

# Push migration
supabase db push
```

---

## Step 2: Get Service Role Key (Optional but Recommended)

For admin operations like caching hotel data:

1. Go to Project Settings → API
2. Find "service_role" key
3. Click "Reveal" and copy
4. Update `.env.local`:
   ```
   SUPABASE_SERVICE_ROLE_KEY=your-actual-service-role-key
   ```

---

## Step 3: Restart Dev Server

After migration:

```bash
# Stop current server (Ctrl+C)
# Then restart:
npm run dev
```

The middleware errors should disappear!

---

## ✅ How to Verify Migration Succeeded

### Check 1: Table Editor
- Go to Supabase Dashboard → Table Editor
- See all 6 tables listed

### Check 2: SQL Query
Run this in SQL Editor:
```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public';
```

You should see:
- profiles
- kplus_tokens
- hotels
- hotel_rooms
- bookings
- payment_transactions

### Check 3: Test in App
After restart, the app should connect without errors.

---

## 🆘 Troubleshooting

**Problem**: "relation already exists" error
- **Solution**: Tables already created, skip to Step 2

**Problem**: "permission denied"
- **Solution**: Make sure you're logged into correct Supabase account

**Problem**: Still seeing middleware errors
- **Solution**: Restart dev server with Ctrl+C then `npm run dev`

---

## 📝 After Migration

Once complete, let me know and I'll:
1. ✅ Test the database connection
2. ✅ Connect KPLUS API for real hotel data
3. ✅ Build authentication pages
4. ✅ Add real payment processing

---

**Ready? Run the migration and let me know when done!** 🚀
