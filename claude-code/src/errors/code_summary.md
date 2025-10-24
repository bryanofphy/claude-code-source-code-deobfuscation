# Error Handling Module

## Overview
The error handling module provides centralized error management for Claude Code, including error formatting, tracking, reporting, and classification. It ensures consistent error handling across the application with user-friendly error messages.

## Purpose
- Centralize error handling logic
- Classify errors by category and severity
- Format errors for user-friendly display
- Track error frequency and patterns
- Report critical errors to monitoring systems
- Handle uncaught exceptions and unhandled rejections

## Key Files

### index.ts
Main error handling implementation and initialization.

**initErrorHandling()**: Initializes the error handling system
- Sets up console error handling
- Configures Sentry reporting (optional)
- Returns ErrorManager instance

**ErrorHandlerImpl class**: Core error manager
- **handleFatalError()**: Handles critical errors and exits process
- **handleUnhandledRejection()**: Handles promise rejections
- **handleUncaughtException()**: Handles uncaught exceptions
- **handleError()**: General error handler with classification
- Error rate limiting (max 100 unique errors tracked)
- Automatic reporting of critical/major errors

### types.ts
Comprehensive error type definitions.

**ErrorLevel enum** (9 levels):
- CRITICAL (0): Application-stopping errors
- MAJOR (1): Significant functionality impact
- MINOR (2): Minor functionality impact
- INFORMATIONAL (3): Non-impactful errors
- DEBUG, INFO, WARNING, ERROR, FATAL

**ErrorCategory enum** (24 categories):
- APPLICATION, AUTHENTICATION, NETWORK, FILE_SYSTEM
- COMMAND_EXECUTION, AI_SERVICE, CONFIGURATION, RESOURCE
- VALIDATION, INITIALIZATION, SERVER, API, TIMEOUT
- RATE_LIMIT, CONNECTION, AUTHORIZATION
- FILE_NOT_FOUND, FILE_ACCESS, FILE_READ, FILE_WRITE
- COMMAND, COMMAND_NOT_FOUND, and more

**UserError class**: Custom error with rich context
- Extends Error with additional properties
- Includes category, level, resolution hints
- Supports cause chaining
- Contains additional details object
- Optional error code

### formatter.ts
Error formatting for user display.

**formatErrorForDisplay()**: Formats errors for terminal output
- Extracts meaningful messages from various error types
- Adds resolution hints
- Formats stack traces
- Includes context information

### console.ts
Console error handler setup.

**setupConsoleErrorHandling()**: Configures console output
- Captures console.error calls
- Integrates with error manager
- Maintains error context

### sentry.ts
Sentry error reporting integration (optional).

**setupSentryReporting()**: Configures Sentry SDK
- Reports errors to Sentry monitoring
- Includes context and tags
- Filters sensitive information

## Error Options Interface

```typescript
interface ErrorOptions {
  level?: ErrorLevel;
  category?: ErrorCategory;
  context?: Record<string, any>;
  report?: boolean;
  userMessage?: string;
  resolution?: string | string[];
}
```

## Dependencies
- **Internal**:
  - `utils/logger`: Logging
- **External**:
  - Optional Sentry SDK for reporting

## Usage Example

```typescript
import { initErrorHandling, UserError, ErrorCategory, ErrorLevel } from './errors/index.js';

// Initialize error handling
const errorManager = initErrorHandling();

// Handle a general error
try {
  // Some operation
  throw new Error('Something went wrong');
} catch (error) {
  errorManager.handleError(error, {
    category: ErrorCategory.APPLICATION,
    level: ErrorLevel.MINOR,
    context: { operation: 'data processing' }
  });
}

// Create a user-friendly error
throw new UserError('Failed to authenticate', {
  category: ErrorCategory.AUTHENTICATION,
  level: ErrorLevel.MAJOR,
  resolution: [
    'Check your API key is correct',
    'Verify your internet connection',
    'Try logging in again with --login flag'
  ],
  cause: originalError,
  details: {
    attemptedMethod: 'api_key',
    timestamp: Date.now()
  }
});

// Handle fatal error
errorManager.handleFatalError(new Error('Critical system failure'));
// (This will log the error and exit the process)
```

## Error Flow

1. **Error Occurs**: Exception thrown anywhere in application
2. **Classification**: Assigned category and level
3. **Formatting**: Converted to user-friendly message
4. **Logging**: Logged at appropriate level (error/warn/info)
5. **Tracking**: Error count incremented
6. **Reporting**: Critical/major errors reported to monitoring
7. **Display**: Formatted error shown to user

## Important Notes
- **Error Rate Limiting**: Tracks up to 100 unique error patterns
- **Severity-based Handling**: Different logging levels per severity
- **Fatal Errors**: Exit process with code 1
- **User Errors**: Include resolution hints for better UX
- **Cause Chaining**: Preserves original error context
- **Stack Traces**: Captured for debugging
- **Monitoring Integration**: Automatic reporting of critical errors
- **Console Integration**: Captures all console.error calls
