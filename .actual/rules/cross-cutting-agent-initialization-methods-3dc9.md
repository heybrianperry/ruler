# Standardize Asynchronous Configuration Loading Pattern for Agent Initialization: Agent Initialization Methods

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading utilities within the codebase, including agent class constructors, initialization methods, and core utility modules that load configuration data from filesystem sources.

### Rules

- **R-ASYNC-001** MUST: Agent initialization methods that load configuration MUST be declared as async and return Promises.

### Verify

```bash
# Check for synchronous fs operations in agent and configuration loading modules
grep -r "fs\.readFileSync" src/agents/ src/core/Subagents* --include="*.ts" | grep -v "test" || echo "No synchronous fs operations found in agent code"

# Count async configuration loading functions
grep -r "async.*load\|async.*read\|async.*apply" src/agents/ src/core/Subagents* --include="*.ts" | wc -l

# Run tests for agent initialization and configuration loading
npm test -- --testPathPattern="(GeminiCliAgent|OpenCodeAgent|ZedAgent|SubagentsUtils|SubagentsProcessor)" --testNamePattern="async|configuration|loading"
```

**Accept when:**
- All agent configuration loading functions use async/await with fs/promises and no synchronous fs operations exist in agent initialization paths
- Unit tests for agent initialization properly await configuration loading and verify async behavior
- Code review confirms separation of I/O and parsing concerns with appropriate error handling for Promise rejections
- All configuration loading functions include JSDoc annotations documenting expected file formats, error conditions, and return types

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline linting rules enforce async/await patterns in designated modules. Violations result in build failure if synchronous filesystem operations are detected in agent initialization code, or if async configuration loading lacks proper error handling.
</enforcement>