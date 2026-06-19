# Isolate MCP Server Configuration State Using Set-Based Deduplication: Isolate Mcp Server

These rules are ALWAYS ACTIVE for all agent configuration systems that manage MCP server state across nested directory hierarchies, including MCP server configuration files (.mcp.json), gitignore entries for agent-specific configuration artifacts, nested directory hierarchies with per-directory .ruler/ruler.toml configuration, configuration backup files with .bak extension, and JSON-based configuration state managed by FirebenderAgent and similar agent implementations.

### Rules

- **R-MCP-001** MUST: Isolate MCP server definitions by directory scope, ensuring child directory configurations do not inherit parent server entries unless explicitly configured.
- **R-MCP-002** MUST: Use Set data structures for all configuration artifact collections to enable O(1) membership testing and prevent duplicate entries in version control ignore files.
- **R-MCP-003** MUST: Wrap all JSON.parse() calls in try-catch blocks with appropriate error logging to console.warn() or equivalent, including file path context.
- **R-MCP-004** MUST: Create backup files (.bak extension) synchronously before writing new configuration to ensure atomic rollback capability on write failure.
- **R-MCP-005** MUST: Normalize filesystem paths using path.relative() and path.sep when adding paths to Sets to ensure consistent key representation across platforms.
- **R-MCP-006** SHOULD: Implement extractServers() helper functions to normalize access to configuration data structures (mcpServers vs servers field names).
- **R-MCP-007** SHOULD: Implement backup rotation policy (keep last N backups) or time-based cleanup to prevent excessive disk space consumption from accumulated .bak files.
- **R-MCP-008** SHOULD: Implement key-based uniqueness checks that include configuration content hash, not just artifact paths, to detect conflicting server definitions with identical names.

### Verify

```bash
# Verify Set usage for configuration deduplication
grep -r 'new Set<' src/ tests/ | grep -i 'gitignore\|config\|mcp'

# Verify all JSON.parse calls have error handling (should return 0)
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l

# Validate all MCP JSON files
find . -name '*.mcp.json' -exec sh -c 'jq empty "$1" 2>/dev/null || echo "Invalid JSON: $1"' _ {} \;
```

**Accept when:**
- All configuration artifact collections use Set data structures and demonstrate O(1) membership testing in code review
- All JSON.parse() calls are wrapped in try-catch blocks with appropriate error logging to console.warn() or similar
- Integration tests verify that MCP server definitions are isolated by directory scope and do not leak between parent/child configurations
- Backup files are created before configuration writes and gitignore entries include both primary and .bak file patterns
- Path normalization using path.relative() and path.sep is applied consistently when adding filesystem paths to Sets
- Configuration validation includes content hash-based uniqueness checks to detect conflicting server definitions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code affecting MCP server configuration state management. Violations result in CI build failure, code review rejection, or runtime console.warn() logging with graceful degradation. Exceptions require engineering lead approval via pull request review with documented rationale and compensating controls.
</enforcement>