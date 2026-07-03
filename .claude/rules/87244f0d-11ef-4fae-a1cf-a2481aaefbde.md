<rule_activation id="87244f0d-11ef-4fae-a1cf-a2481aaefbde" title="Adopt Unit Testing for CI/CD Pipeline Components: Tooling Tests Fast" applies_to="**/*">
These rules are ALWAYS ACTIVE for all CI/CD pipeline components, build tooling, and deployment automation code.
</rule_activation>

### Rules

- **R-CICD-001** SHOULD: CI/CD tooling tests SHOULD be fast-running and isolated, avoiding external dependencies to enable rapid feedback during development.

### In Scope

- All CI/CD agent implementations (FirebaseAgent, CodexCliAgent, OpenHandsAgent, GooseAgent, etc.)
- Configuration loaders and parsers for build and deployment systems
- Build tooling and automation scripts that affect deployment pipelines
- Path resolution and configuration transformation logic
- MCP (Model Context Protocol) configuration handling and propagation
- Revert engines and fix demonstration systems for CI/CD workflows

### Out of Scope

- End-to-end integration tests that require full pipeline execution
- Performance benchmarking tests for CI/CD systems
- Manual testing and exploratory testing of deployment workflows
- Production monitoring and observability of live pipelines
- Third-party CI/CD platform configurations (GitHub Actions, Jenkins, etc.) unless wrapped by custom tooling

### Exceptions

- **EXC-001**: Legacy CI/CD scripts scheduled for deprecation within 30 days
- **EXC-002**: Prototype or experimental CI/CD tooling not yet used in production pipelines

### Verify

```bash
# Count unit test files
find tests/unit -name '*.test.ts' -type f | wc -l

# Count test cases
grep -r "describe\|it\|test" tests/unit/ --include='*.test.ts' | wc -l

# Run unit tests
npm test -- --testPathPattern=tests/unit --passWithNoTests=false 2>&1 | grep -E '(PASS|FAIL|Tests:)'
```

**Accept when:**
- Unit test files exist under tests/unit/ directory with .test.ts extension for all CI/CD agent implementations and configuration loaders
- Test suite executes successfully with passing tests covering core functionality, error handling, and edge cases
- CI pipeline includes unit test execution as a required quality gate that must pass before merge
- Test execution time remains under 30 seconds for the full unit test suite
- Tests are organized mirroring source structure with appropriate use of fixtures and mocks

<enforcement>
Claude Code MUST verify unit test presence and execution status before approving changes to CI/CD components. Verification is mandatory and cannot be deferred.
</enforcement>