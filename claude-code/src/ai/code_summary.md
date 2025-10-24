# AI Module

## Overview
The AI module provides the core interface for interacting with Anthropic's Claude API. It handles all AI-related functionality including chat completions, streaming responses, and prompt management for the Claude Code CLI tool.

## Purpose
- Manage connections to Claude API
- Handle authentication and API requests
- Provide both synchronous and streaming completion capabilities
- Supply pre-built prompt templates for common coding tasks

## Key Files

### client.ts
The main AI client implementation that communicates with the Claude API.
- **AIClient class**: Manages API requests with retry logic, timeout handling, and error recovery
- **Completion methods**: `complete()` for standard requests, `completeStream()` for streaming
- **Connection management**: Tests connectivity and handles authentication
- **Error handling**: Comprehensive error mapping for different API status codes (401, 429, 500, etc.)

Key features:
- Default configuration for Claude models (opus-20240229)
- Automatic retry with exponential backoff
- 60-second default timeout
- Support for streaming via Server-Sent Events (SSE)

### index.ts
Module initialization and singleton management.
- **initAI()**: Initializes the AI client with authentication
- **getAIClient()**: Returns the singleton AI client instance
- **isAIInitialized()**: Checks if the module is ready

Manages the lifecycle of the AI client and ensures proper authentication before use.

### prompts.ts
Prompt templates and formatting utilities for common coding tasks.
- **System prompts**: Pre-defined personas for code assistance, generation, review, and explanation
- **Prompt templates**: Structured templates for tasks like debugging, refactoring, documenting, and testing
- **Formatting utilities**: Functions to replace placeholders and build conversation messages
- **Language detection**: Automatically identifies code language from file extensions

Templates include:
- `explainCode`: Explains code functionality
- `refactorCode`: Suggests improvements
- `debugCode`: Assists with debugging
- `reviewCode`: Provides code review feedback
- `generateCode`: Creates new code from requirements
- `documentCode`: Adds documentation
- `testCode`: Generates test cases

### types.ts
TypeScript type definitions for AI functionality.
- **Message types**: Message, MessageRole for conversation structure
- **Model types**: AIModel interface for model metadata
- **Completion types**: CompletionOptions, CompletionRequest, CompletionResponse
- **Usage tracking**: AIUsage for token consumption metrics
- **Client interface**: AIClientInterface defining the contract for AI clients

## Dependencies
- **Internal**:
  - `utils/logger`: Logging functionality
  - `utils/async`: Timeout and retry utilities
  - `utils/formatting`: Text truncation
  - `auth/manager`: Authentication management
  - `errors/formatter`: User-friendly error messages
- **External**: Node.js fetch API for HTTP requests

## Usage Example

```typescript
import { initAI, getAIClient } from './ai/index.js';

// Initialize the AI module
await initAI();

// Get the client
const client = getAIClient();

// Make a completion request
const response = await client.complete('Explain async/await in JavaScript', {
  maxTokens: 500,
  temperature: 0.7
});

console.log(response.content[0].text);

// Use streaming
await client.completeStream(
  'Write a sorting function',
  { maxTokens: 1000 },
  (event) => {
    if (event.type === 'content_block_delta') {
      process.stdout.write(event.delta.text);
    }
  }
);
```

## Important Notes
- The module uses a singleton pattern to maintain a single AI client instance
- Authentication must be configured before initialization
- All API requests include retry logic with exponential backoff (max 3 retries)
- Default model is Claude 3 Opus (configurable via options)
- Supports both blocking and streaming response modes
- Comprehensive error handling with user-friendly error messages
