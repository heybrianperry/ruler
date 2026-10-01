# Adoption of IAgent Interface Module for Agent Implementations: Agent Implementations Not Expose Proprietary Provider

These rules are ALWAYS ACTIVE for agent integration modules, agent utility routines, and agent orchestration dispatchers within the agent subsystem, as well as new coding assistant or automation agent integrations being introduced to the codebase.

### Rules

- **R-AGENT-001** MUST_NOT: Agent implementations MUST NOT expose proprietary provider APIs directly to orchestrating consumers without adapting them through the IAgent contract.
- **R-AGENT-002** MANDATORY: Ensure that all newly introduced agent providers provide concrete implementations satisfying the IAgent contract before exposing them in the agent index export.
- **R-AGENT-003** MANDATORY: Shared capabilities and foundational file system interactions should be centralized in an abstract base agent class rather than duplicated across individual agent implementations.
- **R-AGENT-004** MANDATORY: Before writing code that uses a versioned library, execute the LOCK-VERSION GROUNDING process (find dependency manifest, identify build tool, inspect lock/resolution artifact, look up official documentation for exact version, confirm every API exists in that version).

### Verify

```bash
# Discover the repository typecheck or build script from the project manifest and execute it
# Example discovery:
BUILD_CMD=$(npm pkg get scripts.build --json 2>/dev/null || echo "")
if [ -n "$BUILD_CMD" ] && [ "$BUILD_CMD" != "null" ]; then
  npm run build
else
  # Fallback to standard check if manifest differs
  echo "Verify build/typecheck script via project manifest."
fi

# Discover the repository test runner script and execute the agent test suite
TEST_CMD=$(npm pkg get scripts.test --json 2>/dev/null || echo "")
if [ -n "$TEST_CMD" ] && [ "$TEST_CMD" != "null" ]; then
  npm test
else
  echo "Verify test script via project manifest."
fi
```

**Accept when:**
- All agent integration classes strictly implement the IAgent contract with zero type checking or compilation errors.
- Agent orchestration workflows invoke diverse agent implementations solely through the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Continuous integration failures prevent merging code containing agent implementations that do not satisfy the IAgent contract.
</enforcement>