# Constants Module Adoption for Core Configuration and Domain Values: When Referencing Versioned External Dependencies Supporting

These rules are ALWAYS ACTIVE for core subsystem modules and utilities requiring access to shared system keys, operational defaults, cross-cutting configuration parameters, and when referencing versioned external dependencies supporting module resolution.

### Rules

- **R-CONST-001** MUST: When referencing versioned external dependencies supporting module resolution, consumers MUST inspect the repository lock artifact to determine the exact resolved dependency version prior to implementation.
- **R-CONST-002** MUST: Core operational modules and processing/agent subsystems MUST use the centralized constants module for shared configuration keys, environment variable references, and system operational defaults rather than hardcoding string literals independently.
- **R-CONST-003** MUST: Constants exported from the central module MUST be defined as read-only or immutable structures to prevent runtime mutation, and MUST be grouped logically by operational concern.

### Verify

```bash
# Discover and run the project static analysis and type verification script
# (Command must be derived from the repository configuration, e.g., package.json, Makefile, etc.)
if [ -f "package.json" ]; then
  npm run typecheck || npx tsc --noEmit
elif [ -f "pyproject.toml" ] || [ -f "setup.py" ]; then
  mypy .
fi

# Discover and run the project test suite
if [ -f "package.json" ]; then
  npm test
elif [ -f "pytest.ini" ] || [ -f "pyproject.toml" ]; then
  pytest
fi
```

**Accept when:**
- Static analysis and type checking pass with zero errors across all core modules importing from the constants module.
- All unit and integration test suites pass successfully, confirming consistent behavior across core processing workflows.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis and peer code reviews will block pull requests that duplicate shared literal strings or bypass the centralized constants module.
</enforcement>