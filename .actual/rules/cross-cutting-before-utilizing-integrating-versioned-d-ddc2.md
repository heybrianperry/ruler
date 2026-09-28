# Adoption of FileSystemUtils Core Module: Before Utilizing Integrating Versioned Dependencies Supporting

These rules are ALWAYS ACTIVE for agent adapter modules requiring filesystem interactions, workspace inspections, and configuration management components that read, parse, or persist local settings.

### Rules

- **R-FSU-001** MUST: Before utilizing or integrating versioned dependencies supporting filesystem interactions, consumers MUST inspect the repository lock artifact to verify the exact resolved version against documented behavior.
- **R-FSU-002** MUST: When adding new filesystem capabilities needed across multiple agents, extend FileSystemUtils rather than embedding custom routines inside individual adapters.
- **R-FSU-003** MUST: Ensure all filesystem operations exported by FileSystemUtils handle asynchronous exceptions and normalize path separators consistently across operating environments.

### Verify

```bash
# Discover and execute the project verification script from the repository manifest to validate module import compliance
# Execute the repository test runner discovered through workspace configuration to confirm filesystem utility integration tests pass
BUILD_TOOL=$(npx -v &>/dev/null && echo "npm" || echo "yarn")
if [ -f "package.json" ]; then
  npm test || yarn test
elif [ -f "pom.xml" ]; then
  mvn test
elif [ -f "build.gradle" ]; then
  ./gradlew test
else
  echo "No recognized build tool manifest found for verification."
  exit 1
fi
```

**Accept when:**
- All agent adapter modules and configuration handlers access shared filesystem operations exclusively through the FileSystemUtils core module.
- All automated integration and unit test suites defined in the repository pass without module resolution or filesystem access errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull requests introducing duplicated filesystem helper functions or bypassing FileSystemUtils for standardized operations will be blocked during code review. Violations identified during static analysis must be refactored to use the centralized utility module.
</enforcement>