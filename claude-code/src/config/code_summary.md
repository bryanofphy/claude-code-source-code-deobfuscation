# Configuration Module

## Overview
The configuration module manages application settings from multiple sources including config files, environment variables, and CLI arguments. It provides a unified configuration interface with sensible defaults and validation.

## Purpose
- Load configuration from multiple sources (files, env, CLI)
- Provide default configuration values
- Merge and prioritize configuration sources
- Validate critical configuration settings
- Support cross-platform config file locations

## Key Files

### index.ts
Main configuration loading and merging logic.

**Configuration sources** (in priority order):
1. Default configuration (lowest priority)
2. Config files from standard locations
3. Environment variables
4. Command-line arguments (highest priority)

**Key functions**:
- **loadConfig()**: Loads and merges configuration from all sources
- **loadConfigFromFile()**: Loads JSON or JS config files
- **loadConfigFromEnv()**: Extracts config from environment variables
- **mergeConfigs()**: Deep merges configuration objects
- **validateConfig()**: Validates critical settings

**Config file search paths**:
- `.claude-code.json` in current directory
- `.claude-code.js` in current directory
- `~/.claude-code/config.json`
- `~/.claude-code.json`
- `~/.config/claude-code/config.json` (XDG on Linux/macOS)
- `%APPDATA%/claude-code/config.json` (Windows)

### schema.ts
Configuration schema and type definitions.

Defines the structure of configuration objects and validation rules.

### defaults.ts
Default configuration values.

Contains sensible defaults for all configuration options.

## Default Configuration

```javascript
{
  api: {
    baseUrl: 'https://api.anthropic.com',
    version: 'v1',
    timeout: 60000 // 60 seconds
  },

  ai: {
    model: 'claude-3-opus-20240229',
    temperature: 0.5,
    maxTokens: 4096,
    maxHistoryLength: 20
  },

  auth: {
    autoRefresh: true,
    tokenRefreshThreshold: 300, // 5 minutes
    maxRetryAttempts: 3
  },

  terminal: {
    theme: 'system',
    useColors: true,
    showProgressIndicators: true,
    codeHighlighting: true
  },

  telemetry: {
    enabled: true,
    submissionInterval: 1800000, // 30 minutes
    maxQueueSize: 100,
    autoSubmit: true
  },

  fileOps: {
    maxReadSizeBytes: 10485760 // 10MB
  },

  execution: {
    shell: process.env.SHELL || 'bash'
  },

  logger: {
    level: 'info',
    timestamps: true,
    colors: true
  },

  version: '0.2.29'
}
```

## Environment Variables

The module recognizes these environment variables:
- **CLAUDE_API_KEY**: API key for authentication
- **CLAUDE_API_URL**: Override API base URL
- **CLAUDE_LOG_LEVEL**: Set logging level (debug, info, warn, error)
- **CLAUDE_TELEMETRY**: Disable telemetry (set to '0' or 'false')
- **CLAUDE_MODEL**: Override AI model selection

## Dependencies
- **Internal**:
  - `utils/logger`: Logging functionality
  - `errors/formatter`: Error formatting
  - `errors/types`: Error categories
- **External**:
  - `fs`: File system operations
  - `path`: Path manipulation
  - `os`: OS information (homedir)

## Usage Example

```typescript
import { loadConfig } from './config/index.js';

// Load configuration with CLI options
const config = await loadConfig({
  verbose: true,          // Enable debug logging
  config: './my-config.json',  // Custom config file
  debug: true
});

// Access configuration
console.log('API URL:', config.api.baseUrl);
console.log('Model:', config.ai.model);
console.log('Log level:', config.logger.level);

// Configuration is automatically validated
// Logger is automatically configured
```

## Config File Examples

**JSON format** (`.claude-code.json`):
```json
{
  "ai": {
    "model": "claude-3-sonnet-20240229",
    "temperature": 0.7
  },
  "terminal": {
    "theme": "dark",
    "codeHighlighting": true
  },
  "telemetry": {
    "enabled": false
  }
}
```

**JavaScript format** (`.claude-code.js`):
```javascript
module.exports = {
  ai: {
    model: process.env.PREFERRED_MODEL || 'claude-3-opus-20240229',
    temperature: 0.5
  },
  auth: {
    autoRefresh: true
  }
};
```

## Priority Order

Configuration values are applied in this order (later sources override earlier):

1. **Defaults**: Built-in default values
2. **Config File**: First found config file from search paths
3. **Environment Variables**: `CLAUDE_*` environment variables
4. **CLI Options**: Command-line flags (highest priority)

Example:
- Default: `model: 'claude-3-opus-20240229'`
- Config file overrides: `model: 'claude-3-sonnet-20240229'`
- Environment overrides: `CLAUDE_MODEL=claude-3-haiku-20240307`
- CLI overrides: `--model claude-3-opus-20240229`

## Important Notes
- **Deep merging**: Nested objects are merged recursively, not replaced
- **Validation**: Critical settings (API URL, model) are validated on load
- **Cross-platform**: Config paths adapt to Windows, Linux, and macOS
- **Multiple formats**: Supports both JSON and JavaScript config files
- **Automatic logger config**: Logger is reconfigured after loading
- **First match wins**: Only the first found config file is used
- **Error handling**: Invalid config files log warnings but don't crash
- **Optional settings**: Most settings have sensible defaults
