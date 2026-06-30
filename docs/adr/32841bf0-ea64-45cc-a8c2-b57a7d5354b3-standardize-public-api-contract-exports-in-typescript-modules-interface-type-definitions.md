# Standardize Public API Contract Exports in TypeScript Modules: Interface Type Definitions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all TypeScript modules that define public API contracts within the codebase.

## Context

- The codebase contains 12 TypeScript modules across agent implementations and core utilities that export public API contracts including classes, interfaces, and utility functions
- Agent modules (OpenCodeAgent, ZedAgent, WarpAgent, ClineAgent, TraeAgent, KiroAgent, PiAgent) consistently export class-based contracts that implement or extend common interfaces (IAgent, AbstractAgent, AgentsMdAgent)
- Core utility modules (path-utils, SubagentsUtils, config-utils, SubagentsProcessor, agent-selection) export function-based contracts for configuration parsing, file system operations, and agent selection logic
- All modules use ES6 import/export syntax with explicit named exports, establishing clear module boundaries and dependency relationships
- The pattern demonstrates a consistent approach to defining public contracts across both domain-specific agent implementations and shared infrastructure utilities

## Problem Statement

Without standardized conventions for defining and exporting public API contracts in TypeScript modules, the codebase risks inconsistent module boundaries, unclear dependency relationships, and difficulty maintaining stable interfaces as the system evolves. The presence of 12 modules with varying contract types (classes, interfaces, functions) requires explicit guidance on how to structure exports, name contracts, and maintain backward compatibility.

## Decision

1. MUST: Interface and type definitions that form part of the public contract (e.g., ParsedFrontmatter, CopilotToolMapping) MUST be exported alongside their implementing classes or consuming functions

## Policy Block

- MUST Interface and type definitions that form part of the public contract (e.g., ParsedFrontmatter, CopilotToolMapping) MUST be exported alongside their implementing classes or consuming functions

In scope:
- All TypeScript modules in src/agents/ that implement agent contracts
- All TypeScript modules in src/core/ that provide shared utilities and infrastructure
- Interface definitions (IAgent, AbstractAgent) that define behavioral contracts
- Type definitions (ParsedFrontmatter, CopilotToolMapping) used across module boundaries
- Utility functions (parseFrontmatter, loadSubagentFile, mapRawAgentConfigs, resolveSelectedAgents) exposed for external consumption

Out of scope:
- Internal helper functions not intended for external use
- Private class methods and properties
- Test utilities and mock implementations
- Build-time scripts and tooling configuration
- Third-party library re-exports without added value

Exceptions:
- EXC-001: A module needs to export a default export for framework compatibility (e.g., Next.js page components, React components)
- EXC-002: Internal testing utilities need to access private implementation details for comprehensive test coverage

## Rationale

- The evidence shows 12 modules consistently using named exports with clear contract boundaries, indicating an established pattern that supports maintainability and explicit dependency management
- Class-based contracts (7 agent implementations) and function-based contracts (5 utility modules) demonstrate two distinct but complementary approaches to defining public APIs, both requiring standardized conventions
- Explicit named exports enable better tree-shaking, clearer import statements, and easier refactoring compared to default exports or mixed export strategies
- The pattern of exporting interfaces (IAgent), abstract classes (AbstractAgent), and concrete implementations together establishes a clear inheritance and composition hierarchy that benefits from standardization

## Consequences

Positive:
- Clear module boundaries improve code navigation, IDE support, and developer understanding of system architecture
- Explicit named exports enable better static analysis, tree-shaking, and bundle optimization in build tools
- Consistent naming conventions (PascalCase for classes, camelCase for functions) reduce cognitive load and improve code readability
- Stable public contracts enable independent module evolution and reduce coupling between components

Negative:
- Enforcing explicit export conventions requires additional discipline during code reviews and may slow initial development
- Maintaining backward compatibility for public contracts constrains refactoring options and may require deprecation cycles
- The distinction between public and private APIs requires careful design decisions and may lead to over-engineering in simple modules
- Multiple export patterns (class-based vs function-based) require developers to understand when to apply each convention

## Alternatives

- Use default exports for all modules to simplify import statements (rejected)
  Rejected because: Default exports obscure module contents, prevent tree-shaking, and make refactoring more difficult. The evidence shows 12 modules successfully using named exports, indicating the current pattern is working well.
  When valid: Only for framework-mandated scenarios (e.g., Next.js pages) where default exports are required
- Export all functions and classes without distinguishing public vs private contracts (rejected)
  Rejected because: Exposing internal implementation details increases coupling, makes breaking changes more likely, and violates encapsulation principles. Clear public contracts are essential for maintainability.
  When valid: Never; use internal/ directories or private naming conventions for implementation details
- Create barrel exports (index.ts files) that re-export all public contracts from subdirectories (deferred)
  Rejected because: Not rejected; this is a complementary pattern that could work alongside explicit named exports. Requires evaluation of build performance impact and maintenance overhead.
  When valid: When a directory contains multiple related modules that should be imported as a cohesive unit

## Risks

- Developers may inadvertently export internal implementation details, creating unintended public contracts that become difficult to change
  Mitigation: Implement automated linting rules that flag exports not matching naming conventions, and require explicit @public JSDoc tags for exported contracts
  Owner: Engineering team
- Breaking changes to public contracts may not be detected until runtime, causing integration failures across module boundaries
  Mitigation: Implement TypeScript strict mode, use API documentation tools (e.g., API Extractor), and maintain integration tests that exercise cross-module contracts
  Owner: Engineering team
- Inconsistent application of naming conventions (PascalCase vs camelCase) may lead to confusion and reduce the value of standardization
  Mitigation: Enforce naming conventions through ESLint rules, provide clear examples in documentation, and review exports during code review
  Owner: Engineering team

## Implementation Notes

- Use TypeScript's export keyword explicitly for each public contract rather than collecting exports at the end of the file
- Document public API contracts with JSDoc comments that describe purpose, parameters, return values, and usage examples
- Consider using TypeScript's export type syntax for type-only exports to improve build performance and clarify intent
- Organize module files with public exports at the top and private implementation details below, making the public contract immediately visible
- For agent implementations, ensure the exported class name matches the file name (e.g., OpenCodeAgent in OpenCodeAgent.ts) for consistency

## Continuation Context


Verify commands:
- grep -r "export default" src/agents/ src/core/ | grep -v ".test.ts" | grep -v ".spec.ts" # Should return no results
- grep -r "export class [A-Z]" src/agents/ src/core/ # Should show PascalCase class exports
- grep -r "export function [a-z]" src/agents/ src/core/ # Should show camelCase function exports
- npx ts-node -e "import * as agents from './src/agents'; import * as core from './src/core'; console.log('Imports successful')" # Should execute without errors

Accept when:
- All TypeScript modules in src/agents/ and src/core/ use explicit named exports with no default exports (except framework-mandated exceptions)
- Class exports follow PascalCase naming and function exports follow camelCase naming consistently across all modules
- Public API contracts are documented with JSDoc comments and can be imported successfully by dependent modules
- Automated linting passes with no violations of export naming conventions or unintended public contract exports

## Enforcement

- Verified by: ESLint rules enforcing named exports and naming conventions (no-default-export, naming-convention)
- Verified by: TypeScript compiler strict mode catching type mismatches across module boundaries
- Verified by: Code review checklist requiring verification of public contract documentation and naming
- Verified by: CI pipeline running verification commands to detect default exports and naming violations
- Violation handling: CI build fails if verification commands detect default exports or naming convention violations
- Violation handling: Pull requests blocked until ESLint and TypeScript compiler errors are resolved
- Violation handling: Code review feedback requires correction of undocumented public contracts or inconsistent naming
- Violation handling: Quarterly architecture reviews identify modules with unclear public contracts for refactoring
- Exception process: Developer documents the framework or technical requirement necessitating an exception (e.g., Next.js default export requirement)
- Exception process: Architecture review approves the exception and verifies that a named export is maintained alongside any required default export
- Exception process: Exception is documented in code comments with a reference to this ADR and the approval decision
- Exception process: Exceptions are tracked in a central registry and reviewed quarterly to determine if they can be eliminated