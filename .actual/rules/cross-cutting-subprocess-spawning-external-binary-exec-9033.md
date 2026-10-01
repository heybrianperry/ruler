# child_process Module Adoption for Subprocess Management: Subprocess Spawning External Binary Execution Coordinated

These rules are ALWAYS ACTIVE for components requiring invocation of host operating system binaries or external system utilities, and test setup files configuring mocks and stubs for subprocess execution.

### Rules

- **R-CP-001** MUST: All subprocess spawning and external binary execution MUST be coordinated through the child_process module.
- **R-CP-002** MUST: Avoid shell invocation mode and pass arguments strictly as disjoint array elements directly to the binary to prevent command injection vulnerabilities.
- **R-CP-003** MUST: Enforce strict execution timeouts and attach error and close event listeners to all spawned process instances to prevent indefinite execution hangs.

### Verify

```bash
find . -type f \( -name "*.ts" -o -name "*.js" \) -not -path "*/.*" -exec grep -l "child_process" {} +
SCRIPT=$(cat <(find . -maxdepth 2 -name "*manifest*" -o -name "*json*") 2>/dev/null | grep -o '"test": *"[^"]*"' | head -n 1 | cut -d'"' -f4) && [ -n "$SCRIPT" ] && sh -c "$SCRIPT"
```

**Accept when:**
- All references to subprocess invocation resolve exclusively through the child_process module.
- Test suites pass successfully with subprocess mocks in place and without unexpected child process termination.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated static analysis checks and test suite runs in continuous integration pipelines, as well as mandatory architectural peer review.
</enforcement>