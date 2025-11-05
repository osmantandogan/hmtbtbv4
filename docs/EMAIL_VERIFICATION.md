# Email Verification System

## Overview
The email verification system ensures that users verify their email addresses after registration. This is a critical security feature that prevents spam accounts and ensures users have access to their registered email.

## Features Implemented

### 1. Database Schema
- **email_verification_tokens** table: Stores verification tokens with expiration
- **profiles** table: Extended with `email_verified` and `email_verified_at` fields
- Row Level Security (RLS) policies for secure token access

### 2. Email Service
Located in `lib/email/verification.ts`:
- `sendVerificationEmail()`: Sends HTML email with verification link
- `createVerificationToken()`: Generates cryptographically secure tokens
- `verifyEmailToken()`: Validates tokens and marks email as verified
- `resendVerificationEmail()`: Resends verification for unverified users

### 3. API Endpoints

#### POST /api/auth/send-verification
Sends a verification email to the authenticated user.
- **Authentication**: Required
- **Response**: `{ success: boolean, message: string, email: string }`

#### POST /api/auth/verify-token
Verifies the email verification token.
- **Body**: `{ token: string }`
- **Response**: `{ success: boolean, message: string, userId: string }`

### 4. User Interface Components

#### Verification Page (`/auth/verify-email`)
- Automatically verifies token from URL query parameter
- Shows loading, success, or error states
- Redirects to dashboard after successful verification

#### EmailVerificationBanner Component
- Displays warning banner for unverified emails
- Allows users to resend verification email
- Shows success/error feedback
- Can be dismissed by user

### 5. Automatic Email Sending
The system automatically sends verification emails when:
- User completes registration (in `AuthContext`)
- User requests resend via banner or API

## Database Migration

### Run Migration
The migration file is located at: `supabase/migrations/20250106_email_verification.sql`

To apply the migration:

```sql
-- Add email verification fields to profiles
ALTER TABLE profiles
ADD COLUMN IF NOT EXISTS email_verified BOOLEAN DEFAULT FALSE,
ADD COLUMN IF NOT EXISTS email_verified_at TIMESTAMPTZ;

-- Create email_verification_tokens table
CREATE TABLE IF NOT EXISTS email_verification_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  token TEXT NOT NULL UNIQUE,
  used BOOLEAN DEFAULT FALSE,
  expires_at TIMESTAMPTZ NOT NULL,
  verified_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create indexes
CREATE INDEX IF NOT EXISTS idx_email_verification_tokens_token ON email_verification_tokens(token);
CREATE INDEX IF NOT EXISTS idx_email_verification_tokens_user_id ON email_verification_tokens(user_id);
```

Alternatively, run via Supabase dashboard:
1. Go to SQL Editor
2. Copy content from `supabase/migrations/20250106_email_verification.sql`
3. Execute the SQL

## Usage Examples

### 1. Add Email Verification Banner to Dashboard

```typescript
import EmailVerificationBanner from "@/components/auth/EmailVerificationBanner";

export default function DashboardPage() {
  const { user, profile } = useAuth();

  return (
    <div>
      <EmailVerificationBanner
        userEmail={profile?.email}
        isVerified={profile?.email_verified}
      />
      {/* Rest of dashboard */}
    </div>
  );
}
```

### 2. Check Email Verification in API Routes

```typescript
import { createClient } from '@/lib/supabase/server';

export async function POST(request: NextRequest) {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();

  // Get user profile with verification status
  const { data: profile } = await supabase
    .from('profiles')
    .select('email_verified')
    .eq('id', user.id)
    .single();

  if (!profile.email_verified) {
    return NextResponse.json(
      { error: 'Please verify your email address first' },
      { status: 403 }
    );
  }

  // Continue with API logic...
}
```

### 3. Manual Verification Email Send

```typescript
// Send verification email via API
const response = await fetch('/api/auth/send-verification', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
});

const data = await response.json();
if (data.success) {
  console.log('Verification email sent to:', data.email);
}
```

## Email Template
The verification emails include:
- Professional HealMedy Travel branding
- Clear call-to-action button
- Alternative text link
- Security notice (24-hour expiration)
- Benefits of verification
- Support contact information

## Security Features

1. **Token Expiration**: Tokens expire after 24 hours
2. **One-Time Use**: Tokens can only be used once
3. **Secure Generation**: Cryptographically secure random tokens (32 bytes)
4. **RLS Policies**: Database-level security for token access
5. **Automatic Invalidation**: Old tokens invalidated when new ones are created

## Configuration

Email service is configured via environment variables in `.env.local`:

```env
# Email Service (Hostinger SMTP)
SMTP_HOST=smtp.hostinger.com
SMTP_PORT=465
SMTP_SECURE=true
SMTP_USER=info@healmedy.travel
SMTP_PASSWORD=Osman.117
EMAIL_FROM=info@healmedy.travel
```

## Testing

### Test Email Sending
1. Register a new user account
2. Check email inbox (and spam folder)
3. Click verification link
4. Verify user is redirected to dashboard
5. Check profile `email_verified` is set to `true`

### Test Resend Functionality
1. Login with unverified account
2. View banner showing unverified status
3. Click "Resend Verification Email"
4. Check email inbox for new verification link

### Test Token Expiration
1. Generate verification token
2. Wait 24 hours (or manually update expires_at in database)
3. Try to verify - should fail with "Token expired" message

## Future Enhancements

Potential improvements for the email verification system:

1. **Email Change Verification**: Require verification when users change their email
2. **SMS Verification**: Add phone number verification alongside email
3. **Reminder Emails**: Send reminders to users who haven't verified after X days
4. **Admin Override**: Allow admins to manually verify user emails
5. **Verification Required**: Block booking/purchases for unverified users
6. **Email Templates**: Create more email template variations
7. **Metrics**: Track verification rates and email delivery success

## Troubleshooting

### Emails Not Sending
- Check SMTP credentials in `.env.local`
- Verify SMTP server is accessible
- Check server logs for email errors
- Test email service with a simple nodemailer script

### Token Not Found
- Token may have expired (24 hours)
- Token may already be used
- Check `email_verification_tokens` table in database

### Verification Not Updating
- Check database permissions
- Verify RLS policies allow updates
- Check server logs for errors
- Ensure profiles table has `email_verified` columns

## Database Queries for Monitoring

```sql
-- Check unverified users
SELECT id, email, email_verified, created_at
FROM profiles
WHERE email_verified = FALSE
ORDER BY created_at DESC;

-- Check pending verification tokens
SELECT user_id, email, created_at, expires_at, used
FROM email_verification_tokens
WHERE used = FALSE AND expires_at > NOW()
ORDER BY created_at DESC;

-- Check verification rate
SELECT
  COUNT(*) FILTER (WHERE email_verified = TRUE) as verified,
  COUNT(*) FILTER (WHERE email_verified = FALSE) as unverified,
  ROUND(100.0 * COUNT(*) FILTER (WHERE email_verified = TRUE) / COUNT(*), 2) as verification_rate_percent
FROM profiles;
```

## Support

For issues or questions about the email verification system:
- Check server logs for detailed error messages
- Review this documentation
- Contact: info@healmedy.travel
