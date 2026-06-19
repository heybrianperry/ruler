# Standardize Core Library Imports for Agent and Configuration Modules: Agent Implementations Import

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent implementations and configuration loading modules within the codebase.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, CopilotAgent, AiderAgent) that share common architectural patterns and dependencies
- Agent modules consistently import from core interfaces (IAgent, AgentsMdAgent) and utility modules (FileSystemUtils, ConfigLoader, agent-utils) establishing a dependency hierarchy
- Configuration loading and parsing operations are distributed across agent implementations and core modules, requiring consistent handling of TOML and JSON formats using @iarna/toml and native JSON.parse
- The revert-engine and UnifiedConfigLoader modules coordinate configuration management and agent lifecycle operations, requiring access to shared utilities and interfaces
- All agent implementations expose public contracts and apply configuration through the applyRulerConfig concurrency model pattern

## Problem Statement

Agent implementations and configuration modules must maintain consistent import patterns and dependency relationships to ensure predictable behavior, facilitate maintenance, and enable reliable configuration management across the system. Without standardized core library usage, modules risk divergent implementations, duplicated logic, and fragile coupling.

## Decision

1. MUST: Agent implementations MUST import from the IAgent interface to establish the base contract

## Policy Block

- MUST Agent implementations MUST import from the IAgent interface to establish the base contract

In scope:
- All agent implementation modules in src/agents/
- Configuration loading modules (UnifiedConfigLoader, ConfigLoader)
- Core utility modules that support agent operations (revert-engine, FileSystemUtils)
- Modules that parse or serialize TOML or JSON configuration data

Out of scope:
- Third-party library implementations
- Test fixtures and mock implementations
- Build and deployment scripts
- Documentation generation tools

Exceptions:
- EX-001: Legacy agent implementations require migration period before adopting standardized imports

## Rationale

- Evidence shows 8 files consistently importing from IAgent, AgentsMdAgent, and core utilities, establishing a proven architectural pattern with 90.75% confidence
- Standardizing on @iarna/toml and native JSON.parse ensures consistent parsing behavior across configuration loading paths and reduces dependency fragmentation
- The applyRulerConfig concurrency model appears across multiple agent implementations, indicating a shared coordination pattern that benefits from consistent core library usage
- Centralizing filesystem operations through FileSystemUtils and native Node.js modules reduces duplication and provides a single point of maintenance for file handling logic

## Consequences

Positive:
- Consistent import patterns reduce cognitive load for developers working across multiple agent implementations
- Shared core utilities enable centralized bug fixes and improvements that propagate to all consumers
- Standardized parsing libraries (native JSON.parse, @iarna/toml) ensure predictable configuration handling and reduce dependency conflicts
- Clear dependency hierarchy facilitates testing, mocking, and module isolation

Negative:
- Requires coordination when updating core interfaces or utilities to avoid breaking multiple agent implementations simultaneously
- Limits flexibility for agent implementations to choose alternative libraries or approaches for specific use cases
- Creates coupling between agent implementations and core module evolution, requiring synchronized updates
- May require refactoring existing code that uses non-standard import patterns or alternative libraries

## Alternatives

- Allow each agent implementation to independently choose parsing libraries and utility modules (rejected)
  Rejected because: Evidence shows consistent usage of @iarna/toml and native JSON.parse across 8 files, indicating existing convergence on these libraries. Divergent choices would fragment the dependency tree and complicate maintenance.
  When valid: When agent implementations have fundamentally different requirements that cannot be satisfied by shared libraries
- Create a single monolithic agent base class that encapsulates all core library imports (rejected)
  Rejected because: Evidence shows agents import specific utilities as needed (FileSystemUtils, agent-utils, merge) rather than inheriting everything from a single base. A monolithic base would force unnecessary dependencies.
  When valid: When all agents require identical core functionality with no variation
- Use dependency injection to provide core utilities rather than direct imports (deferred)
  When valid: When testing requirements or runtime configuration necessitate swapping implementations. Could be adopted incrementally without breaking existing import patterns.

## Risks

- Breaking changes to core interfaces (IAgent, AgentsMdAgent) cascade to all agent implementations simultaneously
  Mitigation: Implement versioned interfaces, use deprecation warnings, and maintain backward compatibility for at least one major version
  Owner: Core platform team
- The @iarna/toml library may become unmaintained or incompatible with future Node.js versions
  Mitigation: Monitor library maintenance status, evaluate alternative TOML parsers, and abstract parsing behind a utility function to enable library swapping
  Owner: Engineering team
- Standardized imports may not accommodate future agent types with significantly different requirements
  Mitigation: Document exception process for agents with justified alternative approaches, review import patterns quarterly for emerging needs
  Owner: Architecture review board

## Implementation Notes

- Audit existing agent implementations to identify any deviations from standard import patterns documented in this ADR
- Create shared TypeScript types for common configuration structures parsed from TOML and JSON to ensure type safety across modules
- Document the applyRulerConfig concurrency model pattern and its relationship to core library imports in architecture documentation
- Establish linting rules to enforce import patterns from approved core modules and flag usage of alternative libraries for the same functionality

## Continuation Context


Verify commands:
- grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l
- grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules
- grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"
- find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;

Accept when:
- All agent implementation files in src/agents/ import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library and JSON parsing uses native JSON.parse
- Agent modules export public contracts and no agent implementation uses alternative filesystem or parsing libraries without documented exception

## Enforcement

- Verified by: Static analysis via ESLint rules checking import statements against approved core module list
- Verified by: Code review checklist requiring verification of standard import patterns for new agent implementations
- Verified by: CI pipeline checks using grep patterns to detect non-standard library usage
- Violation handling: CI build fails if non-standard imports are detected without documented exception
- Violation handling: Code review blocks merge until imports conform to standard patterns or exception is approved
- Violation handling: Quarterly audit reports violations to architecture team for remediation planning
- Exception process: Submit exception request documenting specific requirement that cannot be met with standard imports
- Exception process: Architecture review board evaluates technical justification and approves/rejects within 5 business days
- Exception process: Approved exceptions are documented in code comments with ADR reference and expiration review date