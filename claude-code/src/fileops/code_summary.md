# File Operations Module

## Overview
The file operations module provides high-level file system operations for Claude Code, including reading, writing, searching, and manipulating files with safety checks and error handling.

## Purpose
- Read and write files safely
- Search file contents
- Manipulate file paths
- Check file permissions
- Handle file metadata
- Implement size limits

## Key Files

### index.ts
File operation implementations (14.5KB).

Main features (based on file size):
- File reading with size limits
- Safe file writing
- File search and filtering
- Path manipulation utilities
- Permission checking
- Metadata extraction
- Directory traversal

Expected functionality:
- Read file contents (text/binary)
- Write files with validation
- Search files by name/content
- Check file existence
- Get file size and modification time
- List directory contents
- Create directories

## Dependencies
- **Internal**:
  - `utils/logger`: Logging
  - `errors/formatter`: Error handling
  - `config`: File size limits (maxReadSizeBytes: 10MB)
- **External**:
  - Node.js `fs/promises`: Async file operations
  - Node.js `path`: Path manipulation

## Usage Example

```typescript
import { initFileOps } from './fileops/index.js';

// Initialize file operations
const fileOps = initFileOps(config);

// Read a file
const content = await fileOps.readFile('/path/to/file.txt');

// Write a file
await fileOps.writeFile('/path/to/output.txt', 'content');

// Search files
const matches = await fileOps.searchFiles({
  directory: '/path/to/search',
  pattern: '*.ts',
  content: 'TODO'
});

// Check if file exists
const exists = await fileOps.fileExists('/path/to/file');

// Get file metadata
const metadata = await fileOps.getMetadata('/path/to/file');
console.log(metadata.size, metadata.modified);

// List directory
const files = await fileOps.listDirectory('/path/to/dir');
```

## Important Notes
- **Size limits**: maxReadSizeBytes prevents reading huge files (default: 10MB)
- **Safety checks**: Validates paths and permissions
- **Error handling**: User-friendly error messages for file operations
- **Async operations**: All operations are Promise-based
- **Path validation**: Prevents directory traversal attacks
