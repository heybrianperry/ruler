# Standardize Node.js Core Module Integration for Subagent File System Operations: Path Manipulation Operations

These rules are ALWAYS ACTIVE for all subagent file system operations and configuration parsing within the core module boundary, specifically targeting path manipulation operations in SubagentsProcessor.ts and SubagentsUtils.ts.

### Rules

- **R-PATH-001** MUST: Path manipulation operations MUST use the 'path' core module to ensure cross-platform compatibility

### Verify

```bash
# Verify path module is imported and used for all path operations
grep -r "from 'path'" src/core/Subagents*.ts

# Verify no hardcoded path separators in path manipulation
grep -E "['\"]\\\\['\"]|['\"]\\\\\\['\"]" src/core/Subagents*.ts || echo "No hardcoded separators found"

# Verify path.join() and path.resolve() usage
grep -E "path\.(join|resolve)" src/core/Subagents*.ts

# Verify fs/promises is used for async operations
grep -r "from 'fs/promises'" src/core/Subagents*.ts

# Verify no synchronous fs operations in core subagent modules
grep -r "require('fs')" src/core/Subagents*.ts | grep -v fs/promises || echo "No synchronous fs imports found"
```

**Accept when:**
- Path module is imported in SubagentsProcessor.ts and SubagentsUtils.ts
- All path manipulation uses path.join() or path.resolve() consistently
- No hardcoded path separators (forward or backward slashes) appear in path construction logic
- fs/promises is used exclusively for all file system operations
- No synchronous fs module operations exist in core subagent modules
- All public API functions (discoverSubagents, parseFrontmatter, validateFrontmatter, loadSubagentFile, propagateSubagentsForClaude) properly handle cross-platform paths

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules prohibiting synchronous fs module usage and hardcoded path separators in core subagent modules is mandatory. CI pipeline MUST fail on detection of violations.
</enforcement>