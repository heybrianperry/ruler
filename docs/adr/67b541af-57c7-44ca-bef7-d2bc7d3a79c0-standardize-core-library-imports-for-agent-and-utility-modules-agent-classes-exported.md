# Standardize Core Library Imports for Agent and Utility Modules: Agent Classes Exported

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations and core utility modules within the codebase.

## Context

- The codebase contains 12 agent implementations (OpenCodeAgent, ZedAgent, WarpAgent, ClineAgent, TraeAgent, KiroAgent, PiAgent) and core utility modules that consistently import Node.js core libraries ('path', 'fs', 'fs/promises') and internal interfaces ('./IAgent', './AbstractAgent')
- Agent modules expose public contracts through exported classes and functions, establishing a consistent API surface for agent instantiation and configuration
- Core utility modules (path-utils, SubagentsUtils, SubagentsProcessor, config-utils, agent-selection) provide shared functionality for file system operations, YAML/TOML parsing, and agent discovery
- The pattern demonstrates a separation between agent implementations (which extend base abstractions) and utility modules (which provide cross-cutting concerns like configuration loading and subagent discovery)
- Security-sensitive operations such as JSON.parse for configuration files and input validation are consistently applied across agent implementations

## Problem Statement

Agent implementations and core utility modules require a consistent approach to importing Node.js core libraries and internal interfaces to maintain API stability, enable predictable file system operations, and ensure uniform configuration handling across the agent ecosystem.

## Decision

1. MUST: Agent classes MUST be exported as public contracts to enable external instantiation and configuration

## Policy Block

- MUST Agent classes MUST be exported as public contracts to enable external instantiation and configuration

In scope:
- All agent implementation modules in src/agents/ directory
- All core utility modules in src/core/ directory
- Configuration loading and parsing logic
- File system operations for agent discovery and subagent propagation
- Public API contracts exposed through exported classes and functions

Out of scope:
- Third-party library internal implementations
- Test fixtures and mock implementations
- Build-time tooling and scripts
- Runtime-only dynamic imports for optional features

Exceptions:
- EXC-001: Agent implementations require platform-specific file system operations not available in Node.js core

## Rationale

- The evidence shows 12 files consistently importing Node.js core libraries ('path', 'fs', 'fs/promises') and internal interfaces ('./IAgent', './AbstractAgent'), demonstrating an established pattern with 90.20% confidence
- Agent implementations uniformly expose public contracts through exported classes (OpenCodeAgent, ZedAgent, WarpAgent, ClineAgent, TraeAgent, KiroAgent, PiAgent), establishing a stable API surface for external consumers
- Core utility modules provide shared functionality (parseFrontmatter, validateFrontmatter, loadSubagentFile, discoverSubagents) that enables consistent configuration handling and agent discovery across the ecosystem
- Security input validation through JSON.parse is consistently applied in agent configuration loading (OpenCodeAgent, ZedAgent), establishing a pattern for safe configuration parsing

## Consequences

Positive:
- Consistent import patterns reduce cognitive load for developers working across multiple agent implementations
- Standardized use of Node.js core libraries ensures compatibility and reduces external dependency risk
- Uniform public contract exports enable predictable agent instantiation and configuration by external consumers
- Shared utility modules eliminate code duplication for common operations like file system access and configuration parsing

Negative:
- Tight coupling to Node.js core libraries may complicate future migration to alternative runtimes (Deno, Bun)
- Standardized import patterns may constrain agent implementations that require specialized file system or configuration handling
- Dependency on internal interfaces ('./IAgent', './AbstractAgent') creates internal coupling that requires coordinated changes across all agent implementations

## Alternatives

- Use a unified file system abstraction layer instead of direct Node.js core imports (rejected)
  Rejected because: Adds unnecessary abstraction overhead when Node.js core libraries are sufficient and well-tested; the evidence shows direct imports work effectively across 12 agent implementations
  When valid: When targeting multiple JavaScript runtimes (Node.js, Deno, Bun) or when file system operations require advanced features not available in Node.js core
- Allow each agent implementation to choose its own import strategy and libraries (rejected)
  Rejected because: Creates inconsistency across the agent ecosystem and increases maintenance burden; the evidence shows a consistent pattern that should be preserved
  When valid: When agent implementations have fundamentally different requirements that cannot be satisfied by a common approach
- Consolidate all agent implementations into a single polymorphic agent class (rejected)
  Rejected because: Reduces flexibility and violates separation of concerns; the evidence shows distinct agent implementations (OpenCodeAgent, ZedAgent, WarpAgent, etc.) serve different purposes
  When valid: When agent behavior differences are minimal and can be expressed through configuration rather than separate implementations

## Risks

- Future Node.js core API changes may require coordinated updates across all 12 agent implementations
  Mitigation: Introduce shared wrapper functions in FileSystemUtils for commonly used Node.js core APIs to centralize future migration points
  Owner: Engineering team
- New agent implementations may deviate from established import patterns without automated enforcement
  Mitigation: Implement linting rules and code review checklists to verify compliance with import standards
  Owner: Engineering team
- Security vulnerabilities in JSON.parse usage may affect multiple agent implementations simultaneously
  Mitigation: Centralize configuration parsing in a shared utility with comprehensive error handling and validation; conduct security review of all JSON.parse call sites
  Owner: Security team

## Implementation Notes

- New agent implementations should extend AbstractAgent or AgentsMdAgent base classes to inherit standard import patterns and interface compliance
- Use 'fs/promises' instead of callback-based 'fs' for new code to enable async/await patterns and improve error handling
- Centralize configuration parsing logic in core utility modules (config-utils, SubagentsUtils) rather than duplicating JSON.parse calls in individual agents
- Document public contract exports in module headers to clarify API surface for external consumers

## Continuation Context


Verify commands:
- grep -r "import.*from 'path'" src/agents/ src/core/ | wc -l
- grep -r "import.*from 'fs" src/agents/ src/core/ | wc -l
- grep -r "export.*class.*Agent" src/agents/ | wc -l
- grep -r "JSON.parse" src/agents/ src/core/ | grep -v "try" | wc -l

Accept when:
- All agent implementations in src/agents/ import 'path' from Node.js core
- All agent implementations export their class as a public contract
- All JSON.parse calls are wrapped in try-catch blocks or error handling logic
- Core utility modules export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents)

## Enforcement

- Verified by: ESLint rules enforcing Node.js core import patterns for 'path', 'fs', and 'fs/promises'
- Verified by: Code review checklist verifying agent implementations extend AbstractAgent or implement IAgent
- Verified by: CI pipeline checks for exported public contracts in agent modules
- Verified by: Static analysis scanning for unprotected JSON.parse calls
- Violation handling: CI build fails if agent implementations do not import required Node.js core libraries
- Violation handling: Code review blocks merge if public contracts are not properly exported
- Violation handling: Security scan flags unprotected JSON.parse calls for manual review
- Violation handling: Linting errors must be resolved before merge to main branch
- Exception process: Submit architecture review request documenting why standard import patterns cannot be used
- Exception process: Provide alternative approach with equivalent safety and consistency guarantees
- Exception process: Obtain approval from two senior engineers and document exception in module header
- Exception process: Add exception to policy_exceptions list with clear allowed_when conditions