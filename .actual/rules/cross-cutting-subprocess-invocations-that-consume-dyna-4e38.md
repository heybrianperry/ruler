# child_process Module Adoption for Subprocess Management: Subprocess Invocations That Consume Dynamic Input

These rules are ALWAYS ACTIVE for components requiring invocation of host operating system binaries or external system utilities, and test setup files configuring mocks and stubs for subprocess execution.

### Rules

- **R-SUB-001** MUST: Subprocess invocations that consume dynamic input MUST sanitize all arguments and avoid shell execution interpreters to prevent command injection vulnerabilities.

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