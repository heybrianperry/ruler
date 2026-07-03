# Enforce JSON.parse and TOML Parsing with Error Handling for External Configuration Input: Configuration Loaders Validate

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent implementations and configuration loaders that process external input from files or API contracts.

## Context

- Agent implementations (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent) and UnifiedConfigLoader process external configuration files in JSON and TOML formats from filesystem paths
- These components expose public contracts (api.public.contracts) that accept untrusted or external input requiring validation before use
- Configuration data drives agent behavior through applyRulerConfig and loadUnifiedConfig functions, making input validation critical for system integrity
- The codebase uses JSON.parse and parseTOML (from @iarna/toml) as primary deserialization mechanisms for external data
- File system operations (fs.readFile, fs) retrieve raw content that must be parsed and validated before being consumed by application logic

## Problem Statement

External configuration files and API inputs in JSON and TOML formats must be parsed safely to prevent malformed data from causing runtime failures, security vulnerabilities, or undefined behavior in agent implementations and configuration loaders. Without consistent input validation patterns, each component may handle parsing errors differently, leading to inconsistent error handling and potential system instability.

## Decision

1. SHOULD: Configuration loaders SHOULD validate the structure and required fields of parsed data before returning to callers

## Policy Block

- SHOULD Configuration loaders SHOULD validate the structure and required fields of parsed data before returning to callers

In scope:
- Agent implementations: OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent
- Configuration loaders: UnifiedConfigLoader and similar components
- All functions that read external JSON or TOML files from filesystem
- Public API contracts that accept configuration data
- Functions that invoke applyRulerConfig or loadUnifiedConfig

Out of scope:
- Internal data structures already validated and in-memory
- Hardcoded configuration constants in source code
- Configuration generated programmatically without external input
- Test fixtures with known-valid data

Exceptions:
- EXC-001: Development or test environments where configuration is known to be valid and performance is critical

## Rationale

- Evidence shows consistent use of JSON.parse and parseTOML across 5 files (OpenCodeAgent.ts, MistralVibeAgent.ts, CodexCliAgent.ts, AmazonQCliAgent.ts, UnifiedConfigLoader.ts) indicating an established pattern for input validation
- The security.input_validation facet detection with 91.20% confidence across multiple agent implementations demonstrates this is a critical architectural concern for the system
- Agent public contracts (api.public.contracts) serve as integration points where external data enters the system, making input validation essential for security and reliability
- The pattern of reading from filesystem (fs.readFile, fs) followed by parsing establishes a clear boundary where validation must occur before data flows into application logic

## Consequences

Positive:
- Consistent error handling across all agent implementations and configuration loaders
- Early detection of malformed configuration files prevents runtime failures in downstream components
- Clear validation boundaries at public API contracts improve system security posture
- Standardized parsing approach using JSON.parse and @iarna/toml reduces implementation variance

Negative:
- Additional error handling code increases complexity in configuration loading paths
- Performance overhead from validation checks on every configuration load operation
- Strict validation may reject configurations that would have worked with lenient parsing
- Dependency on @iarna/toml library creates external dependency for TOML parsing

## Alternatives

- Use schema validation libraries (e.g., Zod, Joi) for comprehensive input validation beyond parsing (deferred)
  When valid: Consider when configuration complexity increases and structural validation becomes necessary beyond format parsing
- Allow lenient parsing without explicit error handling, relying on default JavaScript error propagation (rejected)
  Rejected because: Inconsistent error handling across components and poor error messages for users; does not meet security requirements for public API contracts
  When valid: Never appropriate for external input at public API boundaries
- Implement custom parsers for JSON and TOML with built-in validation (rejected)
  Rejected because: Unnecessary complexity and maintenance burden; standard libraries (JSON.parse, @iarna/toml) are well-tested and sufficient
  When valid: Only if standard parsers cannot meet specific security or performance requirements

## Risks

- Parsing errors may expose sensitive file paths or configuration details in error messages
  Mitigation: Sanitize error messages before logging or displaying to users; implement structured error handling that separates internal details from user-facing messages
  Owner: Security team and agent implementation owners
- Performance degradation if validation is performed repeatedly on the same configuration data
  Mitigation: Implement caching for parsed and validated configurations; validate once at load time rather than on every access
  Owner: Engineering team
- Inconsistent validation implementation across different agent types may create security gaps
  Mitigation: Create shared validation utilities or base classes that enforce consistent parsing patterns; include validation checks in code review process
  Owner: Architecture team and code reviewers

## Implementation Notes

- Wrap JSON.parse calls in try-catch blocks that catch SyntaxError and provide meaningful error context including file path
- Use @iarna/toml parseTOML function consistently across all TOML parsing operations; handle parsing exceptions with file context
- Create shared utility functions for common parsing patterns to reduce code duplication across agent implementations
- Document expected configuration schema for each agent type to aid in validation and error diagnosis
- Consider implementing a validation layer after parsing that checks for required fields and valid value ranges

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'
- grep -r 'parseTOML' src/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'
- grep -r 'api\.public\.contracts' src/agents/ | xargs grep -l 'JSON\.parse\|parseTOML' | xargs grep -L 'try.*catch' | wc -l | grep -q '^0$'

Accept when:
- All JSON.parse and parseTOML calls in agent implementations and configuration loaders are wrapped in error handling blocks
- No unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions
- Code review confirms consistent error handling patterns across all components in policy scope

## Enforcement

- Verified by: Automated grep-based verification commands in CI pipeline checking for unprotected parsing calls
- Verified by: Code review checklist requiring validation of external input handling
- Verified by: Static analysis tools configured to flag unhandled JSON.parse and parseTOML calls
- Violation handling: CI pipeline fails if verification commands detect unprotected parsing operations
- Violation handling: Code review blocks merge if input validation is missing from public API contracts
- Violation handling: Security review required for any exceptions to validation requirements
- Exception process: Submit exception request to tech lead with justification for skipping validation
- Exception process: Document exception in code comments with reference to approval
- Exception process: Exceptions reviewed quarterly to determine if they can be removed