# Standardize Agent and Utility Module Exports as Public API Contracts: Agent Implementations Extend

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations and core utility modules that expose public interfaces.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, ZedAgent, WarpAgent, ClineAgent, TraeAgent, KiroAgent, PiAgent) that extend common base classes (AbstractAgent, AgentsMdAgent) and implement the IAgent interface
- Core utility modules (path-utils, SubagentsUtils, SubagentsProcessor, config-utils, agent-selection) export specific functions and types as public contracts for consumption by agents and other modules
- Agent implementations consistently import from core modules using relative paths ('./IAgent', '../core/FileSystemUtils', './AbstractAgent') establishing a dependency hierarchy
- Configuration and processing modules (SubagentsProcessor, ConfigLoader) coordinate agent discovery, selection, and propagation through exported functions like discoverSubagents, resolveSelectedAgents, and propagateSubagentsForClaude
- The pattern emerged across 13 files with 89.42% confidence, indicating a systematic approach to defining module boundaries and public interfaces rather than ad-hoc exports

## Problem Statement

Without standardized public API contracts for agent implementations and core utility modules, the codebase risks inconsistent interface definitions, unclear module boundaries, and fragile coupling between agents and shared utilities. This creates maintenance burden when refactoring, makes it difficult to understand what constitutes the stable public API versus internal implementation details, and complicates the addition of new agent types or utility functions.

## Decision

1. MAY: Agent implementations MAY extend the base agent classes with additional public methods specific to their integration requirements

## Policy Block

- MAY Agent implementations MAY extend the base agent classes with additional public methods specific to their integration requirements

In scope:
- All agent implementation files in src/agents/ directory
- All core utility modules in src/core/ directory that provide shared functionality
- Interface definitions (IAgent) and base classes (AbstractAgent, AgentsMdAgent)
- Type definitions and data structures shared across modules
- Configuration and processing modules that coordinate agent behavior

Out of scope:
- Internal helper functions within a module that are not exported
- Test files and test utilities
- Build configuration files (.prettierrc.js, tsconfig.json)
- Third-party library interfaces (path, fs/promises, js-yaml, @iarna/toml)

Exceptions:
- EXC-001: A module needs to export experimental or unstable APIs for testing purposes

## Rationale

- The evidence shows 13 files consistently exporting named classes and functions as public contracts, indicating this pattern provides clear module boundaries and stable interfaces for agent implementations
- Agent classes (OpenCodeAgent, ZedAgent, WarpAgent, ClineAgent, TraeAgent, KiroAgent, PiAgent) all follow the same export pattern, demonstrating that standardized contracts enable consistent extension of base agent functionality
- Core utility modules export specific functions (parseFrontmatter, validateFrontmatter, loadSubagentFile, isPathInsideOrEqual, discoverSubagents, resolveSelectedAgents) that are imported by multiple agents, proving that explicit public contracts reduce duplication and centralize shared logic
- The 89.42% confidence across 13 files indicates this is an established architectural pattern rather than an emerging one, justifying its formalization as a decision record

## Consequences

Positive:
- Clear module boundaries make it easier to understand what constitutes the public API versus internal implementation details
- Consistent export patterns across agent implementations simplify the addition of new agent types
- Centralized utility functions reduce code duplication and provide single points of maintenance for shared logic
- Explicit interface contracts (IAgent) enable polymorphic treatment of different agent implementations

Negative:
- Requires discipline to maintain clear boundaries between public and private module interfaces
- Changes to public contracts require coordination across multiple consuming modules
- May lead to larger public API surface area if not carefully curated
- Refactoring public interfaces requires careful deprecation planning to avoid breaking changes

## Alternatives

- Use a single monolithic agent class with configuration-driven behavior instead of multiple agent implementations with public contracts (rejected)
  Rejected because: Would reduce flexibility for agent-specific customization and make the codebase harder to extend with new agent types. The evidence shows 7 distinct agent implementations, indicating that polymorphic agent contracts are a core architectural requirement.
  When valid: Only valid for systems with a single agent type or where all agents share identical behavior
- Use barrel exports (index.ts files) to aggregate and re-export public APIs from each directory (deferred)
  Rejected because: Not rejected, but not currently implemented. Could improve import ergonomics but adds indirection.
  When valid: Valid if the number of exported modules grows significantly and import paths become unwieldy
- Define all public contracts in a separate @types package or contracts directory (rejected)
  Rejected because: Would separate interface definitions from implementations, making the codebase harder to navigate. The evidence shows interfaces (IAgent) and implementations colocated in the same directory structure.
  When valid: Valid for multi-package monorepos where contracts need to be shared across package boundaries

## Risks

- Public API surface area grows uncontrolled as new utility functions and agent methods are added without curation
  Mitigation: Establish API review process for new public exports. Document public API in module README files. Use @public JSDoc tags to explicitly mark public contracts.
  Owner: Engineering team / module owners
- Breaking changes to public contracts (IAgent interface, core utility function signatures) impact multiple agent implementations simultaneously
  Mitigation: Use semantic versioning for internal modules. Implement deprecation warnings before removing public APIs. Maintain changelog for public contract changes.
  Owner: Engineering team / lead architect
- Inconsistent import patterns emerge as new developers add modules without following established conventions
  Mitigation: Document import conventions in CONTRIBUTING.md. Use ESLint rules to enforce relative path patterns. Include import pattern checks in code review checklist.
  Owner: Engineering team / code reviewers

## Implementation Notes

- Use TypeScript export keyword to explicitly mark classes, functions, and types as public contracts (e.g., 'export class OpenCodeAgent implements IAgent')
- Organize agent implementations in src/agents/ directory with each agent in its own file, importing shared interfaces from './IAgent' and base classes from './AbstractAgent' or './AgentsMdAgent'
- Place shared utility functions in src/core/ directory with descriptive module names (path-utils.ts, SubagentsUtils.ts, config-utils.ts) and export specific functions rather than default exports
- Document public contracts with JSDoc comments including @public tag, parameter descriptions, and return types to clarify intended usage
- When adding new agent types, extend AbstractAgent or AgentsMdAgent base classes and implement required IAgent interface methods to maintain consistency with existing agents

## Continuation Context


Verify commands:
- grep -r 'export class.*Agent' src/agents/ | wc -l  # Should match number of agent implementations
- grep -r 'export.*function' src/core/ | grep -E '(parseFrontmatter|validateFrontmatter|loadSubagentFile|isPathInsideOrEqual|discoverSubagents|resolveSelectedAgents)'  # Verify core utility exports
- grep -r "from '\./IAgent'" src/agents/ | wc -l  # Verify agents import IAgent interface
- grep -r "from '\.\./core/" src/agents/ | wc -l  # Verify agents import from core utilities

Accept when:
- All agent implementation files export a named class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Core utility modules (path-utils, SubagentsUtils, SubagentsProcessor, config-utils, agent-selection) export documented public functions with clear names
- Import statements in agent files follow the established relative path conventions ('./IAgent', '../core/ModuleName')
- No internal implementation details or undocumented helper functions are exported from modules

## Enforcement

- Verified by: TypeScript compiler checks for interface implementation and type correctness
- Verified by: Code review checklist includes verification of public export patterns and import conventions
- Verified by: ESLint rules enforce relative import path patterns and prevent circular dependencies
- Verified by: Automated grep-based verification in CI pipeline checks for expected export patterns
- Violation handling: TypeScript compilation errors block merge if interface contracts are violated
- Violation handling: Code review feedback requests changes to align with export and import conventions
- Violation handling: CI pipeline fails if verification commands do not pass expected thresholds
- Violation handling: Documentation updates required for any changes to public API contracts
- Exception process: Developer documents rationale for deviation from standard export pattern in pull request description
- Exception process: Lead architect or module owner reviews exception request and approves if justified
- Exception process: Approved exceptions are documented with @experimental or @internal JSDoc tags
- Exception process: Exceptions are tracked in technical debt backlog for future resolution