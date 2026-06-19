# Isolate MCP Server Configuration State Using Set-Based Deduplication: Use Set Data

These rules are ALWAYS ACTIVE for all agent configuration systems that manage MCP server state across nested directory hierarchies, including MCP server configuration files (.mcp.json), gitignore entries for agent-specific configuration artifacts, nested directory hierarchies with per-directory .ruler/ruler.toml configuration, configuration backup files with .bak extension, and JSON-based configuration state managed by FirebenderAgent and similar agent implementations.

### Rules

- **R-MCP-SET-001** MUST: Use Set data structures to deduplicate configuration artifact paths before writing to shared state (e.g., gitignore entries).
- **R-MCP-SET-002** MUST: Initialize Set data structures at the beginning of configuration processing functions before any add() operations to ensure clean state.
- **R-MCP-SET-003** MUST: Use path.relative() and path.sep normalization when adding filesystem paths to Sets to ensure consistent key representation across platforms.
- **R-MCP-SET-004** MUST: Wrap all JSON.parse() calls in try-catch blocks with console.warn() logging for error handling.
- **R-MCP-SET-005** MUST: Create backup files synchronously before writing new configuration to ensure atomic rollback capability on write failure.
- **R-MCP-SET-006** SHOULD: Implement extractServers() helper functions to normalize access to configuration data structures (mcpServers vs servers field names).
- **R-MCP-SET-007** SHOULD: Log all JSON.parse() errors with file path context to aid debugging of malformed configuration files.
- **R-MCP-SET-008** SHOULD: Implement key-based uniqueness checks that include configuration content hash, not just artifact paths, to detect conflicting server definitions.
- **R-MCP-SET-009** MAY: Implement backup rotation policy (keep last N backups) or time-based cleanup to prevent excessive disk space consumption.

### Verify

```bash
# Verify Set usage for configuration deduplication
grep -r 'new Set<' src/ tests/ | grep -i 'gitignore\|config\|mcp'

# All JSON.parse calls must have error handling
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l
# Should return 0

# Validate all MCP JSON files
find . -name '*.mcp.json' -exec sh -c 'jq empty "$1" 2>/dev/null || echo "Invalid JSON: $1"' _ {} \;
```

**Accept when:**
- All configuration artifact collections use Set data structures and demonstrate O(1) membership testing in code review
- All JSON.parse() calls are wrapped in try-catch blocks with appropriate error logging to console.warn() or similar
- Integration tests verify that MCP server definitions are isolated by directory scope and do not leak between parent/child configurations
- Backup files are created before configuration writes and gitignore entries include both primary and .bak file patterns
- Path normalization using path.relative() and path.sep is applied consistently when adding filesystem paths to Sets
- Configuration validation includes content hash-based uniqueness checks to detect conflicting server definitions with identical names

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-MCP-SET rules marked MUST are mandatory for code review acceptance. Violations result in CI build failure and code review rejection. JSON.parse() calls without error handling trigger static analysis failures. Configuration validation tests must pass before merge.
</enforcement>