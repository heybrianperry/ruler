# Isolate MCP Server Configuration State Using Set-Based Deduplication: Parse Validate Json

These rules are ALWAYS ACTIVE for all agent configuration systems that manage MCP server state across nested directory hierarchies, including MCP server configuration files (.mcp.json), gitignore entries for agent-specific configuration artifacts, nested directory hierarchies with per-directory .ruler/ruler.toml configuration, configuration backup files with .bak extension, and JSON-based configuration state managed by FirebenderAgent and similar agent implementations.

### Rules

- **R-MCP-PARSE-001** MUST: Parse and validate JSON configuration files using JSON.parse() before merging or applying configuration state.
- **R-MCP-PARSE-002** MUST: Wrap all JSON.parse() calls in try-catch blocks with console.warn() logging to handle malformed configuration files gracefully.
- **R-MCP-PARSE-003** MUST: Use Set data structures (not arrays) for tracking configuration artifacts and gitignore entries to ensure O(1) membership testing and prevent duplicate state accumulation.
- **R-MCP-PARSE-004** MUST: Initialize Set data structures at the beginning of configuration processing functions before any add() operations to ensure clean state.
- **R-MCP-PARSE-005** MUST: Create backup files synchronously before writing new configuration to ensure atomic rollback capability on write failure.
- **R-MCP-PARSE-006** MUST: Isolate MCP server definitions by directory scope and prevent configuration leakage between parent and child directories.
- **R-MCP-PARSE-007** SHOULD: Use path.relative() and path.sep normalization when adding filesystem paths to Sets to ensure consistent key representation across platforms.
- **R-MCP-PARSE-008** SHOULD: Implement extractServers() helper functions to normalize access to configuration data structures (mcpServers vs servers field names).
- **R-MCP-PARSE-009** SHOULD: Log all JSON.parse() errors with file path context to aid debugging of malformed configuration files.
- **R-MCP-PARSE-010** SHOULD: Implement key-based uniqueness checks that include configuration content hash, not just artifact paths, to detect conflicting server definitions.
- **R-MCP-PARSE-011** MAY: Implement backup rotation policy (keep last N backups) or time-based cleanup to prevent excessive disk space consumption from .bak files.

### Verify

```bash
# Verify Set usage for configuration deduplication
grep -r 'new Set<' src/ tests/ | grep -i 'gitignore\|config\|mcp'

# Verify all JSON.parse calls have error handling (should return 0 unprotected calls)
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l

# Validate all MCP JSON files are valid JSON
find . -name '*.mcp.json' -exec sh -c 'jq empty "$1" 2>/dev/null || echo "Invalid JSON: $1"' _ {} \;

# Verify backup file patterns are included in gitignore entries
grep -r '\.bak' .gitignore .ruler/ruler.toml
```

**Accept when:**
- All configuration artifact collections use Set data structures and demonstrate O(1) membership testing in code review
- All JSON.parse() calls are wrapped in try-catch blocks with appropriate error logging to console.warn() or similar
- Integration tests verify that MCP server definitions are isolated by directory scope and do not leak between parent/child configurations
- Backup files are created before configuration writes and gitignore entries include both primary and .bak file patterns
- No unprotected JSON.parse() calls exist in the codebase (verification command returns 0)
- All .mcp.json files pass JSON validation via jq or equivalent validator

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for configuration processing code. Violations result in CI build failure and code review rejection.
</enforcement>