# Standardize Agent Configuration Loading with Input Validation and Contract Exports: Agent Implementations Extend

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations and configuration loaders within the system.

## Context

- Agent implementations (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent) require loading external configuration from JSON and TOML files with varying schemas
- Configuration files contain sensitive data including API endpoints, credentials, and runtime parameters that must be validated before use
- The UnifiedConfigLoader serves as a central configuration loading mechanism that must handle multiple formats (TOML, JSON) with consistent validation
- Each agent exports a public contract interface (IAgent implementations) that must be consistently exposed across the codebase
- The applyRulerConfig concurrency pattern is applied uniformly across agent implementations, requiring validated configuration input

## Problem Statement

Without standardized input validation during configuration loading and consistent public contract exports, agent implementations are vulnerable to malformed configuration injection, runtime errors from invalid data, and inconsistent API surface exposure that complicates integration and testing.

## Decision

1. MAY: Agent implementations MAY extend the base IAgent interface with additional agent-specific methods

## Policy Block

- MAY Agent implementations MAY extend the base IAgent interface with additional agent-specific methods

In scope:
- All agent implementations in src/agents/ directory
- Configuration loading logic in UnifiedConfigLoader and related utilities
- JSON and TOML configuration file parsing operations
- Public API contract exports for agent interfaces

Out of scope:
- Third-party library configuration validation (handled by library maintainers)
- Runtime type checking beyond initial parse validation
- Configuration schema evolution and migration logic
- Agent-specific business logic unrelated to configuration loading

Exceptions:
- EXC-001: Legacy agent implementations during migration period require gradual adoption of validation patterns

## Rationale

- The pattern appears consistently across 5 agent implementations with 91.20% confidence, indicating an established architectural convention
- Input validation through JSON.parse() and parseTOML() provides immediate detection of malformed configuration before runtime errors propagate
- Standardized public contract exports (IAgent interface) enable consistent agent discovery, testing, and integration patterns
- The applyRulerConfig concurrency pattern requires validated input to prevent race conditions and undefined behavior from malformed data

## Consequences

Positive:
- Configuration parsing errors are caught early with clear error messages, reducing debugging time and production incidents
- Consistent IAgent contract exports enable polymorphic agent handling and simplified testing infrastructure
- Input validation prevents injection attacks and malformed data from reaching business logic
- Standardized configuration loading patterns reduce cognitive load for developers working across multiple agent implementations

Negative:
- Additional parsing overhead on configuration load, though typically negligible for configuration file sizes
- Strict validation may reject valid but unconventional configuration formats, requiring schema updates
- Developers must maintain validation logic alongside configuration schema evolution
- Exception handling complexity increases with multiple validation points across agent implementations

## Alternatives

- Use runtime schema validation libraries (e.g., Zod, Joi) instead of native JSON.parse/parseTOML validation (rejected)
  Rejected because: Adds external dependencies and complexity when native parsing provides sufficient validation for current needs; schema libraries introduce breaking changes and maintenance burden
  When valid: When configuration schemas become complex with nested validation rules, conditional fields, or require detailed error reporting beyond parse errors
- Centralize all configuration loading in UnifiedConfigLoader without agent-specific validation (rejected)
  Rejected because: Removes agent-specific validation context and error handling; agents lose control over their configuration validation requirements
  When valid: When all agents converge on identical configuration schemas and validation requirements
- Defer validation to runtime usage points rather than at configuration load time (rejected)
  Rejected because: Allows invalid configuration to propagate through the system, causing late-stage failures that are harder to diagnose and recover from
  When valid: Never recommended for security-sensitive configuration; only acceptable for optional feature flags with safe defaults

## Risks

- Configuration parsing errors may expose sensitive file paths or configuration structure in error messages
  Mitigation: Sanitize error messages to remove absolute paths and sensitive values; log full details securely for debugging
  Owner: Security team and agent implementation owners
- Inconsistent validation across agent implementations may create security gaps where some agents are more vulnerable
  Mitigation: Implement shared validation utilities and enforce through code review and automated testing; document validation requirements in IAgent interface
  Owner: Engineering team and architecture review board
- Breaking changes in @iarna/toml or native JSON parsing behavior could affect validation reliability
  Mitigation: Pin dependency versions, monitor for security advisories, and maintain comprehensive test coverage for parsing edge cases
  Owner: Platform team and dependency management

## Implementation Notes

- Wrap all JSON.parse() and parseTOML() calls in try-catch blocks with agent-specific error context
- Export agent classes as named exports matching the class name (e.g., export class OpenCodeAgent implements IAgent)
- Validate configuration immediately after parsing and before passing to applyRulerConfig or other business logic
- Document expected configuration schema in agent implementation file headers or adjacent schema files
- Use FileSystemUtils for consistent file reading operations before parsing configuration content

## Continuation Context


Verify commands:
- grep -r 'JSON.parse\|parseTOML' src/agents/ src/core/UnifiedConfigLoader.ts | grep -v 'try\|catch' && echo 'FAIL: Unprotected parsing found' || echo 'PASS: All parsing is protected'
- grep -r 'export class.*Agent.*implements IAgent' src/agents/*.ts | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Agent contracts exported' || echo 'FAIL: Missing agent exports'
- grep -r 'applyRulerConfig' src/agents/ | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Concurrency pattern applied' || echo 'FAIL: Missing applyRulerConfig'

Accept when:
- All agent implementations export IAgent-compliant public contracts with consistent naming
- Configuration parsing operations use JSON.parse() or parseTOML() with error handling before data usage
- No unvalidated configuration data reaches applyRulerConfig or business logic execution paths
- Code review confirms validation occurs at configuration load boundaries for all agents

## Enforcement

- Verified by: Automated grep-based verification in CI pipeline checking for unprotected parsing operations
- Verified by: Code review checklist requiring validation of configuration loading patterns
- Verified by: Static analysis tools scanning for JSON.parse/parseTOML usage without error handling
- Verified by: Integration tests verifying agent contract exports and configuration validation behavior
- Violation handling: CI pipeline fails if unprotected parsing operations are detected in agent implementations
- Violation handling: Code review blocks merge if configuration validation is missing or incomplete
- Violation handling: Security team notification for violations in production code paths
- Violation handling: Mandatory remediation within one sprint for identified validation gaps
- Exception process: Submit exception request to security team with justification and risk assessment
- Exception process: Architecture review board evaluates exception against security requirements
- Exception process: Approved exceptions must document compensating controls and remediation timeline
- Exception process: Exceptions reviewed quarterly and automatically expire after 6 months without renewal