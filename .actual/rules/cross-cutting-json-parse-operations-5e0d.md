# Validate JSON Input with Try-Catch Parsing in Integration Components: Json Parse Operations

These rules are ALWAYS ACTIVE for all integration components that parse external JSON configuration or data, including file system operations using Node.js core libraries ('fs', 'path') and configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration).

### Rules

- **R-JSON-001** MUST: All JSON.parse() operations on external input MUST be wrapped in try-catch blocks to prevent unhandled exceptions.

### Verify

```bash
# Verify no unwrapped JSON.parse calls on external input exist outside tests
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'test' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify FirebenderAgent has defensive parsing with error logging
grep -A5 'JSON\.parse.*existingContent' src/agents/FirebenderAgent.ts | grep -q 'console.warn.*Failed to read/parse'

# Verify configuration loading functions implement try-catch error handling
grep -r 'loadExistingConfig\|saveConfig\|applyRulerConfig\|handleMcpConfiguration' src/ | xargs grep -l 'try.*catch'
```

**Accept when:**
- All JSON.parse operations on external input are wrapped in try-catch blocks with error logging
- Configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) handle parse failures without throwing unhandled exceptions
- Error logs include sufficient context (file path, error message) to diagnose configuration issues
- Integration components remain stable and operational even when external configuration files are malformed or corrupted

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse operations on external input must be wrapped in try-catch blocks. Pull requests with unwrapped JSON.parse on external input are blocked until defensive error handling is added.
</enforcement>