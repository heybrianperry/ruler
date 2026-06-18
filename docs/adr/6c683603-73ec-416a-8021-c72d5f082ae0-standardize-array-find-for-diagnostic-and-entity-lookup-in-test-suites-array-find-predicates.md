# Standardize Array.find() for Diagnostic and Entity Lookup in Test Suites: Array Find Predicates

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test suites require verification of specific entities within collections returned by configuration loaders and agent registries
- Integration and unit tests validate diagnostic messages, agent identifiers, and configuration warnings by searching through result arrays
- The codebase uses describe/it test frameworks (detected in tests/integration/skills-mcp.test.ts, tests/unified-config.mcp-toml-load.test.ts, tests/unified-config.mcp-json-warning.test.ts) requiring consistent data access patterns
- Test assertions depend on locating specific objects by property values (e.g., agent.getIdentifier() === 'zed', d.code === 'MCP_JSON_DEPRECATED') within collections

## Problem Statement

Test suites need a consistent, readable method to locate specific entities within collections for assertion purposes. Without a standardized approach, tests may use inconsistent patterns (filter, find, loops) leading to reduced readability and maintenance burden across the test codebase.

## Decision

1. MUST: Array.find() predicates MUST use explicit property comparisons (e.g., agent.getIdentifier() === 'value', d.code === 'CODE')

## Policy Block

- MUST Array.find() predicates MUST use explicit property comparisons (e.g., agent.getIdentifier() === 'value', d.code === 'CODE')

## Rationale

- Array.find() provides clear intent for single-entity lookup with early termination, matching the observed pattern across 3 test files with 91.70% confidence
- The pattern appears consistently in diagnostic validation (config.diagnostics.find) and agent registry lookups (allAgents.find), indicating established practice
- Explicit property comparisons in predicates improve test readability and make assertion failures easier to diagnose
- Standardizing on Array.find() reduces cognitive load when reading test suites and establishes predictable patterns for new test development

## Consequences

Positive:
- Improved test readability through consistent entity lookup patterns across integration and unit tests
- Reduced maintenance burden by establishing a single standard approach for collection searches
- Better test failure diagnostics when entity lookups return undefined
- Natural alignment with JavaScript/TypeScript idiomatic patterns for single-entity retrieval

Negative:
- Array.find() returns undefined when no match is found, requiring explicit undefined checks in assertions
- Performance characteristics differ from filter()[0] pattern, though negligible in test contexts
- May require refactoring existing tests using alternative patterns (filter, loops) to achieve consistency

## Alternatives

- Use Array.filter()[0] for entity lookup (rejected)
  Rejected because: filter() iterates entire array even after match found, and [0] access is less explicit about intent than find()
  When valid: When multiple matches are expected and first match is desired
- Use for loops with break on match (rejected)
  Rejected because: Verbose syntax reduces test readability and obscures lookup intent compared to declarative find()
  When valid: When complex multi-step logic is required during iteration
- Use lodash _.find() utility (rejected)
  Rejected because: Introduces external dependency when native Array.find() provides equivalent functionality
  When valid: When project already has lodash as a dependency and uses it extensively

## Risks

- Tests may fail to check for undefined results from find(), leading to runtime errors in test execution
  Mitigation: Establish linting rules or test patterns that require explicit undefined checks (e.g., expect(result).toBeDefined())
  Owner: engineering team
- Inconsistent application of the pattern across existing test suites may create confusion
  Mitigation: Conduct incremental refactoring of existing tests and document pattern in test style guide
  Owner: engineering team

## Implementation Notes

- When writing new tests, use Array.find() with explicit property comparisons: collection.find(item => item.property === 'value')
- Always follow find() calls with expect(result).toBeDefined() or similar assertions before accessing nested properties
- For diagnostic validation, use pattern: config.diagnostics.find((d: any) => d.code === 'DIAGNOSTIC_CODE')
- For agent registry lookups, use pattern: allAgents.find((agent) => agent.getIdentifier() === 'identifier')

## Continuation Context


Verify commands:
- grep -r '\.find(' tests/ | grep -E '(diagnostics\.find|allAgents\.find)' | wc -l
- grep -r '\.filter(.*\[0\]' tests/ | wc -l
- npm test -- --testPathPattern='(skills-mcp|unified-config)' --passWithNoTests

Accept when:
- Array.find() usage count in test files exceeds Array.filter()[0] pattern count by at least 3:1 ratio
- All diagnostic and agent lookup tests use Array.find() with explicit property comparisons
- Test suite passes with no undefined access errors from find() results

## Enforcement

- Verified by: Code review checklist requiring Array.find() pattern for entity lookups in test PRs
- Verified by: ESLint custom rule detecting filter()[0] pattern in test files
- Verified by: Test suite execution in CI pipeline validating no undefined access errors
- Violation handling: PR comments requesting refactoring to Array.find() pattern during code review
- Violation handling: ESLint warnings flagged in CI for filter()[0] usage in test files
- Violation handling: Test failures from undefined access treated as blocking issues requiring immediate fix
- Exception process: Document rationale in test file comments when alternative pattern is necessary
- Exception process: Obtain approval from tech lead when performance characteristics require different approach
- Exception process: Add ESLint disable comment with justification for specific cases where find() is not appropriate