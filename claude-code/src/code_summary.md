# Claude Code Source Directory

## Overview
This is the main source directory for Claude Code, an AI-powered CLI tool that helps developers with coding tasks using Anthropic's Claude API. The codebase is written in TypeScript and organized into modular subsystems.

## Architecture

Claude Code follows a modular architecture with clear separation of concerns:

```
src/
├── ai/            # AI integration and Claude API client
├── auth/          # Authentication and token management
├── codebase/      # Code analysis and project understanding
├── commands/      # CLI command framework and implementations
├── config/        # Configuration management
├── errors/        # Error handling and formatting
├── execution/     # Shell command execution
├── fileops/       # High-level file operations
├── fs/            # Low-level filesystem operations
├── telemetry/     # Usage analytics and metrics
├── terminal/      # Terminal UI and interaction
├── utils/         # Shared utility functions
├── cli.ts         # CLI entry point
└── index.ts       # Main application entry point
```

## Core Modules

### AI Module (`ai/`)
**Purpose**: Integration with Claude API
- Claude API client with retry and timeout logic
- Streaming and non-streaming completions
- Prompt templates for common coding tasks
- Model configuration (default: Claude 3 Opus)

**Key Features**:
- Automatic retry with exponential backoff
- 60-second default timeout
- Support for multiple Claude models
- Pre-built prompts for code explanation, generation, review, etc.

### Authentication Module (`auth/`)
**Purpose**: User authentication and token management
- API key authentication
- OAuth 2.0 flow with PKCE
- Token storage and refresh
- Automatic token renewal

**Key Features**:
- Singleton AuthManager with state machine
- Automatic token refresh before expiration
- Support for multiple auth methods
- Event-based authentication updates

### Commands Module (`commands/`)
**Purpose**: CLI command system
- Command registration and parsing
- Argument validation (positional, flags, types)
- Auto-generated help documentation
- Interactive REPL mode

**Key Features**:
- 49KB of command implementations
- Category-based organization
- Command aliases support
- Rich argument parsing (string, number, boolean, array)

### Configuration Module (`config/`)
**Purpose**: Application settings management
- Multi-source configuration (files, env, CLI)
- Default values for all settings
- Cross-platform config file locations
- Configuration validation

**Key Features**:
- Loads from 6+ config file locations
- Environment variable overrides
- Deep merge strategy
- Version: 0.2.29

### Error Handling Module (`errors/`)
**Purpose**: Centralized error management
- 24 error categories
- 9 severity levels
- User-friendly error messages
- Error tracking and reporting

**Key Features**:
- UserError class with resolution hints
- Automatic Sentry reporting (optional)
- Error rate limiting
- Console error capture

### Terminal Module (`terminal/`)
**Purpose**: Command-line interface
- Formatted output with colors
- Interactive prompts
- Progress indicators (spinners)
- Table rendering

**Key Features**:
- Color support detection
- TTY/non-TTY adaptation
- Unicode icons (✓, ✗, ⚠, ℹ)
- Clickable terminal links

### Utilities Module (`utils/`)
**Purpose**: Common helper functions
- Async operations (timeout, retry, concurrency)
- Logging with multiple levels
- Text formatting and validation
- Type definitions

**Key Features**:
- Exponential backoff with jitter
- Debounce and throttle
- Singleton logger
- Comprehensive validation utilities

## Supporting Modules

### Codebase Module (`codebase/`)
Analyzes project structure, dependencies, and file organization. Supports background analysis for keeping context up-to-date.

### Execution Module (`execution/`)
Executes shell commands safely with timeout, output capture, and error handling.

### File Operations Module (`fileops/`)
High-level file operations with safety checks, size limits (10MB), and validation.

### Filesystem Module (`fs/`)
Low-level filesystem access, file watching, atomic operations, and streaming I/O.

### Telemetry Module (`telemetry/`)
Collects usage analytics and error metrics (opt-out supported, privacy-first approach).

## Entry Points

### cli.ts (5KB)
CLI entry point that:
- Parses command-line arguments
- Initializes all subsystems
- Routes to appropriate commands
- Handles top-level errors

### index.ts (4.7KB)
Main application entry that:
- Coordinates module initialization
- Manages application lifecycle
- Exports public API
- Handles graceful shutdown

## Module Dependencies

```
Flow of dependencies:

utils (foundation)
  ↓
errors, config, logger
  ↓
auth, terminal, fs
  ↓
ai, fileops, codebase, telemetry, execution
  ↓
commands (orchestrates everything)
  ↓
cli, index (entry points)
```

## Key Technologies

- **Language**: TypeScript
- **AI**: Anthropic Claude API
- **Terminal**: chalk, inquirer, ora, terminal-link
- **HTTP**: Node.js fetch API
- **Authentication**: OAuth 2.0 with PKCE
- **File System**: Node.js fs/promises
- **Process**: Node.js child_process

## Design Patterns

- **Singleton**: Logger, AuthManager, CommandRegistry
- **Factory**: Config loading, token storage, prompts
- **Strategy**: Multiple auth methods, output destinations
- **Observer**: Event emitters for auth, errors
- **Decorator**: withTimeout, withRetry wrappers
- **Command**: Command pattern for CLI operations

## Configuration Sources (Priority Order)

1. **Defaults**: Built-in sensible defaults
2. **Config Files**: First found from search paths
3. **Environment Variables**: `CLAUDE_*` variables
4. **CLI Arguments**: Highest priority overrides

## Error Handling Strategy

1. **Categorization**: 24 error categories for classification
2. **Severity**: 9 levels from DEBUG to FATAL
3. **User-Friendly**: UserError class with resolution hints
4. **Tracking**: Rate limiting and pattern detection
5. **Reporting**: Optional Sentry integration

## Module Communication

- **Dependency Injection**: Commands receive all dependencies
- **Event Emitters**: Auth state changes, terminal events
- **Shared State**: Singleton managers and registries
- **Async Operations**: Promise-based throughout
- **Error Propagation**: Centralized error handling

## Key Files by Size

1. **commands/register.ts**: 49KB (command implementations)
2. **codebase/analyzer.ts**: 16.6KB (code analysis)
3. **fs/operations.ts**: 16.3KB (filesystem operations)
4. **commands/index.ts**: 16.3KB (command framework)
5. **telemetry/index.ts**: 14.6KB (analytics)
6. **fileops/index.ts**: 14.5KB (file operations)
7. **ai/client.ts**: 12.7KB (AI client)
8. **auth/index.ts**: 12.7KB (auth manager)
9. **execution/index.ts**: 11.5KB (command execution)
10. **utils/async.ts**: 10KB (async utilities)

## Total Codebase Stats

- **Total Files**: 37 TypeScript files
- **Modules**: 12 main modules
- **Lines of Code**: ~10,000+ (estimated)
- **Version**: 0.2.29

## Development Workflow

1. **Configuration**: Load from multiple sources
2. **Authentication**: Initialize auth manager
3. **Subsystems**: Initialize AI, terminal, codebase, etc.
4. **Command Processing**: Parse and execute commands
5. **Error Handling**: Catch and format all errors
6. **Telemetry**: Track usage (if enabled)
7. **Cleanup**: Graceful shutdown

## Important Notes

- **Modular Design**: Each module is self-contained with clear interfaces
- **TypeScript**: Full type safety throughout
- **Async/Await**: Modern async patterns everywhere
- **Error First**: Comprehensive error handling at every layer
- **Configurable**: Nearly everything can be configured
- **Privacy**: Telemetry is opt-out with no PII
- **Security**: OAuth with PKCE, secure token storage
- **Performance**: Retry logic, timeouts, concurrency limits
- **User Experience**: Colored output, progress indicators, helpful errors
- **Extensibility**: Easy to add new commands, auth methods, config sources

This architecture enables Claude Code to be a robust, maintainable, and user-friendly AI coding assistant.
