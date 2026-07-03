# Isolate MCP Server Configuration State Using Set-Based Deduplication: Use Relative Path

These rules are ALWAYS ACTIVE for all agent configuration systems that manage MCP server state across nested directory hierarchies, including configuration files (.mcp.json), gitignore entries, and per-directory .ruler/ruler.toml files.

### Rules

- **R-MCP-001** MUST: Use Set data structures (not arrays) for tracking configuration artifacts and gitignore entries to ensure O(1) membership testing and prevent duplicate state accumulation.
- **R-MCP-002** MUST: Wrap all JSON.parse() calls in try-catch blocks with console.warn() logging that includes file path context for debugging malformed configuration files.
- **R-MCP-003** MUST: Use path.relative() and path.sep normalization when adding filesystem paths to Sets to ensure consistent key representation across platforms.
- **R-MCP-004** MUST: Create backup files (.bak extension) synchronously before writing new configuration to ensure atomic rollback capability on write failure.
- **R-MCP-005** SHOULD: Implement extractServers() helper functions to normalize access to configuration data structures (mcpServers vs servers field names) across different agent implementations.
- **R-MCP-006** SHOULD: Implement backup rotation policy (keep last N backups) or time-based cleanup to prevent excessive disk space consumption from accumulated .bak files.
- **R-MCP-007** MAY: Use relative path normalization when adding configuration artifacts to version control ignore files to ensure cross-platform compatibility.

### Verify

```bash
# Verify Set usage for configuration deduplication
grep -r 'new Set<' src/ tests/ | grep -i 'gitignore\|config\|mcp'

# Verify all JSON.parse calls have error handling (should return 0)
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l

# Validate all MCP JSON files are well-formed
find . -name '*.mcp.json' -exec sh -c 'jq empty "$1" 2>/dev/null || echo "Invalid JSON: $1"' _ {} \;
```

**Accept when:**
- All configuration artifact collections use Set data structures and demonstrate O(1) membership testing in code review
- All JSON.parse() calls are wrapped in try-catch blocks with appropriate error logging to console.warn() or similar
- Integration tests verify that MCP server definitions are isolated by directory scope and do not leak between parent/child configurations
- Backup files are created before configuration writes and gitignore entries include both primary and .bak file patterns
- Set data structures are initialized at the beginning of configuration processing functions before any add() operations
- Path normalization using path.relative() is applied consistently when adding filesystem paths to Sets

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-MCP-001 through R-MCP-007 are mandatory for configuration management code. Violations result in CI build failure, code review rejection, or runtime console.warn() logging with graceful degradation.
</enforcement>