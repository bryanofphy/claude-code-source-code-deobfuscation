# Utilities Module

## Overview
The utilities module provides a collection of commonly-used helper functions for async operations, logging, formatting, validation, and type utilities. These are foundational utilities used throughout the Claude Code application.

## Purpose
- Provide async operation helpers (timeout, retry, concurrency)
- Centralize logging functionality
- Offer text formatting and validation utilities
- Supply common type definitions
- Enable rate limiting and debouncing

## Key Files

### logger.ts
Comprehensive logging system (5.5KB).

**Logger class**: Singleton logger with multiple levels
- **Log levels**: DEBUG (0), INFO (1), WARN (2), ERROR (3), SILENT (4)
- **Methods**: debug(), info(), warn(), error()
- **Features**:
  - Configurable log level filtering
  - Timestamp inclusion
  - Color-coded output (cyan/green/yellow/red)
  - Verbose mode with context serialization
  - Custom output destinations
  - Environment-based configuration

**Configuration**:
- `level`: Minimum log level to display
- `timestamps`: Include timestamps (default: true)
- `colors`: Colorize output (default: true)
- `verbose`: Include context objects (default: false)
- `destination`: Custom output function

**Environment variables**:
- `NODE_ENV=development` or `DEBUG=true` → DEBUG level
- `VERBOSE=true` → Verbose mode
- `LOG_LEVEL=<number>` → Custom level

### async.ts
Async operation utilities (10KB).

**Core utilities**:
- **delay()**: Sleep for specified milliseconds
- **withTimeout()**: Add timeout to async functions
- **withRetry()**: Add retry logic with exponential backoff
- **withConcurrency()**: Run operations in parallel with limit
- **debounce()**: Debounce function calls
- **throttle()**: Throttle function calls
- **createDeferred()**: Create deferred promise
- **runSequentially()**: Run functions in sequence

**Retry features**:
- Exponential backoff with jitter (±10%)
- Configurable max retries (default: 3)
- Custom retry conditions (isRetryable)
- Retry callbacks (onRetry)
- Initial delay: 1000ms, max delay: 10000ms

**Concurrency control**:
- Parallel execution with worker pool
- Configurable concurrency limit (default: 5)
- Maintains result order
- Fail-fast on errors

### formatting.ts
Text formatting utilities (7KB).

Functions for:
- Text wrapping and truncation
- Code syntax highlighting
- Markdown rendering
- String padding and alignment
- Size formatting (bytes → KB/MB/GB)

### validation.ts
Input validation utilities (5.6KB).

Functions for:
- String validation (non-empty, length, patterns)
- Number validation (range, positive/negative)
- URL validation
- Email validation
- File path validation
- Object property validation

### types.ts
Common type definitions.

Shared TypeScript types and interfaces used across modules.

### index.ts
Module exports aggregator.

Re-exports all utilities for convenient importing.

## Dependencies
- **Internal**:
  - `errors/types`: Error level types
- **External**: None (pure utilities)

## Usage Examples

### Logging
```typescript
import { logger, LogLevel } from './utils/logger.js';

// Configure logger
logger.configure({
  level: LogLevel.DEBUG,
  timestamps: true,
  verbose: true
});

// Log messages
logger.debug('Debugging info', { userId: 123 });
logger.info('Operation started');
logger.warn('Potential issue detected');
logger.error('Operation failed', error);
```

### Async Operations
```typescript
import { withTimeout, withRetry, withConcurrency, delay } from './utils/async.js';

// Add timeout
const fetchWithTimeout = withTimeout(fetchData, 5000);
await fetchWithTimeout(url);

// Add retry with exponential backoff
const fetchWithRetry = withRetry(fetchData, {
  maxRetries: 3,
  initialDelayMs: 1000,
  maxDelayMs: 10000,
  backoff: true,
  onRetry: (error, attempt) => {
    console.log(`Retry attempt ${attempt}: ${error.message}`);
  }
});
await fetchWithRetry(url);

// Process items with concurrency limit
const results = await withConcurrency(
  urls,
  async (url) => await fetch(url),
  5 // max 5 concurrent requests
);

// Debounce function
const debouncedSave = debounce(saveToDatabase, 1000, {
  leading: false,
  trailing: true
});

// Throttle function
const throttledUpdate = throttle(updateUI, 100);

// Simple delay
await delay(1000); // wait 1 second
```

### Deferred Promises
```typescript
import { createDeferred } from './utils/async.js';

const { promise, resolve, reject } = createDeferred<string>();

// Later...
resolve('Success!');

// Or reject
reject(new Error('Failed'));

// Await the promise
const result = await promise;
```

### Retry with Custom Logic
```typescript
const apiCall = withRetry(fetchAPI, {
  maxRetries: 5,
  initialDelayMs: 500,
  maxDelayMs: 5000,
  // Only retry on network errors or rate limits
  isRetryable: (error) => {
    return error.name === 'NetworkError' ||
           error.message.includes('rate limit');
  }
});
```

## Important Notes
- **Logger singleton**: Single logger instance used throughout app
- **Retry jitter**: ±10% randomization prevents retry stampedes
- **Exponential backoff**: Delays grow exponentially (1s, 2s, 4s, 8s, 10s max)
- **Concurrency limit**: Prevents overwhelming resources with parallel operations
- **Debounce vs Throttle**: Debounce delays, throttle limits frequency
- **Timeout errors**: Named 'TimeoutError' for easy identification
- **Color codes**: ANSI escape codes for terminal colors
- **Fail-fast**: withConcurrency stops all workers on first error
- **Context serialization**: JSON.stringify with error handling
- **Environment awareness**: Auto-configures based on NODE_ENV
