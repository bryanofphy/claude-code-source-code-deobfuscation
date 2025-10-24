# Codebase Analysis Module

## Overview
The codebase analysis module provides tools for analyzing code structure, understanding project organization, and gathering metrics about a codebase. It helps Claude Code understand the context of the working directory and provide better assistance.

## Purpose
- Analyze directory structures and file organization
- Extract project dependencies and relationships
- Search files by content patterns
- Provide background analysis capabilities
- Generate project structure summaries

## Key Files

### index.ts
Entry point and API for codebase analysis functionality.

**Main functions**:
- **analyzeProject()**: Analyzes a codebase directory and returns its structure
- **initCodebaseAnalysis()**: Initializes the analysis system with configuration

**Background analysis features**:
- **startBackgroundAnalysis()**: Runs periodic analysis (default: 5 minutes)
- **stopBackgroundAnalysis()**: Stops background analysis
- **getBackgroundAnalysisResults()**: Retrieves cached analysis results

Provides methods for:
- Analyzing current or specific directories
- Finding files by content patterns
- Analyzing project dependencies
- Background monitoring of code changes

### analyzer.ts
Core analysis logic and algorithms (16KB implementation).

**Key capabilities** (based on imports):
- **analyzeCodebase()**: Main analysis function
- **analyzeProjectDependencies()**: Dependency graph analysis
- **findFilesByContent()**: Content-based file search

**Type definitions**:
- **FileInfo**: Metadata about individual files
- **DependencyInfo**: Dependency relationship data
- **ProjectStructure**: Complete project analysis result

## Dependencies
- **Internal**:
  - File system utilities for traversing directories
  - Likely uses pattern matching for file search
- **External**:
  - Node.js `fs` and `path` modules

## Usage Example

```typescript
import { initCodebaseAnalysis } from './codebase/index.js';

// Initialize codebase analysis
const codebaseAnalyzer = initCodebaseAnalysis({
  codebase: {
    ignorePatterns: ['node_modules', '.git', 'dist'],
    maxFiles: 1000,
    maxSizePerFile: 1048576 // 1MB
  }
});

// Analyze current directory
const structure = await codebaseAnalyzer.analyzeCurrentDirectory();
console.log(`Found ${structure.fileCount} files`);

// Find files containing specific pattern
const results = await codebaseAnalyzer.findFiles('TODO:', process.cwd());
console.log(`Found ${results.length} files with TODOs`);

// Analyze dependencies
const dependencies = await codebaseAnalyzer.analyzeDependencies();
console.log('Project dependencies:', dependencies);

// Start background analysis
codebaseAnalyzer.startBackgroundAnalysis(5 * 60 * 1000); // Every 5 minutes

// Later, get cached results
const cachedResults = codebaseAnalyzer.getBackgroundAnalysisResults();
```

## Important Notes
- **Background analysis**: Runs periodically to keep project structure up-to-date
- **Configurable limits**: maxFiles and maxSizePerFile prevent analysis of large files/directories
- **Ignore patterns**: Supports excluding directories like node_modules, .git, dist
- **Performance**: Background analysis interval is configurable (default 5 minutes)
- **Caching**: Background results are cached to avoid redundant analysis
