# Standardize Core Library Module Detection and Integration: Production Code Not

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all modules and libraries integrated into the codebase.

## Context

- The codebase demonstrates consistent patterns of library and module integration across test files, CLI components, agent abstractions, and configuration files
- Pattern signature 352185f1ce9ffb39c86c964ce8959ec4 was detected in 6 files with 87.92% confidence, indicating a systematic approach to library management
- Core libraries are being used across multiple layers including testing infrastructure (jest.setup.js, test files), build configuration (eslint.config.mjs), CLI tooling (src/cli/index.ts), and agent abstractions (src/agents/AbstractAgent.ts)
- The detection spans both JavaScript and TypeScript files, suggesting a polyglot environment requiring consistent library integration patterns
- Integration tests and TOML format handling indicate the need for standardized external library usage patterns to maintain consistency across the codebase

## Problem Statement

Without standardized guidelines for core library detection and integration, the codebase risks inconsistent dependency management, duplicated functionality, incompatible library versions, and fragmented patterns across different modules. This can lead to maintenance overhead, testing complexity, and potential runtime conflicts when libraries are used differently across components.

## Decision

1. MUST_NOT: Production code MUST NOT directly depend on test-specific libraries or mock implementations

## Policy Block

- MUST_NOT Production code MUST NOT directly depend on test-specific libraries or mock implementations

In scope:
- All JavaScript and TypeScript source files in src/ directory
- Test files and test infrastructure (*.test.ts, *.test.js, jest.setup.js)
- Configuration files (eslint.config.mjs, tsconfig.json, package.json)
- CLI tools and command-line interfaces
- Agent abstractions and core architectural components
- Build and development tooling

Out of scope:
- Third-party vendor code not maintained by the team
- Generated code or auto-generated type definitions
- Legacy code marked for deprecation
- Experimental prototypes in isolated branches
- Documentation-only repositories without executable code

Exceptions:
- EXC-001: A specific module requires a different version of a core library due to compatibility constraints
- EXC-002: Test files need to import production code that transitively depends on test utilities for integration testing

## Rationale

- The pattern was detected across 6 files with 87.92% confidence, indicating this is an established architectural pattern worth codifying
- Consistent library integration reduces cognitive load for developers and makes the codebase more maintainable and predictable
- Standardizing library usage patterns enables automated detection and validation, catching integration issues early in the development cycle
- Clear separation between test, configuration, and production libraries prevents accidental dependencies and reduces bundle sizes

## Consequences

Positive:
- Improved consistency in how libraries are imported and used across the entire codebase
- Easier onboarding for new developers who can rely on predictable library integration patterns
- Automated detection pipelines can identify deviations and enforce standards through CI/CD
- Reduced risk of version conflicts and dependency hell through centralized library management
- Clear separation of concerns between test, build, and production dependencies

Negative:
- Additional overhead in documenting and maintaining the central library registry
- May require refactoring existing code that doesn't conform to the standardized patterns
- Could slow down rapid prototyping if architectural review is required for new libraries
- Potential friction when integrating libraries that don't fit the established patterns

## Alternatives

- Allow ad-hoc library integration without standardization, letting each team choose their own patterns (rejected)
  Rejected because: This approach leads to fragmentation, inconsistent patterns, and maintenance overhead as evidenced by the need for automated pattern detection
  When valid: Only appropriate for small, single-developer projects or early-stage prototypes
- Enforce strict monorepo with workspace-level dependency management and no local overrides (rejected)
  Rejected because: Too restrictive for polyglot environments and may prevent legitimate use cases where different modules need different library versions
  When valid: Valid for homogeneous codebases with uniform technology stacks and no legacy constraints
- Use automated dependency scanning tools without explicit architectural guidelines (rejected)
  Rejected because: Tools alone cannot enforce architectural intent or provide context for why certain patterns exist; explicit ADRs provide the necessary context
  When valid: Can be used as a complementary approach alongside this ADR for security scanning

## Risks

- Existing code may not comply with the standardized patterns, requiring significant refactoring effort
  Mitigation: Implement gradual migration strategy with automated detection to identify non-compliant code; prioritize high-impact areas first
  Owner: Engineering team with architecture review board oversight
- New library requirements may not fit the established patterns, causing delays or workarounds
  Mitigation: Establish clear exception process and regular review cycles to update patterns based on evolving needs
  Owner: Architecture review board
- Automated detection may produce false positives or miss legitimate pattern violations
  Mitigation: Continuously tune detection algorithms based on feedback; maintain human review for edge cases
  Owner: DevOps and tooling team

## Implementation Notes

- Create a central library registry document listing all approved core libraries, their purposes, and usage guidelines
- Configure ESLint or similar linting tools to enforce import patterns and detect violations of library usage rules
- Update CI/CD pipelines to run automated pattern detection and fail builds on critical violations
- Provide code templates and examples demonstrating correct library integration patterns for common scenarios
- Schedule quarterly reviews of the library registry to add new approved libraries and deprecate unused ones

## Continuation Context


Verify commands:
- grep -r "^import\|^const.*require" src/ tests/ | grep -v node_modules | sort | uniq -c | sort -rn
- npm ls --depth=0 2>&1 | grep -E 'UNMET|extraneous'
- find . -name '*.ts' -o -name '*.js' | xargs grep -l 'jest\|@testing-library' | grep -v -E '(test\.|spec\.|jest\.setup)'

Accept when:
- All core libraries are declared in package.json with appropriate version constraints and no unmet peer dependencies
- Import statements follow consistent patterns across the codebase with no mixed import styles within the same module type
- No production code files import test-specific libraries (verification command returns empty results)
- Automated pattern detection pipeline runs successfully and reports compliance above 95% threshold

## Enforcement

- Verified by: Automated CI/CD pipeline checks running pattern detection on every pull request
- Verified by: ESLint rules enforcing import patterns and dependency boundaries
- Verified by: Code review checklist requiring verification of library usage compliance
- Verified by: Quarterly architecture audits reviewing library integration patterns
- Violation handling: Critical violations (production code importing test libraries) block PR merges and require immediate remediation
- Violation handling: Major violations (inconsistent import patterns) generate warnings and must be addressed within one sprint
- Violation handling: Minor violations (missing documentation) are tracked as technical debt and prioritized in backlog
- Violation handling: Repeated violations trigger architectural review to determine if patterns need updating or additional training is needed
- Exception process: Developer submits exception request to architecture review board with detailed justification and impact analysis
- Exception process: Review board evaluates request within 3 business days, considering alternatives and long-term implications
- Exception process: Approved exceptions are documented in ADR amendments with expiration dates and review triggers
- Exception process: All exceptions are reviewed quarterly to determine if they should become permanent patterns or be remediated