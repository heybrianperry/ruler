# Standardize Agent Configuration Parsing with Input Validation: Agent Public Contracts

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent implementations and configuration processing modules within the codebase.

## Context

- The codebase implements multiple agent types (CodexCliAgent, ZedAgent, RooCodeAgent) that require configuration loading from external sources including TOML and JSON files
- Agent configuration files are parsed using parseTOML and JSON.parse operations on file system content, creating potential security vulnerabilities if input validation is insufficient
- A centralized apply-engine module coordinates configuration application across agents through processHierarchicalConfigurations and processSingleConfiguration functions
- The system uses applyRulerConfig as a concurrency coordination point for applying configurations to agent instances
- Configuration loading involves nested directory traversal (findRulerDirectories, loadNestedConfigurations) and hierarchical configuration merging (createHierarchicalConfiguration)

## Problem Statement

Agent configuration parsing from external TOML and JSON files introduces security risks through potential injection attacks, malformed data handling, and unvalidated input processing. Without standardized input validation at parse boundaries, the system is vulnerable to configuration-based attacks that could compromise agent behavior or system integrity.

## Decision

1. MUST: Agent public contracts (CodexCliAgent, ZedAgent, RooCodeAgent) MUST define explicit configuration schemas that specify allowed fields, types, and value constraints

## Policy Block

- MUST Agent public contracts (CodexCliAgent, ZedAgent, RooCodeAgent) MUST define explicit configuration schemas that specify allowed fields, types, and value constraints

In scope:
- All agent implementations in src/agents/ directory (CodexCliAgent.ts, ZedAgent.ts, RooCodeAgent.ts)
- Configuration processing in src/core/apply-engine.ts
- TOML and JSON parsing operations using @iarna/toml and JSON.parse
- File system operations for configuration loading using fs module
- Public API contracts exposed by agent classes

Out of scope:
- Runtime data validation unrelated to configuration parsing
- Network input validation from external APIs
- User input validation from command-line interfaces
- Database query parameter validation

Exceptions:
- EXC-001: Legacy configuration files require backward compatibility during migration period

## Rationale

- The evidence shows consistent use of parseTOML and JSON.parse across four agent-related files, indicating a systematic pattern of configuration parsing that requires standardized security controls
- The presence of applyRulerConfig as a concurrency coordination point demonstrates that configuration validation must occur before concurrent application to prevent race conditions with invalid data
- Hierarchical configuration loading (loadNestedConfigurations, createHierarchicalConfiguration) increases attack surface through multiple parse points, requiring validation at each boundary
- Public API contracts (CodexCliAgent, ZedAgent, RooCodeAgent) serve as trust boundaries where external configuration data enters the system, making them critical validation points

## Consequences

Positive:
- Reduces risk of configuration-based injection attacks and malformed data causing system failures
- Establishes clear security boundaries at configuration parse points across all agent implementations
- Improves system reliability by catching configuration errors early in the loading process
- Creates consistent validation patterns that simplify security auditing and code review

Negative:
- Adds validation overhead to configuration loading operations, potentially impacting startup time
- Requires additional development effort to define and implement configuration schemas for each agent type
- May break existing configurations that rely on undocumented or loosely-typed fields
- Increases code complexity in configuration parsing modules with additional validation logic

## Alternatives

- Implement validation only at the apply-engine level without per-agent validation (rejected)
  Rejected because: Centralized validation alone cannot enforce agent-specific schema constraints and delays error detection until after parsing, increasing attack surface
  When valid: When all agents share identical configuration schemas with no agent-specific requirements
- Use runtime type checking libraries (e.g., zod, io-ts) for automatic schema validation (deferred)
  Rejected because: Not rejected; deferred pending evaluation of library integration costs and bundle size impact
  When valid: When type safety benefits outweigh dependency and bundle size costs, particularly for complex nested configurations
- Replace TOML and JSON with a DSL that enforces validation at parse time (rejected)
  Rejected because: Introduces significant migration costs and reduces interoperability with standard tooling that expects TOML/JSON formats
  When valid: For new projects without existing configuration files or when configuration complexity justifies custom DSL investment

## Risks

- Overly strict validation may reject valid legacy configurations, breaking existing deployments during rollout
  Mitigation: Implement validation in warning mode initially, collect telemetry on validation failures, then enforce after migration period
  Owner: Engineering team with security team oversight
- Performance degradation from validation overhead may impact systems with frequent configuration reloading
  Mitigation: Cache validated configurations, implement lazy validation for non-critical paths, and benchmark validation performance
  Owner: Engineering team
- Incomplete schema definitions may leave validation gaps that attackers can exploit
  Mitigation: Conduct security review of all configuration schemas, implement deny-by-default for unknown fields, and perform penetration testing
  Owner: Security team with engineering support

## Implementation Notes

- Start by defining TypeScript interfaces for RulerConfiguration and HierarchicalRulerConfiguration that serve as validation schemas
- Wrap all parseTOML and JSON.parse calls in try-catch blocks with specific error types for parse failures vs validation failures
- Implement a shared validation utility in src/core/ that can be reused across agent implementations to ensure consistency
- Add validation checks in applyConfigurationsToAgents before configuration application to serve as a final security gate
- Consider using TypeScript's type guards or assertion functions to provide compile-time and runtime type safety

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'
- grep -r 'export.*Agent' src/agents/*.ts | xargs -I {} sh -c 'grep -l "interface.*Config" $(dirname {})/$(basename {} .ts).ts'
- find src/agents src/core -name '*.ts' -exec grep -l 'applyRulerConfig\|applyConfigurationsToAgents' {} \; | xargs grep -l 'validate\|schema'

Accept when:
- All parseTOML and JSON.parse operations in agent files are wrapped in error handling blocks
- Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) has an associated configuration interface or schema definition
- The apply-engine module includes validation logic before applying configurations to agents

## Enforcement

- Verified by: Automated static analysis scanning for unprotected parse operations in CI pipeline
- Verified by: Code review checklist requiring validation logic for any new configuration parsing code
- Verified by: Security-focused unit tests verifying rejection of malformed and malicious configuration inputs
- Violation handling: CI pipeline fails if unprotected parse operations are detected in agent or configuration modules
- Violation handling: Pull requests adding configuration parsing without validation are blocked until validation is added
- Violation handling: Security team is notified of validation bypasses discovered in production code
- Exception process: Developer submits exception request documenting why validation cannot be applied and proposed compensating controls
- Exception process: Security team reviews exception request and assesses risk level
- Exception process: Architecture team approves exception with documented rationale and time-bound review period
- Exception process: Approved exceptions are tracked in security exception registry with quarterly review