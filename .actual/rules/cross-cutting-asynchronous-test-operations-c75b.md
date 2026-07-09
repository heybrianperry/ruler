# Standardize on describe/it Test Structure with Jest-Compatible Framework: Asynchronous Test Operations

These rules are ALWAYS ACTIVE for all TypeScript test files in the `tests/` directory, including unit tests, integration tests, and functional tests.

### Rules

- **R-ASYNC-001** MUST: Asynchronous test operations MUST use async/await syntax with proper error handling.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count lifecycle hooks
grep -r "beforeEach\|afterEach\|beforeAll\|afterAll" tests/ | wc -l

# Count test files
npm test -- --listTests | grep -E '\.test\.ts$' | wc -l

# Verify async/await usage in test files
grep -r "async\s*" tests/ | grep -E "(beforeEach|afterEach|beforeAll|afterAll|it)" | wc -l
```

**Accept when:**
- All test files in `tests/` directory use `describe` blocks for test suite organization
- All individual test cases are defined using `it` blocks with descriptive names
- Test files requiring setup/teardown use appropriate lifecycle hooks (`beforeEach`, `afterEach`, `beforeAll`, `afterAll`)
- Asynchronous operations in test files and lifecycle hooks use `async/await` syntax
- Test suite executes successfully with no hanging tests or resource leaks
- No promises are left unhandled in test operations

<enforcement>
Clause Code MUST NOT skip or defer verification of async/await compliance in test files. All asynchronous test operations must be reviewed for proper error handling and await usage before merge.
</enforcement>