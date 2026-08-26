# Core Infrastructure Depends on Agent Abstraction Interface and Centralized Configuration Loader: Core Infrastructure Components Not Import Concrete

These rules are ALWAYS ACTIVE for all core infrastructure components responsible for agent orchestration, agent selection logic, and coordination of multiple agent implementations.

### Rules

- **R-CORE-001** MUST NOT: Core infrastructure components MUST NOT import concrete agent implementation classes directly; all agent interactions MUST occur through the IAgent interface.

### Verify

```bash
# Discover the project's module resolution configuration and verify that relative imports from core infrastructure to agent abstractions resolve correctly
find . -name "*.json" -o -name "*.toml" -o -name "*.yaml" -o -name "*.yml" | xargs grep -l "paths\|alias\|resolution" 2>/dev/null | head -5

# Locate and execute the project's static analysis or type checking tooling to verify that all core infrastructure imports of IAgent and ConfigLoader satisfy type contracts
if command -v mypy &> /dev/null; then mypy --strict core/; fi
if command -v pyright &> /dev/null; then pyright core/; fi
if command -v eslint &> /dev/null; then eslint core/ --rule 'no-restricted-imports'; fi

# Identify the project's test suite and run tests covering core infrastructure components to verify that agent abstraction dependencies function correctly
if [ -f "pytest.ini" ] || [ -f "setup.py" ]; then pytest core/ -v; fi
if [ -f "package.json" ]; then npm test -- core/; fi
if [ -f "go.mod" ]; then go test ./core/...; fi

# Verify that core infrastructure modules import IAgent interface rather than concrete agent classes
grep -r "from.*agent.*import" core/ | grep -v "IAgent" | grep -v "ConfigLoader" | grep -v "__pycache__" || echo "No direct concrete agent imports found"
```

**Accept when:**
- Static analysis confirms all core infrastructure modules import IAgent interface rather than concrete agent classes
- Type checking verifies that all uses of IAgent and ConfigLoader in core infrastructure satisfy their respective interface contracts
- Test suite passes for core infrastructure components, confirming that agent abstraction dependencies integrate correctly
- No direct imports of concrete agent implementation classes are detected in core infrastructure modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All core infrastructure components must be audited against R-CORE-001 before code is committed. Type checking and static analysis violations block merge until corrected.
</enforcement>