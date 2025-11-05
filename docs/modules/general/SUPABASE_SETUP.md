# Supabase Database Setup - Quick Guide

## ✅ Step 1: Credentials Updated

Your Supabase credentials have been added to `.env.local`:
- URL: `https://ytgerojajvgyywamdfjm.supabase.co`
- Anon Key: Added ✅

## 📋 Step 2: Run Database Migration

Follow these steps to set up your database tables:

### Option A: Via Supabase Dashboard (Easiest)

1. **Go to your Supabase project**:
   - Visit: https://supabase.com/dashboard/projects
   - Select your project: `ytgerojajvgyywamdfjm`

2. **Open SQL Editor**:
   - Click on "SQL Editor" in the left sidebar
   - Click "New Query"

3. **Copy the migration SQL**:
   - Open this file: `supabase/migrations/20250103000000_initial_schema.sql`
   - Copy ALL the contents (Ctrl+A, Ctrl+C)

4. **Paste and Run**:
   - Paste into the SQL Editor
   - Click "Run" button (or press Ctrl+Enter)
   - Wait for "Success. No rows returned" message

5. **Verify Tables Created**:
   - Click on "Table Editor" in the left sidebar
   - You should see these 6 tables:
     - ✓ profiles
     - ✓ kplus_tokens
     - ✓ hotels
     - ✓ hotel_rooms
     - ✓ bookings
     - ✓ payment_transactions

### Option B: Via Supabase CLI (Advanced)

```bash
# Install Supabase CLI (if not installed)
npm install -g supabase

# Login to Supabase
supabase login

# Link to your project
supabase link --project-ref ytgerojajvgyywamdfjm

# Push the migration
supabase db push
```

## 🔄 Step 3: Restart Development Server

After running the migration:

1. Stop the current dev server (Ctrl+C in terminal)
2. Restart it:
   ```bash
   npm run dev
   ```

The middleware errors should disappear, and Supabase will be fully connected!

## 🎉 What You'll Have After This

Once the migration is complete, you'll have:

✅ **Full database with 6 tables**
✅ **Row Level Security enabled**
✅ **Automatic timestamp triggers**
✅ **Token caching for KPLUS API**
✅ **Ready for hotel bookings**
✅ **Ready for user authentication**

## 🚀 What's Already Working

Even before the migration, you can:

- ✅ View the homepage: http://localhost:3060
- ✅ Browse hotels page: http://localhost:3060/hotels
- ✅ Search for hotels (try the form!)
- ✅ View search results: http://localhost:3060/hotels/search

## ⏭️ Next Steps After Migration

Once Supabase is set up, I'll:

1. Build the Hotel Details page
2. Build the Booking page
3. Connect KPLUS API for real hotel data
4. Add Iyzico payment integration
5. Build user authentication

---

## 📝 Need Help?

If you encounter any issues:

1. **Tables not showing up?**
   - Go to SQL Editor
   - Paste and run the migration again
   - Check for error messages

2. **Service Role Key needed?**
   - Go to Project Settings → API
   - Click "Reveal" on service_role key
   - Update `.env.local` with the value

3. **Still seeing middleware errors?**
   - Restart the dev server
   - Check `.env.local` has correct URL
   - Clear browser cache

---

**Ready to continue?** Once you've run the migration, let me know and I'll build the Hotel Details page! 🏨
