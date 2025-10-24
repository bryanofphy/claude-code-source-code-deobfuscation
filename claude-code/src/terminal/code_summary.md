# Terminal Interface Module

## Overview
The terminal module provides a rich command-line interface for Claude Code, handling all user interaction including formatted output, user input, progress indicators, and terminal capabilities detection.

## Purpose
- Provide formatted terminal output with colors
- Handle user input and prompts
- Display progress indicators (spinners)
- Render tables and structured data
- Support clickable links in terminals
- Adapt to terminal capabilities
- Manage screen clearing and sizing

## Key Files

### index.ts
Main terminal interface implementation (9KB).

**initTerminal()**: Initializes terminal with configuration
- Detects terminal capabilities
- Configures theme and colors
- Sets up terminal sizing

**Terminal class**: Complete terminal interface
- **Display methods**: display(), emphasize(), info(), success(), warn(), error()
- **Interactive elements**: prompt(), spinner()
- **Formatting**: table(), link()
- **Screen management**: clear(), displayWelcome()
- **Terminal detection**: isInteractive, detectCapabilities()
- **Dynamic sizing**: Responds to terminal resize events

**Features**:
- Color support detection (via chalk)
- TTY detection for interactive vs non-interactive
- Spinner management with IDs
- Formatted output with word wrapping
- Code syntax highlighting
- Unicode icons (✓, ✗, ⚠, ℹ)

### formatting.ts
Output formatting utilities.

**formatOutput()**: Formats text for terminal display
- Word wrapping to terminal width
- Code block highlighting
- Markdown rendering
- Color application

**clearScreen()**: Clears terminal screen
**getTerminalSize()**: Gets current terminal dimensions

### prompt.ts
User input prompts.

**createPrompt()**: Creates interactive prompts
- Text input
- Confirm (yes/no)
- Select from choices
- Multi-select
- Password input

### types.ts
TypeScript type definitions.

**TerminalInterface**: Complete interface specification
**TerminalConfig**: Configuration options
**PromptOptions**: Prompt configuration
**SpinnerInstance**: Spinner control interface

## Terminal Configuration

```typescript
interface TerminalConfig {
  theme: 'light' | 'dark' | 'system';
  useColors: boolean;
  showProgressIndicators: boolean;
  codeHighlighting: boolean;
  maxHeight?: number;
  maxWidth?: number;
}
```

## Dependencies
- **Internal**:
  - `utils/logger`: Logging
- **External**:
  - `chalk`: Terminal colors
  - `inquirer`: Interactive prompts
  - `ora`: Spinners
  - `terminal-link`: Clickable links
  - `table`: Table rendering

## Usage Example

```typescript
import { initTerminal } from './terminal/index.js';

// Initialize terminal
const terminal = await initTerminal({
  terminal: {
    theme: 'dark',
    useColors: true,
    codeHighlighting: true
  }
});

// Display welcome message
terminal.displayWelcome();

// Display formatted messages
terminal.info('Starting analysis...');
terminal.success('Analysis complete!');
terminal.warn('Found 3 warnings');
terminal.error('Operation failed');

// Show progress
const spinner = terminal.spinner('Loading data...');
// ... do work ...
spinner.succeed('Data loaded successfully');

// Prompt for input
const answer = await terminal.prompt({
  type: 'confirm',
  name: 'proceed',
  message: 'Continue with operation?',
  default: true
});

// Display table
terminal.table([
  ['File', 'Size', 'Lines'],
  ['index.ts', '12KB', '450'],
  ['types.ts', '3KB', '120']
], { header: ['File', 'Size', 'Lines'], border: true });

// Create clickable link
const link = terminal.link('Documentation', 'https://docs.example.com');
console.log(link);

// Clear screen
terminal.clear();
```

## Spinner Management

```typescript
// Create spinner
const spinner = terminal.spinner('Processing...', 'task1');

// Update spinner text
spinner.update('Still processing...');

// Complete with success
spinner.succeed('Processing complete!');

// Or fail
spinner.fail('Processing failed');

// Or warn
spinner.warn('Processing completed with warnings');

// Or just stop
spinner.stop();
```

## Prompt Types

```typescript
// Text input
const { name } = await terminal.prompt({
  type: 'input',
  name: 'name',
  message: 'Enter your name:'
});

// Confirmation
const { confirmed } = await terminal.prompt({
  type: 'confirm',
  name: 'confirmed',
  message: 'Are you sure?',
  default: false
});

// Selection
const { choice } = await terminal.prompt({
  type: 'list',
  name: 'choice',
  message: 'Select an option:',
  choices: ['Option 1', 'Option 2', 'Option 3']
});

// Multi-select
const { selected } = await terminal.prompt({
  type: 'checkbox',
  name: 'selected',
  message: 'Select items:',
  choices: ['Item 1', 'Item 2', 'Item 3']
});

// Password
const { password } = await terminal.prompt({
  type: 'password',
  name: 'password',
  message: 'Enter password:'
});
```

## Important Notes
- **Color Fallback**: Automatically disables colors if terminal doesn't support them
- **Non-Interactive Mode**: Provides text-only fallbacks when not in TTY
- **Spinner Management**: Tracks active spinners by ID to prevent duplicates
- **Terminal Resize**: Automatically adapts to terminal size changes
- **Version Display**: Shows current version (0.2.29) in welcome message
- **Graceful Degradation**: Works in CI/CD environments without interactive features
- **Unicode Support**: Uses Unicode icons when available, falls back to text
- **Word Wrapping**: Automatically wraps text to terminal width
- **Code Highlighting**: Syntax highlighting for code blocks in output
