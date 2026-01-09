# Laravel Claude Code Plugins

A collection of Claude Code plugins tailored for PHP / Laravel development.

## Plugins

### laravel-simplifier

A code simplification agent that refines PHP / Laravel code for clarity, consistency, and maintainability while preserving functionality.

**Features:**

- Applies common Laravel conventions and standards
- Reduces unnecessary complexity and nesting
- Improves readability through clear naming
- Focuses on recently modified code by default
- Preserves all original functionality

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add laravel/claude-code
/plugin install laravel-simplifier@laravel
```

## Usage

Once installed, tell Claude Code to use the agent after a long coding session:

```
> Review recent changes using the laravel-simplifier agent
```
