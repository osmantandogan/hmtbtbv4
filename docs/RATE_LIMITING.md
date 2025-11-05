# Rate Limiting System

## Overview
The rate limiting system protects API endpoints from abuse, DDoS attacks, and ensures fair usage. It uses an in-memory LRU cache for development and can be easily switched to Redis for production.

## Features Implemented

### 1. Core Rate Limiting Service
Located in `lib/rate-limit/redis-store.ts`:
- **rateLimit()**: Tracks and enforces rate limits
- **checkRateLimit()**: Check limits without incrementing
- **resetRateLimit()**: Manually reset limits for an identifier
- **getRateLimitStats()**: Monitor cache performance

### 2. Middleware System
Located in `lib/rate-limit/middleware.ts`:
- **applyRateLimit()**: Apply rate limiting to API routes
- **withRateLimit()**: Higher-order function wrapper for routes
- **RateLimitPresets**: Pre-configured limits for different endpoint types

### 3. Rate Limit Presets

The system includes pre-configured rate limits for common use cases:

```typescript
RateLimitPresets.AUTH          // 5 requests per 15 minutes
RateLimitPresets.API           // 100 requests per minute
RateLimitPresets.SEARCH        // 30 requests per minute
RateLimitPresets.PASSWORD_RESET // 3 requests per hour
RateLimitPresets.BOOKING       // 10 requests per 5 minutes
RateLimitPresets.EMAIL         // 5 requests per hour
```

### 4. Response Headers
Rate-limited responses include standard headers:
- `X-RateLimit-Limit`: Maximum requests allowed
- `X-RateLimit-Remaining`: Requests remaining in window
- `X-RateLimit-Reset`: Unix timestamp when limit resets
- `Retry-After`: Seconds until limit resets

## Usage Examples

### Method 1: Using applyRateLimit()

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { applyRateLimit, RateLimitPresets } from '@/lib/rate-limit/middleware';

export async function POST(request: NextRequest) {
  // Apply rate limiting
  const rateLimitResult = await applyRateLimit(request, RateLimitPresets.AUTH);
  if (!rateLimitResult.success) {
    return rateLimitResult.response;
  }

  // Your API logic here
  return NextResponse.json({ success: true });
}
```

### Method 2: Using withRateLimit() Wrapper

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { withRateLimit, RateLimitPresets } from '@/lib/rate-limit/middleware';

export const POST = withRateLimit(
  async (request: NextRequest) => {
    // Your API logic here
    return NextResponse.json({ success: true });
  },
  RateLimitPresets.AUTH
);
```

### Method 3: Custom Rate Limit Configuration

```typescript
import { applyRateLimit } from '@/lib/rate-limit/middleware';

export async function POST(request: NextRequest) {
  const rateLimitResult = await applyRateLimit(request, {
    limit: 50,                    // Max 50 requests
    windowMs: 10 * 60 * 1000,    // Per 10 minutes
    message: 'Custom error message',

    // Custom identifier (e.g., by user ID instead of IP)
    getIdentifier: (req) => {
      const userId = req.headers.get('x-user-id');
      return userId || 'anonymous';
    },

    // Skip rate limiting for certain conditions
    skip: (req) => {
      return req.headers.get('x-api-key') === 'admin-key';
    },
  });

  if (!rateLimitResult.success) {
    return rateLimitResult.response;
  }

  // Your API logic
  return NextResponse.json({ success: true });
}
```

## Implemented Endpoints

Rate limiting has been applied to the following endpoints:

### Authentication Endpoints
- ✅ `POST /api/auth/send-verification` - Email verification (5/hour)

### Search Endpoints
- ✅ `POST /api/hotels/search` - Hotel search (30/minute)

### Booking Endpoints
- ✅ `POST /api/hotels/book` - Hotel booking (10/5min)

### Recommended for Future Implementation
- `POST /api/flights/search` - Flight search (30/minute)
- `POST /api/tours/search` - Tour search (30/minute)
- `POST /api/transfers/search` - Transfer search (30/minute)
- `POST /api/flights/book` - Flight booking (10/5min)
- `POST /api/tours/book` - Tour booking (10/5min)
- `POST /api/transfers/book` - Transfer booking (10/5min)
- `POST /api/auth/login` - Login attempts (5/15min)
- `POST /api/auth/register` - Registration (5/15min)
- `POST /api/auth/reset-password` - Password reset (3/hour)
- `POST /api/payment/*` - Payment endpoints (10/5min)
- `POST /api/reviews/create` - Review creation (5/hour)

## Client Identifier

By default, the system identifies clients by IP address, checking multiple headers to handle proxies:

1. `x-forwarded-for` (first IP in list)
2. `x-real-ip`
3. `cf-connecting-ip` (Cloudflare)
4. Falls back to 'unknown' if none found

### Custom Identifiers

You can customize how clients are identified:

```typescript
// Rate limit by user ID (authenticated users)
getIdentifier: (request) => {
  const userId = request.cookies.get('user_id')?.value;
  return userId || request.headers.get('x-forwarded-for') || 'anonymous';
}

// Rate limit by API key
getIdentifier: (request) => {
  return request.headers.get('x-api-key') || 'anonymous';
}

// Combine multiple factors
getIdentifier: (request) => {
  const ip = request.headers.get('x-forwarded-for');
  const userId = request.cookies.get('user_id')?.value;
  return `${userId || 'anon'}-${ip || 'unknown'}`;
}
```

## Storage Backend

### Current: In-Memory (LRU Cache)
- **Pros**: Fast, no external dependencies
- **Cons**: Doesn't persist across server restarts, not suitable for multi-server deployments
- **Configuration**: 10,000 max items, 1-hour TTL

### Production: Redis (Recommended)

To switch to Redis for production:

```typescript
// lib/rate-limit/redis-store.ts
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function rateLimit(identifier: string, limit: number, windowMs: number) {
  const key = `ratelimit:${identifier}`;
  const now = Date.now();

  // Use Redis MULTI/EXEC for atomic operations
  const pipeline = redis.pipeline();
  pipeline.zadd(key, now, `${now}-${Math.random()}`);
  pipeline.zremrangebyscore(key, 0, now - windowMs);
  pipeline.zcard(key);
  pipeline.pexpire(key, windowMs);

  const results = await pipeline.exec();
  const count = results[2][1] as number;

  return {
    success: count <= limit,
    limit,
    remaining: Math.max(0, limit - count),
    reset: now + windowMs,
  };
}
```

### Environment Variable for Redis:
```env
REDIS_URL=redis://localhost:6379
# OR for Redis Cloud:
REDIS_URL=redis://:password@your-redis-host:6379
```

## Error Handling

When rate limit is exceeded:

**HTTP Response:**
```json
{
  "error": "Too many requests. Please try again later.",
  "retryAfter": 45
}
```

**Status Code:** `429 Too Many Requests`

**Headers:**
```
X-RateLimit-Limit: 30
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1736091234567
Retry-After: 45
```

## Monitoring

### Check Rate Limit Stats

```typescript
import { getRateLimitStats } from '@/lib/rate-limit/redis-store';

const stats = getRateLimitStats();
console.log('Total rate limit keys:', stats.totalKeys);
console.log('Cache size:', stats.cacheSize);
```

### Reset a User's Rate Limit (Admin)

```typescript
import { resetRateLimit } from '@/lib/rate-limit/redis-store';

// Reset by IP
await resetRateLimit('192.168.1.1');

// Reset by user ID
await resetRateLimit('user-123');
```

### Check Without Incrementing

```typescript
import { checkRateLimit } from '@/lib/rate-limit/redis-store';

const result = await checkRateLimit('192.168.1.1', 100);
console.log('Remaining requests:', result.remaining);
console.log('Can proceed:', result.success);
```

## Testing

### Test Rate Limiting Locally

```bash
# Send multiple requests quickly
for i in {1..35}; do
  curl -X POST http://localhost:3060/api/hotels/search \
    -H "Content-Type: application/json" \
    -d '{"checkIn":"2025-12-01","checkOut":"2025-12-05","rooms":[{"adults":2}]}'
done

# After 30 requests, you should get:
# {"error":"Too many search requests. Please wait a moment.","retryAfter":60}
```

### Test with Different IPs (for testing)

```javascript
// Mock different IPs by setting x-forwarded-for header
await fetch('/api/hotels/search', {
  method: 'POST',
  headers: {
    'X-Forwarded-For': '192.168.1.100',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({...}),
});
```

## Best Practices

### 1. Choose Appropriate Limits
- **Authentication endpoints**: Strict limits (3-5 per 15-60 minutes)
- **Search endpoints**: Moderate limits (20-50 per minute)
- **Read-only APIs**: Lenient limits (100-1000 per minute)
- **Write operations**: Moderate limits (10-20 per 5 minutes)

### 2. Inform Users
Always include clear error messages and `Retry-After` headers so clients know when they can retry.

### 3. Whitelist Admin Users
```typescript
skip: (request) => {
  const userRole = request.headers.get('x-user-role');
  return userRole === 'admin';
}
```

### 4. Use Redis in Production
Switch from in-memory to Redis for:
- Multi-server deployments
- Persistent rate limiting across restarts
- Better scalability

### 5. Monitor and Adjust
- Log rate limit violations
- Monitor which endpoints are hit most
- Adjust limits based on actual usage patterns

## Security Considerations

### 1. IP Spoofing
The system trusts proxy headers (`x-forwarded-for`, etc.). In production:
- Only trust these headers if behind a reverse proxy
- Configure your load balancer/proxy to set these headers correctly
- Consider additional authentication-based rate limiting

### 2. Distributed Attacks
In-memory storage won't protect against distributed attacks across multiple servers. Use Redis in production.

### 3. Brute Force Protection
For authentication endpoints, combine rate limiting with:
- Account lockout after X failed attempts
- CAPTCHA after threshold
- Email notifications for suspicious activity

## Troubleshooting

### Rate Limits Not Working
1. Check if middleware is imported and called
2. Verify identifier extraction works (check logs)
3. Check if `skip` function is bypassing limits unintentionally

### Rate Limits Too Aggressive
1. Increase `limit` value
2. Increase `windowMs` duration
3. Use user-based identifiers instead of IP

### Performance Issues
1. Monitor `getRateLimitStats()` for cache size
2. Switch to Redis for better performance
3. Consider using edge functions (Vercel Edge, Cloudflare Workers)

## Future Enhancements

1. **Redis Integration**: Production-ready distributed rate limiting
2. **Dashboard**: Admin UI to view and manage rate limits
3. **Dynamic Limits**: Adjust limits based on user tier (free/premium)
4. **Burst Protection**: Allow short bursts but enforce overall limits
5. **Geographic Limits**: Different limits for different regions
6. **Cost-Based Limiting**: Limit based on computational cost, not just count
7. **Metrics**: Integration with monitoring tools (Datadog, New Relic)

## Additional Resources

- [IETF Rate Limiting Standard](https://datatracker.ietf.org/doc/html/draft-ietf-httpapi-ratelimit-headers)
- [OWASP Rate Limiting Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DDoS_Prevention_Cheat_Sheet.html)
- [Redis Rate Limiting Patterns](https://redis.io/topics/introduction)
