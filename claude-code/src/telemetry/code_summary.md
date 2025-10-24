# Telemetry Module

## Overview
The telemetry module collects usage analytics and metrics for Claude Code. It tracks feature usage, errors, and performance data to help improve the product while respecting user privacy.

## Purpose
- Collect usage metrics
- Track errors and exceptions
- Monitor performance
- Queue events for submission
- Respect user privacy preferences
- Support opt-out mechanisms

## Key Files

### index.ts
Telemetry collection and submission (14.6KB).

Main features (based on file size):
- Event collection and queuing
- Batch submission to analytics service
- Privacy-aware data sanitization
- Opt-out support
- Performance metrics tracking
- Error tracking
- Feature usage analytics

Expected functionality:
- Track command usage
- Monitor AI interactions
- Record error occurrences
- Measure operation durations
- Queue events for batch submission
- Periodic auto-submission
- Manual submission control

## Configuration

From config module:
```javascript
telemetry: {
  enabled: true,                    // User can disable
  submissionInterval: 1800000,      // 30 minutes
  maxQueueSize: 100,                // Max events before forced submission
  autoSubmit: true                  // Auto-submit at intervals
}
```

## Dependencies
- **Internal**:
  - `utils/logger`: Logging
  - `config`: Telemetry settings
  - `errors`: Error tracking
- **External**:
  - HTTP client for submission

## Usage Example

```typescript
import { initTelemetry } from './telemetry/index.js';

// Initialize telemetry
const telemetry = initTelemetry(config);

// Track command usage
telemetry.trackCommand('explain', {
  duration: 1250,
  success: true,
  fileType: 'typescript'
});

// Track error
telemetry.trackError(error, {
  category: 'authentication',
  severity: 'major'
});

// Track performance metric
telemetry.trackPerformance('ai_response', {
  duration: 3500,
  tokens: 450
});

// Track feature usage
telemetry.trackFeature('code_generation', {
  language: 'python',
  linesGenerated: 45
});

// Manual submission
await telemetry.submit();

// Get queue status
const status = telemetry.getStatus();
console.log(`${status.queuedEvents} events pending`);
```

## Environment Variable

- **CLAUDE_TELEMETRY**: Set to '0' or 'false' to disable

## Important Notes
- **Privacy-first**: All telemetry respects user privacy
- **Opt-out supported**: Users can disable via config or environment variable
- **No PII**: Personal Identifiable Information is sanitized
- **Batched submission**: Events queued and sent in batches
- **Auto-submission**: Configurable interval (default: 30 minutes)
- **Queue limits**: Forced submission at maxQueueSize (default: 100 events)
- **Anonymous**: No user identification in telemetry data
- **Performance tracking**: Helps identify slow operations
- **Error insights**: Helps prioritize bug fixes
