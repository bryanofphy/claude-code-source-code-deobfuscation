# Filesystem Module

## Overview
The filesystem module provides low-level filesystem operations and abstractions for Claude Code. It offers a more direct interface to file system operations compared to the higher-level fileops module.

## Purpose
- Provide low-level file system access
- Implement file system abstractions
- Handle file system events
- Support file watching
- Manage file system state

## Key Files

### operations.ts
Core filesystem operations (16.3KB).

Main features (based on file size):
- Raw file system operations
- File descriptors management
- Streaming file I/O
- File watching and monitoring
- Low-level path operations
- File system events
- Atomic operations

Expected functionality:
- Open/close file descriptors
- Stream file reading/writing
- Watch files for changes
- Handle file locks
- Manage temporary files
- Atomic file operations

## Dependencies
- **Internal**:
  - `utils/logger`: Logging
  - `errors/formatter`: Error handling
- **External**:
  - Node.js `fs`: Core filesystem module
  - Node.js `fs/promises`: Promise-based operations

## Usage Example

```typescript
import { createFileWatcher, atomicWrite } from './fs/operations.js';

// Watch a file for changes
const watcher = createFileWatcher('/path/to/file', {
  onChange: (event, filename) => {
    console.log(`File ${filename} changed: ${event}`);
  }
});

// Atomic file write (prevents corruption)
await atomicWrite('/path/to/file', 'content', {
  encoding: 'utf8',
  mode: 0o644
});

// Stream large file
const stream = createReadStream('/path/to/large/file');
stream.on('data', (chunk) => {
  // Process chunk
});
```

## Important Notes
- **Low-level access**: More direct control than fileops module
- **File watching**: Real-time file system monitoring
- **Atomic operations**: Prevents partial writes and corruption
- **Streaming**: Efficient handling of large files
- **Descriptors**: Explicit file descriptor management
