# Isolate MCP Server Configuration State Using Set-Based Deduplication: Handle Json Parsing

These rules are ALWAYS ACTIVE for all agent configuration systems that manage MCP server state across nested directory hierarchies, including configuration files (.mcp.json), gitignore entries, and per-directory .ruler/ruler.toml files.

### Rules

- **R-MCP-JSON-001** MUST: Handle JSON parsing errors gracefully with console.warn() logging and continue processing remaining configurations.
- **R-MCP-JSON-002** MUST: Wrap all JSON.parse() calls in try-catch blocks with appropriate error logging.
- **R-MCP-JSON-003** MUST: Use Set data structures for configuration artifact collections to ensure O(1) membership testing and prevent duplicate entries.
- **R-MCP-JSON-004** MUST: Create backup files (.bak) synchronously before writing new configuration to ensure atomic rollback capability.
- **R-MCP-JSON-005** MUST: Normalize filesystem paths using path.relative() and path.sep when adding paths to Sets to ensure consistent key representation across platforms.
- **R-MCP-JSON-006** SHOULD: Implement extractServers() helper functions to normalize access to configuration data structures (mcpServers vs servers field names).
- **R-MCP-JSON-007** SHOULD: Log all JSON.parse() errors with file path context to aid debugging of malformed configuration files.
- **R-MCP-JSON-008** SHOULD: Implement key-based uniqueness checks that include configuration content hash, not just artifact paths, to detect conflicting server definitions.

### Verify

```bash
# Verify Set usage for configuration deduplication
grep -r 'new Set<' src/ tests/ | grep -i 'gitignore\|config\|mcp'

# All JSON.parse calls must have error handling - should return 0
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l

# Validate all MCP JSON files
find . -name '*.mcp.json' -exec sh -c 'jq empty "$1" 2>/dev/null || echo "Invalid JSON: $1"' _ {} \;
```

**Accept when:**
- All configuration artifact collections use Set data structures and demonstrate O(1) membership testing in code review
- All JSON.parse() calls are wrapped in try-catch blocks with appropriate error logging to console.warn() or similar
- Integration tests verify that MCP server definitions are isolated by directory scope and do not leak between parent/child configurations
- Backup files are created before configuration writes and gitignore entries include both primary and .bak file patterns
- All .mcp.json files pass jq validation or equivalent JSON schema validation
- JSON.parse() errors are logged with file path context for debugging

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON.parse() calls MUST be wrapped in try-catch blocks. All configuration collections MUST use Set data structures. Violations result in CI build failure and code review rejection.
</enforcement>