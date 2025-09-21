---
description: How to add or edit Claude Code Commands in our project by following the Cursor rules formatting structure
globs: 
alwaysApply: false
---
# Claude Code Commands Location

How to add new Claude Code Commands to the project

1. Always place command files in PROJECT_ROOT/.claude/commands/:
    ```
    .claude/commands/
    ├── your-command-name.md
    ├── another-command.md
    └── ...
    ```

2. Follow the naming convention:
    - Use kebab-case for filenames
    - Always use .md extension
    - Make names descriptive of the commmands's purpose

3. Directory structure:
    ```
    PROJECT_ROOT/
    ├── .claude/
    │   └── commands/
    │       ├── your-command-name.md
    │       └── ...
    └── ...
    ```

4. Never place command files:
    - In the project root
    - In subdirectories outside .claude/commands
    - In any other location

5. Cursor rules have the following structure (Use this structure):

````
---
description: Short description of the commands's purpose
globs: optional/path/pattern/**/* 
alwaysApply: false
---
# Command Title

Main content explaining the command with markdown formatting.

1. Step-by-step instructions
2. Code examples
3. Guidelines

Example:
```typescript
// Good example
function goodExample() {
  // Implementation following guidelines
}

// Bad example
function badExample() {
  // Implementation not following guidelines
}
```
````
