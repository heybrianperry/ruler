# child_process Module Adoption for Subprocess Management: Developers Inspect Authoritative Lock Artifact Repository

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CP-001** MUST: Developers MUST inspect the authoritative lock artifact in the repository to resolve the exact dependency versions and runtime environment specifications before implementing or modifying subprocess integrations.

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