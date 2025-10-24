# Authentication Module

## Overview
The authentication module provides comprehensive authentication functionality for Claude Code CLI, supporting both API key and OAuth authentication methods. It manages the complete authentication lifecycle including token storage, validation, refresh, and state management.

## Purpose
- Provide multiple authentication methods (API key and OAuth)
- Manage authentication tokens securely
- Handle automatic token refresh for OAuth
- Maintain authentication state throughout application lifecycle
- Offer event-based authentication updates

## Key Files

### index.ts
Main entry point and exported API for the authentication system.
- **initAuthentication()**: Initializes the authentication system with configuration
- **authManager**: Singleton instance of AuthManager
- Exports a unified interface for authentication operations including `authenticate()`, `logout()`, `getToken()`, `isAuthenticated()`
- Integrates with environment variables for API key authentication (`ANTHROPIC_API_KEY`)

Key features:
- Provides a functional API layer over the AuthManager class
- Handles method selection (API key vs OAuth) based on config or environment
- Emits authentication events for monitoring

### manager.ts
Core authentication manager implementation with two versions (index.ts includes v1, manager.ts is v2).

**AuthManager class** provides:
- **State management**: Tracks authentication state (INITIAL, AUTHENTICATING, AUTHENTICATED, FAILED, REFRESHING)
- **Token lifecycle**: Initialization, storage, validation, refresh, and cleanup
- **Auto-refresh**: Automatic token refresh before expiration (configurable threshold)
- **Event emitter**: Emits events for state changes, login, logout, token refresh, and errors
- **Multiple auth methods**: Supports both API key and OAuth flows

Key features:
- Singleton pattern for global authentication state
- Automatic token refresh scheduling (80% of token lifetime)
- 5-minute default refresh threshold
- Maximum 3 retry attempts for operations
- Token validation on initialization

### oauth.ts
Complete OAuth 2.0 authentication flow implementation.

**performOAuthFlow()**: Main OAuth flow orchestration
- Generates PKCE parameters for enhanced security
- Opens browser for user authorization
- Starts local server to receive OAuth callback
- Validates state parameter to prevent CSRF attacks
- Exchanges authorization code for access token

**refreshOAuthToken()**: Refreshes expired OAuth tokens
- Uses refresh token to obtain new access token
- Handles token endpoint communication
- Falls back to existing refresh token if new one not provided

Default OAuth configuration:
- Client ID: `claude-code-cli`
- Authorization endpoint: `https://auth.anthropic.com/oauth2/auth`
- Token endpoint: `https://auth.anthropic.com/oauth2/token`
- Redirect URI: `http://localhost:3000/callback`
- Scopes: `['anthropic.claude']`
- PKCE enabled by default

### tokens.ts
Token storage, validation, and utility functions.

**createTokenStorage()**: Factory for token storage provider
- In-memory storage implementation (development)
- Provides async interface for save/get/delete/clear operations
- Designed for easy replacement with secure storage (OS keychain, encrypted files)

**Token utilities**:
- `isTokenExpired()`: Checks if token is expired with optional threshold
- `validateToken()`: Validates token structure and expiration
- `formatTokenExpiration()`: Human-readable expiration timestamps
- `getTokenDetails()`: Safe display of token information (masks sensitive data)
- `createAuthorizationHeader()`: Formats Bearer token headers
- `extractTokenFromHeader()`: Parses authorization headers

### types.ts
TypeScript type definitions for authentication.

**Core types**:
- **AuthToken**: Complete token structure (access, refresh, expiration, type, scope)
- **AuthMethod**: Enum for API_KEY and OAUTH methods
- **AuthState**: Enum for authentication states (7 states)
- **AuthResult**: Result object for authentication operations
- **TokenStorage**: Interface for storage providers
- **OAuthConfig**: Configuration for OAuth flows
- **AuthConfig**: Configuration for AuthManager

## Dependencies
- **Internal**:
  - `utils/logger`: Logging
  - `utils/async`: Async utilities (deferred promises, timeouts)
  - `errors/formatter`: User-friendly error messages
  - `errors/types`: Error categories
- **External**:
  - `events`: EventEmitter for state change notifications
  - `open`: Opening browser for OAuth flow
  - Node.js `fetch`: HTTP requests for OAuth

## Usage Example

```typescript
import { initAuthentication, authManager } from './auth/index.js';

// Initialize authentication system
const auth = await initAuthentication({
  preferredMethod: AuthMethod.API_KEY,
  autoRefresh: true,
  tokenRefreshThreshold: 300 // 5 minutes
});

// Check if already authenticated
if (auth.isAuthenticated()) {
  console.log('Already logged in');
} else {
  // Authenticate with API key (from environment)
  const result = await auth.authenticate({ method: AuthMethod.API_KEY });

  if (result.success) {
    console.log('Authenticated successfully');
  }
}

// Get authorization header for API calls
const authHeader = auth.getAuthorizationHeader();

// Listen to auth events
authManager.on('auth:state_changed', (state) => {
  console.log('Auth state:', state);
});

authManager.on('auth:token_refreshed', () => {
  console.log('Token refreshed automatically');
});

// Logout
await auth.logout();
```

## OAuth Flow Example

```typescript
// OAuth authentication
const result = await auth.authenticate({ method: AuthMethod.OAUTH });

// This will:
// 1. Generate PKCE code challenge
// 2. Open browser to authorization URL
// 3. Start local server on localhost:3000
// 4. Wait for user to authorize
// 5. Receive callback with authorization code
// 6. Exchange code for access token
// 7. Store token securely
// 8. Schedule automatic refresh
```

## Important Notes
- **API Key Authentication**: Never expires (expiresAt set to MAX_SAFE_INTEGER)
- **OAuth Tokens**: Auto-refresh enabled by default, refreshes at 80% of token lifetime
- **Token Storage**: Currently in-memory (development), should be replaced with secure storage for production
- **PKCE Security**: OAuth flow uses PKCE for enhanced security (note: current implementation has placeholder SHA256)
- **State Management**: Uses state machine pattern with event emissions for all transitions
- **Error Handling**: All operations include comprehensive error handling with user-friendly messages
- **Thread Safety**: Singleton pattern ensures single authentication state across application
- **Refresh Scheduling**: Uses `unref()` on timers to prevent blocking Node.js exit
