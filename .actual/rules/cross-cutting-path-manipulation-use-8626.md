# Standardize Core Library Imports for Subagent File Processing: Path Manipulation Use

These rules are ALWAYS ACTIVE for all modules in src/core/ that process subagent files, including SubagentsUtils.ts and SubagentsProcessor.ts, and all functions that parse, validate, discover, or write subagent configuration files.

### Rules

- **R-SUBAGENT-PATH-001** MUST: Path manipulation MUST use the 'path' module to ensure cross-platform compatibility.

### Verify

```bash
# Verify path module is imported in subagent processing modules
grep -r "import.*from 'path'" src/core/Subagents*.ts

# Verify no hardcoded path separators in subagent processing
grep -r "['\"].*[/\\\\].*['\"]" src/core/Subagents*.ts | grep -v "node_modules" | grep -v "path.join\|path.resolve"

# Verify fs/promises is used for file operations
grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts

# Verify no synchronous fs methods in async contexts
grep -r "fs\.readFileSync\|fs\.writeFileSync\|fs\.readdirSync" src/core/Subagents*.ts
```

**Accept when:**
- All subagent processing modules import 'path' from the standard library
- path.join() and path.resolve() are used consistently for all path operations
- No hardcoded forward slashes or backslashes appear in path strings
- fs/promises is used for all file operations in SubagentsUtils.ts and SubagentsProcessor.ts
- No synchronous fs methods are present in async subagent processing functions

<enforcement>
Claude Code MUST NOT skip or defer verification of path module usage in subagent processing modules. All path manipulation MUST use the 'path' module for cross-platform compatibility.
</enforcement>