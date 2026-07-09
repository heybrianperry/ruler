# Validate JSON Input Before Parsing in VSCode Settings Management UPDATE

These rules are ALWAYS ACTIVE for all files matching the configured scope, particularly src/vscode/settings.ts and any module that deserializes JSON input from file content or external sources into in-memory caches or data structures.

### Rules

- **R-JSON-001** MUST: Wrap all JSON.parse() operations in try-catch blocks to handle parse exceptions gracefully and prevent malformed data from corrupting the in-memory cache or causing runtime failures.
- **R-JSON-002** MUST: Validate JSON input at the parsing boundary before persisting deserialized data to cache layers (e.g., existingServerMap.set() operations).
- **R-JSON-003** SHOULD: Log or report parse errors with sufficient context to aid debugging without exposing sensitive configuration details.

### Verify

```bash
# Check that all JSON.parse() calls in src/vscode/settings.ts are wrapped in try-catch
grep -n "JSON\.parse" src/vscode/settings.ts | while read line; do
  line_num=$(echo "$line" | cut -d: -f1)
  context=$(sed -n "$((line_num-2)),$((line_num+2))p" src/vscode/settings.ts)
  if ! echo "$context" | grep -q "try"; then
    echo "FAIL: JSON.parse at line $line_num not wrapped in try-catch"
    exit 1
  fi
done
echo "PASS: All JSON.parse() calls are wrapped in try-catch blocks"
```

**Accept when:**
- All JSON.parse() operations in src/vscode/settings.ts are wrapped in try-catch blocks
- Parse exceptions are caught and handled before data is written to existingServerMap or other cache structures
- Error handling prevents malformed JSON from corrupting in-memory state
- Public contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings) validate input before cache operations

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON parsing safety. All JSON deserialization boundaries must be protected with exception handling before this rule is considered satisfied.
</enforcement>