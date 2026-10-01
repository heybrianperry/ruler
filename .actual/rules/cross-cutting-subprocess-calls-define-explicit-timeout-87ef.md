# child_process Module Adoption for Subprocess Management: Subprocess Calls Define Explicit Timeout Thresholds

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SUB-001** MUST: Subprocess calls MUST define explicit timeout thresholds and error handling routines to prevent hung processes and resource leakage.

### Verify

```bash
find . -type f \( -name "*.ts" -o -name "*.js" \) -not -path "*/.*" -exec grep -l "child_process" {} +
SCRIPT=$(cat <(find . -maxdepth 2 -name "*manifest*" -o -name "*json*") 2>/dev/null | grep -o '"test": *"[^"]*"' | head -n 1 | cut -d'"' -f4) && [ -n "$SCRIPT" ] && sh -c "$SCRIPT"
```

**Accept when:**
- All references to subprocess invocation resolve exclusively through the child_process module.
- Test suites pass successfully with subprocess mocks in place and without unexpected child process termination.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>