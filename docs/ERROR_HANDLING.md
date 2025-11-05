# Error Handling & Logging System

## Overview
The centralized error handling and logging system provides consistent error responses, structured logging, and comprehensive error tracking throughout the application.

## Components

### 1. Custom Error Types (`lib/errors/types.ts`)

Pre-defined error classes for common scenarios:

| Error Class | Status Code | Use Case |
|------------|-------------|----------|
| `ValidationError` | 400 | Invalid input or request data |
| `AuthenticationError` | 401 | Unauthenticated requests |
| `AuthorizationError` | 403 | Unauthorized access attempts |
| `NotFoundError` | 404 | Resources that don't exist |
| `ConflictError` | 409 | Conflicting operations (e.g., duplicate email) |
| `RateLimitError` | 429 | Rate limit exceeded |
| `PaymentError` | 402 | Payment processing failures |
| `ExternalAPIError` | 502 | External service failures (KPLUS, payment gateways) |
| `DatabaseError` | 500 | Database-related errors |
| `InternalServerError` | 500 | Unexpected server errors |
| `ServiceUnavailableError` | 503 | Temporary service unavailability |

### 2. Error Handler (`lib/errors/handler.ts`)

Functions for handling and responding to errors:

- **`handleAPIError()`**: Converts errors to NextResponse with proper formatting
- **`withErrorHandler()`**: HOF wrapper for API routes with automatic error handling
- **`tryCatch()`**: Async operation wrapper that returns `[data, error]` tuple
- **`assert()`**: Assert condition or throw error
- **`safeJSONParse()`**: Safe JSON parsing with fallback
- **`getErrorMessage()`**: Extract error message from unknown types
- **`isOperationalError()`**: Check if error is expected (operational)

### 3. Logger (`lib/errors/logger.ts`)

Structured logging functions:

- **`logDebug()`**: Development-only debug messages
- **`logInfo()`**: Informational messages
- **`logWarn()`**: Warning messages
- **`logError()`**: Error messages with stack trace
- **`logRequest()`**: HTTP request logging
- **`logResponse()`**: HTTP response logging
- **`logExternalAPI()`**: External API call logging
- **`logQuery()`**: Database query logging (dev only)
- **`logSecurity()`**: Security event logging
- **`logPerformance()`**: Performance metric logging
- **`logUserAction()`**: User action logging for analytics
- **`createTimer()`**: Performance timer utility
- **`withTiming()`**: Async wrapper with execution time logging

## Usage Examples

### Basic Error Handling

```typescript
import { NextRequest, NextResponse } from 'next/server';
import {
  withErrorHandler,
  ValidationError,
  NotFoundError,
  logInfo,
} from '@/lib/errors';

export const POST = withErrorHandler(async (request: NextRequest) => {
  logInfo('Processing booking request');

  const body = await request.json();

  // Validate input
  if (!body.hotelId) {
    throw new ValidationError('Hotel ID is required');
  }

  // Check if resource exists
  const hotel = await getHotel(body.hotelId);
  if (!hotel) {
    throw new NotFoundError('Hotel', { hotelId: body.hotelId });
  }

  // Process booking
  return NextResponse.json({ success: true });
});
```

### Try-Catch Pattern

```typescript
import { tryCatch, DatabaseError } from '@/lib/errors';

const [user, error] = await tryCatch(
  supabase.from('profiles').select('*').eq('id', userId).single()
);

if (error) {
  throw new DatabaseError('Failed to fetch user', error);
}

// Use user data safely
console.log(user.email);
```

### Assert Pattern

```typescript
import { assert, AuthorizationError } from '@/lib/errors';

assert(
  user.role === 'admin',
  new AuthorizationError('Admin access required')
);

// Continue with admin-only logic
```

### Performance Monitoring

```typescript
import { withTiming, createTimer } from '@/lib/errors';

// Method 1: withTiming wrapper
const hotels = await withTiming(
  'KPLUS Hotel Search',
  async () => await searchHotels(params),
  { destination: params.destination }
);

// Method 2: Manual timer
const endTimer = createTimer('Database Query', { table: 'bookings' });
const bookings = await fetchBookings();
endTimer(); // Logs: "Performance: Database Query took 150ms"
```

### External API Error Handling

```typescript
import { ExternalAPIError, logExternalAPI } from '@/lib/errors';

try {
  const startTime = Date.now();
  const response = await kplusAPI.searchHotels(params);
  const duration = Date.now() - startTime;

  logExternalAPI('KPLUS', '/search/hotels', true, duration);

  return response;
} catch (error) {
  logExternalAPI('KPLUS', '/search/hotels', false);
  throw new ExternalAPIError(
    'KPLUS',
    'Hotel search failed',
    { originalError: error.message }
  );
}
```

### Security Logging

```typescript
import { logSecurity } from '@/lib/errors';

// Log security events
logSecurity('Failed Login Attempt', 'medium', {
  userId: userId,
  ip: request.ip,
  attempts: failedAttempts,
});

// Log critical security events
logSecurity('Unauthorized API Access', 'critical', {
  endpoint: request.url,
  ip: request.ip,
  userId: user?.id,
});
```

## Error Response Format

All errors are returned in a consistent format:

```json
{
  "error": {
    "message": "Hotel not found",
    "code": "NOT_FOUND",
    "statusCode": 404,
    "details": {
      "hotelId": "12345"
    },
    "timestamp": "2025-01-06T10:30:00.000Z",
    "requestId": "req_1704537000000_abc123"
  }
}
```

**Response Headers:**
```
X-Request-ID: req_1704537000000_abc123
```

## Log Format

All logs follow a structured format:

```
[2025-01-06T10:30:00.000Z] [INFO] Processing booking request | {"userId":"user123","hotelId":"hotel456"}
[2025-01-06T10:30:01.500Z] [WARN] Slow operation: Database Query took 1500ms | {"table":"bookings"}
[2025-01-06T10:30:02.000Z] [ERROR] External API failed | {"service":"KPLUS","endpoint":"/search","message":"Connection timeout"}
```

## Operational vs Programming Errors

### Operational Errors (Expected)
These are errors we expect to happen in normal operation:
- Invalid user input (ValidationError)
- Resource not found (NotFoundError)
- Authentication failure (AuthenticationError)
- Rate limit exceeded (RateLimitError)
- External API failure (ExternalAPIError)

**Handling:** Return appropriate HTTP status codes with helpful error messages.

### Programming Errors (Unexpected)
These indicate bugs in the code:
- Type errors
- Null pointer exceptions
- Unhandled promise rejections

**Handling:** Log with full stack trace, return generic 500 error to client (don't expose internals).

## Integration with External Services

In production, integrate with monitoring services:

### Sentry (Error Tracking)

```typescript
// lib/errors/logger.ts
import * as Sentry from '@sentry/nextjs';

export function logError(error: Error, context?: LogContext): void {
  console.error(formatLog(LogLevel.ERROR, error.message, errorContext));

  if (process.env.NODE_ENV === 'production') {
    Sentry.captureException(error, {
      contexts: { custom: context },
    });
  }
}
```

### Datadog (Logging & APM)

```typescript
// lib/errors/logger.ts
import { datadogLogs } from '@datadog/browser-logs';

export function logInfo(message: string, context?: LogContext): void {
  console.log(formatLog(LogLevel.INFO, message, context));

  if (process.env.NODE_ENV === 'production') {
    datadogLogs.logger.info(message, context);
  }
}
```

### LogRocket (Session Replay)

```typescript
// lib/errors/logger.ts
import LogRocket from 'logrocket';

export function logError(error: Error, context?: LogContext): void {
  console.error(formatLog(LogLevel.ERROR, error.message, errorContext));

  if (process.env.NODE_ENV === 'production') {
    LogRocket.captureException(error, {
      extra: context,
    });
  }
}
```

## Best Practices

### 1. Use Specific Error Types

```typescript
// ❌ Bad
throw new Error('User not found');

// ✅ Good
throw new NotFoundError('User', { userId });
```

### 2. Provide Context

```typescript
// ❌ Bad
throw new DatabaseError('Query failed');

// ✅ Good
throw new DatabaseError('Failed to fetch user bookings', {
  userId,
  table: 'bookings',
  query: 'SELECT * FROM bookings WHERE user_id = $1',
});
```

### 3. Don't Swallow Errors

```typescript
// ❌ Bad
try {
  await riskyOperation();
} catch (error) {
  // Silent failure
}

// ✅ Good
try {
  await riskyOperation();
} catch (error) {
  logError(error, { context: 'important-operation' });
  throw new InternalServerError('Operation failed', error);
}
```

### 4. Log at Appropriate Levels

```typescript
// Debug: Development information
logDebug('Cache hit for key', { key: 'user:123' });

// Info: Normal operations
logInfo('User logged in', { userId: '123' });

// Warn: Concerning but not breaking
logWarn('API response slow', { duration: 2000, endpoint: '/search' });

// Error: Something went wrong
logError(error, { operation: 'payment-processing' });
```

### 5. Use Error Handlers Consistently

```typescript
// Apply to all API routes
export const POST = withErrorHandler(async (request) => {
  // Your logic
});

export const GET = withErrorHandler(async (request) => {
  // Your logic
});
```

## Examples in Existing Code

### Email Verification Route
[app/api/auth/verify-token/route.ts:17-43](app/api/auth/verify-token/route.ts#L17-L43)

```typescript
export const POST = withErrorHandler(async (request: NextRequest) => {
  logInfo('Verify Token: Processing request');

  const body = await request.json();
  const { token } = body;

  if (!token) {
    throw new ValidationError('Verification token is required');
  }

  const result = await verifyEmailToken(token);

  if (!result.success) {
    logError(new Error(`Token verification failed: ${result.error}`));
    throw new ValidationError(result.error || 'Invalid or expired token');
  }

  logInfo(`Email verified successfully for user: ${result.userId}`);

  return NextResponse.json({
    success: true,
    message: 'Email verified successfully',
    userId: result.userId,
  });
});
```

## Testing

### Test Error Responses

```typescript
// Test validation error
const response = await fetch('/api/auth/verify-token', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({}), // Missing token
});

expect(response.status).toBe(400);
const data = await response.json();
expect(data.error.code).toBe('VALIDATION_ERROR');
expect(data.error.message).toBe('Verification token is required');
```

### Test Error Logging

```typescript
// Mock console.error to verify logging
const errorSpy = jest.spyOn(console, 'error');

logError(new Error('Test error'), { context: 'test' });

expect(errorSpy).toHaveBeenCalled();
expect(errorSpy.mock.calls[0][0]).toContain('[ERROR]');
expect(errorSpy.mock.calls[0][0]).toContain('Test error');
```

## Troubleshooting

### Errors Not Being Caught
1. Ensure `withErrorHandler` is used on route handler
2. Check that errors are being thrown, not returned
3. Verify error extends `Error` or `AppError`

### Logs Not Appearing
1. Check `NODE_ENV` for debug logs (only in development)
2. Verify console methods aren't being overridden
3. Check log level configuration

### Stack Traces Missing
1. Ensure `Error.captureStackTrace` is supported
2. Check that error is thrown from correct location
3. Verify source maps are enabled for production

## Future Enhancements

1. **Distributed Tracing**: OpenTelemetry integration
2. **Error Aggregation**: Group similar errors
3. **Alert System**: Automatic notifications for critical errors
4. **Error Analytics**: Track error trends and patterns
5. **User Context**: Include user information in error logs
6. **Request Context**: Include full request details
7. **Performance Profiling**: Detailed performance metrics
8. **Custom Error Pages**: Branded error pages for users

## Related Documentation

- [Rate Limiting](./RATE_LIMITING.md) - Uses error types for rate limit violations
- [Email Verification](./EMAIL_VERIFICATION.md) - Uses error handling for validation

## Support

For questions about error handling:
- Review existing error handler implementations
- Check console logs for detailed error context
- Contact: info@healmedy.travel
