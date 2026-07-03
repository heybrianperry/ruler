# Isolate MCP Server Configuration State Using Set-Based Deduplication: Create Backup Files

These rules are ALWAYS ACTIVE for all agent configuration systems that manage MCP server state across nested directory hierarchies, including FirebenderAgent implementations and similar systems managing .mcp.json configuration files.

### Rules

- **R-MCP-001** SHOULD: Create backup files (.bak suffix) before overwriting existing configuration files to enable rollback on validation failure.
- **R-MCP-002** MUST: Use Set data structures (not arrays) for tracking configuration artifacts and gitignore entries to ensure O(1) membership testing and prevent duplicate state accumulation.
- **R-MCP-003** MUST: Wrap all JSON.parse() calls in try-catch blocks with console.warn() logging to handle malformed configuration files gracefully.
- **R-MCP-004** SHOULD: Normalize filesystem paths using path.relative() and path.sep when adding paths to Sets to ensure consistent key representation across platforms.
- **R-MCP-005** SHOULD: Implement extractServers() helper functions to normalize access to configuration data structures (mcpServers vs servers field names) across different configuration file formats.
- **R-MCP-006** MUST: Isolate MCP server definitions by directory scope—do not automatically inherit parent directory server definitions in child directories without explicit configuration.
- **R-MCP-007** SHOULD: Create backup files synchronously before writing new configuration to ensure atomic rollback capability on write failure.
- **R-MCP-008** SHOULD: Log all JSON.parse() errors with file path context to aid debugging of malformed configuration files.
- **R-MCP-009** SHOULD: Implement backup rotation policy (keep last N backups) or time-based cleanup to prevent excessive disk space consumption from accumulated .bak files.

### Verify

```bash
# Verify Set usage for configuration deduplication
grep -r 'new Set<' src/ tests/ | grep -i 'gitignore\|config\|mcp'

# Verify all JSON.parse calls have error handling (should return 0 matches)
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l

# Validate all MCP JSON files are well-formed
find . -name '*.mcp.json' -exec sh -c 'jq empty "$1" 2>/dev/null || echo "Invalid JSON: $1"' _ {} \;

# Verify backup file patterns are included in gitignore entries
grep -r '\.bak' .gitignore .ruler/
```

**Accept when:**
- All configuration artifact collections use Set data structures and demonstrate O(1) membership testing in code review
- All JSON.parse() calls are wrapped in try-catch blocks with appropriate error logging to console.warn() or similar
- Integration tests verify that MCP server definitions are isolated by directory scope and do not leak between parent/child configurations
- Backup files are created before configuration writes and gitignore entries include both primary and .bak file patterns
- No array-based configuration state exists where Set-based deduplication should be used
- All .mcp.json files pass jq validation or equivalent JSON schema validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for MCP configuration management code. Violations must be caught during code review and CI pipeline validation before merge.
</enforcement>