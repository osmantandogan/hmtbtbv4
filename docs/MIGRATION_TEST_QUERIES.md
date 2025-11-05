# Migration Test Queries

## Test These Queries Before Running Migrations

### 1. Check Existing Tables

```sql
-- Check if tables already exist
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
AND table_name IN ('payment_methods', 'payment_transactions', 'refund_requests', 'payment_retries', 'reviews')
ORDER BY table_name;
```

### 2. Check Profiles Table Structure

```sql
-- Verify profiles table columns
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'profiles'
ORDER BY ordinal_position;
```

### 3. Check Bookings Table Structure

```sql
-- Verify bookings table has user_id
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'bookings'
AND column_name = 'user_id';
```

### 4. Check Reviews Table Structure

```sql
-- Verify reviews table exists and structure
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'reviews'
ORDER BY ordinal_position;
```

### 5. Drop Existing Views (If Rerunning)

```sql
-- Drop views if they exist
DROP VIEW IF EXISTS payment_statistics CASCADE;
DROP VIEW IF EXISTS review_statistics CASCADE;
```

### 6. Check Existing Policies

```sql
-- Check existing policies on reviews table
SELECT policyname, cmd, qual
FROM pg_policies
WHERE tablename = 'reviews';
```

## Migration Order

Apply in this exact order:

1. **20250106_email_verification.sql**
   - Creates: email_verification_tokens table
   - Adds: email_verified, email_verified_at to profiles

2. **20250106_enhance_reviews.sql** (FIXED)
   - Adds columns to existing reviews table
   - Creates: review_statistics view
   - Updates: RLS policies

3. **20250106_notifications_system.sql**
   - Creates: notifications table
   - Creates: notification_preferences table
   - Creates: notification functions

4. **20250106_payment_improvements.sql** (FIXED)
   - Creates: payment_methods table
   - Creates: payment_transactions table
   - Creates: refund_requests table
   - Creates: payment_retries table
   - Creates: payment_statistics view
   - Adds: payment_status, payment_transaction_id to bookings

## Common Errors & Solutions

### Error: "column profiles.role does not exist"
**Fix**: Remove admin role check from RLS policy
**Status**: ✅ FIXED in enhanced_reviews migration

### Error: "column user_id does not exist"
**Cause**: View created before table or table not applied yet
**Solution**: Ensure payment_transactions table is created first
**Status**: ✅ Should be fixed (table created before view)

### Error: "FILTER specified, but round is not an aggregate function"
**Fix**: Move FILTER inside AVG() before ROUND()
**Status**: ✅ FIXED in enhanced_reviews migration

### Error: "relation already exists"
**Solution**: Add IF NOT EXISTS to CREATE TABLE
**Status**: ✅ All tables use IF NOT EXISTS

### Error: "policy already exists"
**Solution**: Add DROP POLICY IF EXISTS before CREATE POLICY
**Status**: ✅ All policies have DROP IF EXISTS

## Verification After Migration

### Verify Enhanced Reviews

```sql
-- Check new columns exist
SELECT column_name FROM information_schema.columns
WHERE table_name = 'reviews'
AND column_name IN ('photos', 'status', 'rating_cleanliness', 'rating_location', 'rating_service');

-- Check view exists
SELECT * FROM review_statistics LIMIT 1;

-- Check RLS policies
SELECT policyname FROM pg_policies WHERE tablename = 'reviews';
```

### Verify Payment Tables

```sql
-- Check all payment tables exist
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public'
AND table_name LIKE 'payment%'
OR table_name = 'refund_requests';

-- Check view exists
SELECT * FROM payment_statistics LIMIT 1;

-- Check bookings columns
SELECT column_name FROM information_schema.columns
WHERE table_name = 'bookings'
AND column_name IN ('payment_status', 'payment_transaction_id');
```

### Verify Notifications

```sql
-- Check notifications table
SELECT * FROM notifications LIMIT 1;

-- Check preferences
SELECT * FROM notification_preferences LIMIT 1;

-- Test notification function
SELECT create_notification(
    auth.uid()::uuid,
    'booking_confirmed',
    'Test Notification',
    'This is a test message',
    '{}'::jsonb
);
```

### Verify Email Verification

```sql
-- Check email_verification_tokens table
SELECT * FROM email_verification_tokens WHERE used = false LIMIT 1;

-- Check profiles columns
SELECT email_verified, email_verified_at
FROM profiles
WHERE email_verified = true
LIMIT 1;
```

## Rollback Procedure

If migrations fail, rollback in reverse order:

### 1. Rollback Payment Improvements

```sql
DROP TABLE IF EXISTS payment_retries CASCADE;
DROP TABLE IF EXISTS refund_requests CASCADE;
DROP TABLE IF EXISTS payment_transactions CASCADE;
DROP TABLE IF EXISTS payment_methods CASCADE;
DROP VIEW IF EXISTS payment_statistics;
DROP FUNCTION IF EXISTS ensure_single_default_payment_method();
DROP FUNCTION IF EXISTS schedule_payment_retry(UUID, UUID, INTEGER);
DROP FUNCTION IF EXISTS process_refund(UUID, BOOLEAN, UUID, TEXT);

ALTER TABLE bookings DROP COLUMN IF EXISTS payment_status;
ALTER TABLE bookings DROP COLUMN IF EXISTS payment_transaction_id;
```

### 2. Rollback Notifications System

```sql
DROP TABLE IF EXISTS notifications CASCADE;
DROP TABLE IF EXISTS notification_preferences CASCADE;
DROP TYPE IF EXISTS notification_type;
DROP FUNCTION IF EXISTS create_notification(UUID, notification_type, TEXT, TEXT, JSONB);
DROP FUNCTION IF EXISTS mark_notification_read(UUID);
DROP FUNCTION IF EXISTS get_unread_count(UUID);
```

### 3. Rollback Enhanced Reviews

```sql
ALTER TABLE reviews DROP COLUMN IF EXISTS photos;
ALTER TABLE reviews DROP COLUMN IF EXISTS rating_cleanliness;
ALTER TABLE reviews DROP COLUMN IF EXISTS rating_location;
ALTER TABLE reviews DROP COLUMN IF EXISTS rating_service;
ALTER TABLE reviews DROP COLUMN IF EXISTS rating_value;
ALTER TABLE reviews DROP COLUMN IF EXISTS rating_amenities;
ALTER TABLE reviews DROP COLUMN IF EXISTS status;
ALTER TABLE reviews DROP COLUMN IF EXISTS moderation_reason;
ALTER TABLE reviews DROP COLUMN IF EXISTS moderated_at;
ALTER TABLE reviews DROP COLUMN IF EXISTS moderated_by;
ALTER TABLE reviews DROP COLUMN IF EXISTS response_text;
ALTER TABLE reviews DROP COLUMN IF EXISTS response_by;
ALTER TABLE reviews DROP COLUMN IF EXISTS response_date;

DROP VIEW IF EXISTS review_statistics;
DROP INDEX IF EXISTS idx_reviews_status;
DROP INDEX IF EXISTS idx_reviews_photos;
DROP INDEX IF EXISTS idx_reviews_response_date;
```

### 4. Rollback Email Verification

```sql
DROP TABLE IF EXISTS email_verification_tokens CASCADE;
ALTER TABLE profiles DROP COLUMN IF EXISTS email_verified;
ALTER TABLE profiles DROP COLUMN IF EXISTS email_verified_at;
```

## Final Checklist

After all migrations applied:

- [ ] All 4 migrations completed without errors
- [ ] All new tables exist (run SELECT table_name query)
- [ ] All new views work (run SELECT * FROM view_name LIMIT 1)
- [ ] RLS policies active (run pg_policies query)
- [ ] Indexes created (run \\d table_name in psql)
- [ ] Functions work (test with sample data)
- [ ] No policy conflicts
- [ ] Build succeeds (npm run build)
- [ ] Application starts (npm run dev)
- [ ] Test key features work

---

**Status**: Ready for Migration
**Last Updated**: January 6, 2025
