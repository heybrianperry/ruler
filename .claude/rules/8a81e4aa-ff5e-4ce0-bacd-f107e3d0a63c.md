<rule_activation id="8a81e4aa-ff5e-4ce0-bacd-f107e3d0a63c" title="Adopt Async/Await Concurrency Model for External API Interactions: Developers Use Async" applies_to="**/*">
These rules are ALWAYS ACTIVE for all external API interactions and public-facing API implementations within the system.
</rule_activation>

### Rules

- **R-ASYNC-001** MAY: Developers MAY use async generators (async function*) for streaming API responses or paginated data

### Scope

**In scope:**
- All HTTP/HTTPS requests to external services
- Database queries and transactions
- File system operations that may block (large files, network mounts)
- Third-party SDK calls that perform I/O
- WebSocket and streaming API interactions
- Public API endpoints exposed by this system

**Out of scope:**
- Pure computational functions with no I/O
- Synchronous utility functions (string manipulation, math operations)
- In-memory data structure operations
- Configuration object initialization

**Exceptions:**
- EXC-001: Legacy code integration where refactoring async boundaries would require extensive changes across module boundaries
- EXC-002: Performance-critical hot paths where async overhead is measured and documented to cause unacceptable latency

### Verify

```bash
# Check for synchronous API calls
grep -r 'function.*(' src/ | grep -v 'async' | grep -E '(fetch|axios|http|request)' || echo 'No synchronous API calls found'

# Enforce no-floating-promises rule
eslint --rule '@typescript-eslint/no-floating-promises: error' src/

# Count .then() usages and warn if excessive
grep -r '\.then(' src/ --include='*.ts' | wc -l | awk '{if($1>10) print "Warning: Found " $1 " .then() usages, prefer async/await"; else print "OK"}'
```

**Accept when:**
- All functions making external API calls are declared with async keyword and return Promise types
- ESLint checks pass with no-floating-promises and require-await rules enabled
- Code review confirms proper try/catch error handling around all await statements for external APIs
- No synchronous blocking operations detected in API interaction code paths

<enforcement>
Claude Code MUST NOT skip or defer verification. All async/await patterns MUST be verified via ESLint rules in CI/CD pipeline, TypeScript compiler strict mode, code review checklist, and automated static analysis before accepting changes.
</enforcement>