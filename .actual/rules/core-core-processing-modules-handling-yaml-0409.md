# Adoption of js-yaml for Core YAML Document Processing: Core Processing Modules Handling Yaml Document

These rules are ALWAYS ACTIVE for core processing and utility modules performing YAML parsing, serialization, or configuration loading.

### Rules

- **R-YML-001** MUST: Core processing modules handling YAML document parsing and serialization MUST use js-yaml as the standard YAML processing library.
- **R-YML-002** MANDATORY: Discover the repository build script from the project manifest and execute the primary build target.
- **R-YML-003** MANDATORY: Discover the automated test suite runner from the project manifest and execute unit and integration test suites covering document processing modules.
- **R-YML-004** MANDATORY: Discover the static analysis and linting script from the project manifest and run source code validation across core modules.

### Verify

```bash
# Discover and execute project build script
BUILD_SCRIPT=$(node -p "const pkg = require('./package.json'); Object.keys(pkg.scripts || {}).find(s => s === 'build' || s.startsWith('build:')) || 'build'")
npm run $BUILD_SCRIPT

# Discover and execute test suite
TEST_SCRIPT=$(node -p "const pkg = require('./package.json'); Object.keys(pkg.scripts || {}).find(s => s === 'test' || s.includes('test')) || 'test'")
npm run $TEST_SCRIPT

# Discover and execute linter / static analysis
LINT_SCRIPT=$(node -p "const pkg = require('./package.json'); Object.keys(pkg.scripts || {}).find(s => s === 'lint' || s.includes('lint')) || 'lint'")
npm run $LINT_SCRIPT
```

**Accept when:**
- All core modules parsing YAML documents invoke js-yaml consistently without secondary parser imports.
- Repository verification and automated test suites confirm successful parsing and validation across document processing workflows.
- Dependency lock artifacts confirm resolved library versions match project specifications.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests introducing unauthorized parser dependencies or unsafe loading calls are blocked at review.
</enforcement>