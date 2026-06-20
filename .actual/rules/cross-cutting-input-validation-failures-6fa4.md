# Validate JSON Input Before Parsing in VSCode Settings Management: Input Validation Failures

These rules are ALWAYS ACTIVE for all JSON parsing operations in VSCode settings management, including readVSCodeSettings, writeVSCodeSettings, and cache layer operations that handle server configuration state.

### Rules

- **R-INPUT-001** SHOULD: Input validation failures SHOULD log detailed error information including file path and validation failure reason.

### Verify

```bash
# Verify no unprotected JSON.parse calls in settings management
grep -n 'JSON\.parse' src/vscode/settings.ts | grep -v 'try\|catch' # Should return no unprotected JSON.parse calls

# Verify validation functions exist
grep -n 'function.*validate.*Settings\|isValid.*Settings' src/vscode/settings.ts # Should find validation functions

# Verify malformed JSON handling tests pass
npm test -- --grep 'settings.*validation.*malformed' # Should pass tests for malformed JSON handling
```

**Accept when:**
- All JSON.parse operations in src/vscode/settings.ts are wrapped in try-catch blocks
- Type guard functions exist for VSCodeSettings and AugmentMcpServer with required field validation
- Tests demonstrate graceful handling of malformed JSON with fallback to defaults and error logging
- Validation failures log file path, error type, and sanitized partial content for debugging

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON parsing operations must include validation with detailed error logging before merge.
</enforcement>
