# child_process Module Adoption for Subprocess Management: Test Environments Executing Code That Depends

These rules are ALWAYS ACTIVE for test environments executing code that depends on the child_process module.

### Rules

- **R-CP-001** SHOULD: Test environments executing code that depends on the child_process module SHOULD mock or stub subprocess interfaces rather than invoking live operating system binaries.

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