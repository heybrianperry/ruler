# Adoption of IAgent Interface Module for Agent Implementations: Agent Implementations Extend Shared Abstract Base

These rules are ALWAYS ACTIVE for all agent integration modules, agent utility routines, and agent orchestration dispatchers within the agent subsystem, including all new coding assistant or automation agent integrations being introduced to the codebase.

### Rules

- **R-AGT-001** SHOULD: Agent implementations SHOULD extend the shared abstract base agent class to inherit common lifecycle hooks and file system operations rather than re-imposing or re-implementing core execution flows.

### Verify

```bash
# Discover and execute the project typecheck or build script
BUILD_SCRIPT=$(node -p "
  const fs = require('fs');
  const pkg = JSON.parse(fs.readFileSync('package.json', 'utf8'));
  const scripts = pkg.scripts || {};
  const candidate = Object.keys(scripts).find(k => k.match(/typecheck|build|compile/));
  candidate ? scripts[candidate] : ''
")
if [ -n "$BUILD_SCRIPT" ]; then
  eval "$BUILD_SCRIPT"
fi

# Discover and execute the agent test suite
TEST_SCRIPT=$(node -p "
  const fs = require('fs');
  const pkg = JSON.parse(fs.readFileSync('package.json', 'utf8'));
  const scripts = pkg.scripts || {};
  const candidate = Object.keys(scripts).find(k => k.match(/test|agent/));
  candidate ? scripts[candidate] : ''
")
if [ -n "$TEST_SCRIPT" ]; then
  eval "$TEST_SCRIPT"
fi
```

**Accept when:**
- All agent integration classes strictly implement the IAgent contract with zero type checking or compilation errors.
- Agent orchestration workflows invoke diverse agent implementations solely through the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated CI build checks, static typing, architecture peer code reviews, and enforcement of the IAgent contract.
</enforcement>