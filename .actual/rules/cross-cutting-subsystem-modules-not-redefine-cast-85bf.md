# Adoption of Internal Types Module for Centralized Domain Contracts: Subsystem Modules Not Redefine Cast Divergent

These rules are ALWAYS ACTIVE for domain models, data transfer objects, configuration interfaces, and protocol definitions shared across subsystems.

### Rules

- **R-DOM-001** MUST_NOT: Subsystem modules MUST NOT redefine or cast divergent structural interfaces for domain entities managed by the internal types module.

### Verify

```bash
# Discover and execute the project-wide type validation script
type_checker=$(npm pkg get scripts.typecheck --json 2>/dev/null || echo "")
if [ -n "$type_checker" ] && [ "$type_checker" != "null" ]; then
  npm run typecheck
else
  npx tsc --noEmit
fi

# Run the project automated test suite
if npm run | grep -q "test"; then
  npm test
fi
```

**Accept when:**
- All subsystem modules import shared contracts from the designated internal types module without local interface duplication.
- Static type analysis across the entire project repository completes with zero errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static type analysis and peer code review.
</enforcement>