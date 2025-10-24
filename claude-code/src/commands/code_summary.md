# Commands Module

## Overview
The commands module provides a comprehensive CLI command framework for Claude Code. It handles command registration, argument parsing, validation, help generation, and execution with support for both one-off commands and interactive REPL mode.

## Purpose
- Define and register CLI commands
- Parse and validate command arguments
- Generate auto-formatted help documentation
- Execute commands with dependency injection
- Provide interactive command loop (REPL)
- Support command aliases and categories

## Key Files

### index.ts
Core command system implementation (16KB).

**CommandRegistry class**: Central command management
- Registers commands with validation
- Maps aliases to command names
- Categorizes commands for organization
- Lists and filters commands

**Main functions**:
- **parseArgs()**: Parses command-line arguments with type conversion
- **executeCommand()**: Executes a registered command by name
- **generateCommandHelp()**: Auto-generates help text from command definitions
- **initCommandProcessor()**: Initializes the command system with dependencies

**Features**:
- Positional and flag-based arguments
- Type validation (string, number, boolean, array)
- Required/optional arguments with defaults
- Argument choices for enum-like validation
- Short flags (e.g., `-v` for `--verbose`)
- Hidden commands for internal use

**Interactive mode**:
- REPL command loop with prompt
- Special exit commands (exit, quit, q, .exit)
- Error handling per command
- Command existence checking

### register.ts
Command registration and implementation (49KB).

Contains the actual command implementations including:
- Core commands (help, version, login, logout)
- AI interaction commands (ask, chat, explain)
- Code manipulation commands (refactor, review, generate)
- File operations (read, write, edit)
- Codebase analysis commands
- Configuration commands

Each command includes:
- Name, description, and examples
- Argument definitions with types
- Handler implementation
- Category for grouping
- Auth requirements

## Type Definitions

### CommandDef
Complete command specification:
```typescript
{
  name: string;
  description: string;
  examples?: string[];
  args?: CommandArgDef[];
  handler: (args) => Promise<any>;
  aliases?: string[];
  category?: string;
  requiresAuth?: boolean;
  interactive?: boolean;
  hidden?: boolean;
}
```

### CommandArgDef
Argument specification:
```typescript
{
  name: string;
  description: string;
  type: ArgType;
  required?: boolean;
  default?: any;
  choices?: string[];
  position?: number;  // for positional args
  shortFlag?: string; // e.g., 'v' for -v
  hidden?: boolean;
}
```

## Dependencies
- **Internal**:
  - `errors/formatter`: User-friendly error messages
  - `utils/logger`: Logging
  - `utils/validation`: Input validation
- **External**:
  - Depends on injected dependencies (terminal, auth, ai, codebase, fileOps, execution)

## Usage Example

```typescript
import { initCommandProcessor, commandRegistry, ArgType } from './commands/index.js';

// Initialize with dependencies
const commandProcessor = await initCommandProcessor(config, {
  terminal, auth, ai, codebase, fileOps, execution, errors
});

// Register a custom command
commandRegistry.register({
  name: 'greet',
  description: 'Greet the user',
  category: 'Utility',
  args: [{
    name: 'name',
    description: 'Name to greet',
    type: ArgType.STRING,
    position: 0,
    required: true
  }, {
    name: 'uppercase',
    description: 'Use uppercase',
    type: ArgType.BOOLEAN,
    shortFlag: 'u'
  }],
  handler: async (args) => {
    let greeting = `Hello, ${args.name}!`;
    if (args.uppercase) {
      greeting = greeting.toUpperCase();
    }
    console.log(greeting);
  },
  examples: [
    'greet Alice',
    'greet Bob --uppercase',
    'greet Charlie -u'
  ]
});

// Execute a command
await commandProcessor.executeCommand('greet', ['Alice', '--uppercase']);

// Start interactive mode
await commandProcessor.startCommandLoop();

// List commands by category
const aiCommands = commandProcessor.getCommandsByCategory('AI');
```

## Command Parsing Features

**Positional arguments**:
```bash
claude-code explain <file> <function>
```

**Flag arguments**:
```bash
claude-code chat --model opus --temperature 0.7
```

**Short flags**:
```bash
claude-code chat -m opus -t 0.7
```

**Boolean flags**:
```bash
claude-code refactor --verbose
```

**Array arguments**:
```bash
claude-code analyze --ignore node_modules,dist,.git
```

**Choices validation**:
```typescript
{
  name: 'level',
  type: ArgType.STRING,
  choices: ['debug', 'info', 'warn', 'error']
}
```

## Important Notes
- **Singleton registry**: Commands registered globally via `commandRegistry`
- **Type conversion**: Automatic conversion based on argument type
- **Error handling**: Detailed validation errors with resolution suggestions
- **Help generation**: Auto-generated from command definitions
- **Category organization**: Commands grouped by category in help
- **Alias support**: Multiple names for the same command
- **Interactive REPL**: Full command loop with prompt and error handling
- **Dependency injection**: Commands receive all app dependencies
