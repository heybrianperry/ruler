<rule_activation id="db12dac3-1acb-4c70-b8b7-49ff127bb1af" title="Adopt TypeScript Test Files with .test.ts Extension as Standard Testing Convention: Unit Tests Placed" applies_to="**/*.test.ts">
These rules are ALWAYS ACTIVE for all TypeScript test file creation and organization within the codebase.
</rule_activation>

### Rules

- **R-TS-001** SHOULD: Unit tests SHOULD be placed in tests/unit/ directory and mirror the source code structure.

### Verify

```bash
# Verify all test files follow .test.ts naming convention
find tests/ -type f -name '*.ts' ! -name '*.test.ts' | grep -v 'helpers\|fixtures\|mocks' && echo 'Found non-test TypeScript files in tests/' || echo 'All test files follow .test.ts convention'

# Verify test runner is configured for .test.ts pattern
grep -r 'testMatch\|testRegex' jest.config.* tsconfig.json package.json | grep -q '\.test\.ts' && echo 'Test runner configured for .test.ts pattern' || echo 'Warning: Test runner may not be configured correctly'

# Verify test directory structure exists
ls tests/unit/ tests/integration/ tests/e2e/ 2>/dev/null && echo 'Test directory structure exists' || echo 'Warning: Expected test directories not found'
```

**Accept when:**
- All TypeScript test files in the codebase use the .test.ts extension and are located under the tests/ directory
- Test runner configuration includes glob patterns that match **/*.test.ts files
- CI/CD pipeline successfully discovers and executes all test files without manual configuration of individual test paths
- No TypeScript test files exist in production source directories outside of tests/

<enforcement>
Claude Code MUST verify test file naming conventions and directory structure before accepting TypeScript test files. Verification is mandatory and MUST NOT be skipped or deferred.
</enforcement>