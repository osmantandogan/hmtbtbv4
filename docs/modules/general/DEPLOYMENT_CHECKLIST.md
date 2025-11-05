# HealMedy Travel - Production Deployment Checklist

## 🚀 Pre-Deployment Preparation

### 1. Code Review & Testing
- [ ] All test cases passed (see TESTING_GUIDE.md)
- [ ] No console errors in browser
- [ ] All booking flows tested end-to-end
- [ ] Mobile responsiveness verified
- [ ] Cross-browser testing complete
- [ ] No TypeScript errors (`npm run build`)
- [ ] Code reviewed and approved

### 2. Environment Configuration

#### Required Environment Variables
Create `.env.production` file with:

```env
# Supabase (Production)
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_production_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_production_service_key

# KPLUS API (Hotels) - Production
KPLUS_BASE_URL=https://api.kplus.com
KPLUS_API_KEY=your_production_kplus_key

# BTB API (Flights, Tours, Transfers) - Production
BTB_BASE_URL=https://tourbridge.production.com/api
BTB_API_KEY=your_production_btb_key

# Iyzico Payment Gateway - Production
IYZICO_API_KEY=your_production_iyzico_key
IYZICO_SECRET_KEY=your_production_iyzico_secret
IYZICO_BASE_URL=https://api.iyzipay.com

# Application Settings
NEXT_PUBLIC_SITE_URL=https://healmedy.com
NEXT_PUBLIC_ENVIRONMENT=production
```

- [ ] All environment variables configured
- [ ] API keys validated in production
- [ ] No hardcoded secrets in code
- [ ] `.env` files in `.gitignore`

### 3. API Integration

#### Test API Connectivity
```bash
# Test BTB API
node scripts/test-api-connection.js

# Expected: All endpoints return 200 OK
```

#### Enable Real API
```bash
# Switch from mock to real API
node scripts/enable-real-api.js enable
```

- [ ] BTB API connectivity tested
- [ ] KPLUS API connectivity tested
- [ ] Real API enabled in all services
- [ ] Demo mode badges removed
- [ ] API rate limits understood
- [ ] Error handling verified

### 4. Database Setup

#### Supabase Production Database
- [ ] Production project created
- [ ] Database schema deployed
- [ ] Row Level Security (RLS) policies configured
- [ ] Authentication settings configured
- [ ] Storage buckets created (if needed)
- [ ] Database backups enabled

#### Run Migrations
```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  phone TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Bookings table
CREATE TABLE bookings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  booking_type TEXT NOT NULL,
  booking_reference TEXT UNIQUE NOT NULL,
  booking_details JSONB,
  total_price DECIMAL(10,2),
  status TEXT DEFAULT 'pending',
  payment_status TEXT DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_bookings_user_id ON bookings(user_id);
CREATE INDEX idx_bookings_status ON bookings(status);
CREATE INDEX idx_bookings_reference ON bookings(booking_reference);
```

- [ ] Tables created
- [ ] Indexes added
- [ ] Test data cleared
- [ ] RLS policies tested

### 5. Payment Gateway Configuration

#### Iyzico Production Setup
- [ ] Production account activated
- [ ] API keys obtained
- [ ] Webhook URLs configured
- [ ] 3D Secure enabled
- [ ] Test transaction completed
- [ ] Callback URLs whitelisted
- [ ] Currency settings verified

#### Payment Testing
- [ ] Test card: Successful payment
- [ ] Test card: Failed payment
- [ ] 3D Secure flow works
- [ ] Webhook callbacks received
- [ ] Refund process tested

### 6. Admin Configuration

#### Admin Access
Edit `app/admin/layout.tsx`:
```typescript
const adminEmails = [
  "admin@healmedy.com",
  "osman@healmedy.com",
  // Add production admin emails
];
```

- [ ] Admin emails configured
- [ ] Admin accounts created
- [ ] Admin panel access tested
- [ ] Admin permissions verified

### 7. Security Checklist

- [ ] All API keys in environment variables
- [ ] No sensitive data in client code
- [ ] HTTPS enforced
- [ ] CORS configured correctly
- [ ] Rate limiting implemented
- [ ] SQL injection prevention verified
- [ ] XSS protection enabled
- [ ] CSRF tokens configured
- [ ] Authentication tokens secure
- [ ] Password requirements enforced

### 8. Performance Optimization

- [ ] Images optimized
- [ ] Code splitting implemented
- [ ] Lazy loading configured
- [ ] Bundle size optimized
- [ ] CDN configured (if applicable)
- [ ] Caching strategies implemented
- [ ] Database queries optimized
- [ ] Lighthouse score > 90

### 9. SEO Configuration

- [ ] Meta tags configured
- [ ] Open Graph tags added
- [ ] Sitemap generated
- [ ] robots.txt configured
- [ ] Google Analytics setup
- [ ] Google Search Console verified
- [ ] Structured data added

---

## 📦 Deployment to Vercel

### Step 1: Prepare Repository
```bash
# Ensure clean git state
git status

# Commit all changes
git add .
git commit -m "Production ready deployment"

# Push to GitHub
git push origin main
```

- [ ] Code committed
- [ ] Pushed to GitHub
- [ ] No uncommitted changes

### Step 2: Vercel Setup
```bash
# Install Vercel CLI (if not installed)
npm install -g vercel

# Login to Vercel
vercel login

# Deploy
vercel --prod
```

**Or use Vercel Dashboard:**
1. Go to https://vercel.com
2. Import GitHub repository
3. Configure project settings
4. Add environment variables
5. Deploy

- [ ] Vercel project created
- [ ] GitHub connected
- [ ] Build settings configured
- [ ] Environment variables added

### Step 3: Configure Environment Variables in Vercel

Go to Project Settings → Environment Variables and add:

**Production Variables:**
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- `KPLUS_BASE_URL`
- `KPLUS_API_KEY`
- `BTB_BASE_URL`
- `BTB_API_KEY`
- `IYZICO_API_KEY`
- `IYZICO_SECRET_KEY`
- `IYZICO_BASE_URL`
- `NEXT_PUBLIC_SITE_URL`

- [ ] All environment variables added
- [ ] Variables set to "Production" environment
- [ ] Sensitive values encrypted
- [ ] Preview environment configured

### Step 4: Custom Domain Setup

1. Add domain in Vercel dashboard
2. Update DNS records:
   - A record: `76.76.21.21`
   - CNAME: `cname.vercel-dns.com`

- [ ] Domain added to Vercel
- [ ] DNS records updated
- [ ] SSL certificate issued
- [ ] Domain verified

### Step 5: Build Configuration

Verify `package.json` scripts:
```json
{
  "scripts": {
    "dev": "next dev --port 3060",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

- [ ] Build command correct
- [ ] Start command configured
- [ ] Node version specified (18.x)

---

## ✅ Post-Deployment Verification

### Immediate Checks (First 10 minutes)
- [ ] Site loads successfully
- [ ] Homepage renders correctly
- [ ] All routes accessible
- [ ] Assets loading (images, CSS, JS)
- [ ] No console errors
- [ ] SSL certificate valid
- [ ] API endpoints responding

### Functional Testing (First Hour)
- [ ] User registration works
- [ ] User login works
- [ ] Hotel search and booking
- [ ] Flight search and booking
- [ ] Tour listing and booking
- [ ] Transfer booking
- [ ] Payment flow completes
- [ ] Admin panel accessible
- [ ] Database writes successful
- [ ] Email notifications sent (if enabled)

### Performance Checks
```bash
# Run Lighthouse audit
npm install -g lighthouse

lighthouse https://healmedy.com --view
```

**Target Scores:**
- Performance: > 90
- Accessibility: > 95
- Best Practices: > 95
- SEO: > 95

- [ ] Lighthouse scores meet targets
- [ ] Page load times < 3s
- [ ] Time to Interactive < 5s
- [ ] No performance bottlenecks

### Monitoring Setup
- [ ] Error tracking (Sentry)
- [ ] Analytics (Google Analytics)
- [ ] Uptime monitoring (UptimeRobot)
- [ ] Performance monitoring (Vercel Analytics)
- [ ] Log aggregation configured
- [ ] Alert thresholds set

---

## 🔄 Post-Deployment Tasks

### First 24 Hours
- [ ] Monitor error logs
- [ ] Check booking success rate
- [ ] Review payment transactions
- [ ] Test email delivery
- [ ] Monitor API usage
- [ ] Check database performance
- [ ] Verify backup systems

### First Week
- [ ] Analyze user behavior
- [ ] Review conversion rates
- [ ] Check payment success rates
- [ ] Monitor API rate limits
- [ ] Review error patterns
- [ ] Gather user feedback
- [ ] Plan improvements

### Documentation
- [ ] Update README with production URL
- [ ] Document deployment process
- [ ] Create runbook for incidents
- [ ] Document rollback procedure
- [ ] Update API documentation
- [ ] Create user guides

---

## 🔴 Rollback Procedure

If critical issues are found:

### Option 1: Revert to Previous Deployment
```bash
# In Vercel dashboard:
# 1. Go to Deployments
# 2. Find last stable deployment
# 3. Click "Promote to Production"
```

### Option 2: Rollback Git
```bash
# Revert last commit
git revert HEAD

# Push to trigger new deployment
git push origin main
```

### Option 3: Disable Real API
```bash
# Switch back to mock data
node scripts/enable-real-api.js disable

# Commit and deploy
git add .
git commit -m "Rollback to mock data"
git push origin main
```

- [ ] Rollback procedure tested
- [ ] Team notified of rollback
- [ ] Issue documented
- [ ] Fix scheduled

---

## 📞 Emergency Contacts

**Technical Team:**
- Developer: [Contact]
- DevOps: [Contact]
- Database Admin: [Contact]

**Service Providers:**
- Vercel Support: https://vercel.com/support
- Supabase Support: https://supabase.com/support
- Iyzico Support: [Contact]
- BTB API Support: [Contact]

---

## 📋 Deployment Sign-Off

### Pre-Deployment Approval
- [ ] Developer: _________________ Date: _______
- [ ] QA Lead: __________________ Date: _______
- [ ] Project Manager: ___________ Date: _______

### Post-Deployment Verification
- [ ] All checks passed
- [ ] Monitoring active
- [ ] Team notified
- [ ] Documentation updated

**Deployed By:** _________________
**Deployment Date:** _________________
**Deployment Time:** _________________
**Build Version:** v1.0.0
**Git Commit:** _________________

---

## 🎯 Success Criteria

Deployment is considered successful when:
- ✅ All critical paths working
- ✅ No P0/P1 bugs
- ✅ Performance targets met
- ✅ Security audit passed
- ✅ Monitoring active
- ✅ Team trained
- ✅ Documentation complete

---

**Last Updated:** 2025-11-03
**Version:** 1.0.0
**Next Review:** [Date]
