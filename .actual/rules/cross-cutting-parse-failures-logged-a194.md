# Validate JSON Input with Try-Catch Parsing in Integration Components: Parse Failures Logged

These rules are ALWAYS ACTIVE for all integration components that parse external JSON configuration or data, including file system operations using Node.js core libraries ('fs', 'path') and configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration).

### Rules

- **R-PARSE-001** MUST: Parse failures MUST be logged with context identifying the source file and error details (e.g., console.warn with file path and error message).
- **R-PARSE-002** MUST: All JSON.parse() calls on external input MUST be wrapped in try-catch blocks.
- **R-PARSE-003** MUST: Configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) MUST handle parse failures without throwing unhandled exceptions.
- **R-PARSE-004** SHOULD: Provide sensible fallback values or default configurations when parsing fails, ensuring the component can continue operating in degraded mode.
- **R-PARSE-005** SHOULD: Use FileSystemUtils helper module to centralize file read error handling and reduce duplication across integration components.

### Verify

```bash
# Verify no unwrapped JSON.parse calls on external input
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'test' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify FirebenderAgent has parse failure logging
grep -A5 'JSON\.parse.*existingContent' src/agents/FirebenderAgent.ts | grep -q 'console.warn.*Failed to read/parse'

# Verify configuration loading functions have try-catch blocks
grep -r 'loadExistingConfig\|saveConfig\|applyRulerConfig\|handleMcpConfiguration' src/ | xargs grep -l 'try.*catch'
```

**Accept when:**
- All JSON.parse operations on external input are wrapped in try-catch blocks with error logging
- Configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) handle parse failures without throwing unhandled exceptions
- Error logs include sufficient context (file path, error message) to diagnose configuration issues
- Integration components remain stable and operational even when external configuration files are malformed or corrupted

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse operations on external input require try-catch wrapping with logged context. Pull requests with unwrapped JSON.parse on external input are blocked until defensive error handling is added.
</enforcement>