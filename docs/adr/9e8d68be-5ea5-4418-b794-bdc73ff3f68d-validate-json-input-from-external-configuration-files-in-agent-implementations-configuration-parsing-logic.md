# Validate JSON Input from External Configuration Files in Agent Implementations: Configuration Parsing Logic

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent implementations that consume external configuration files.

## Context

- Agent implementations (OpenCodeAgent, ZedAgent) consume external JSON configuration files from the filesystem to initialize their behavior and configuration state
- These configuration files represent untrusted or semi-trusted input that crosses a trust boundary between the filesystem and the application runtime
- JSON.parse operations are performed on file content read via fs/promises and fs modules without explicit validation or error handling visible in the IR evidence
- The pattern appears in multiple agent implementations (2 files), suggesting a common architectural approach to configuration management across the agent subsystem
- FileSystemUtils and path modules are used alongside the parsing operations, indicating filesystem-based configuration loading is a standard integration pattern

## Problem Statement

Agent implementations must safely consume JSON configuration from external files without introducing security vulnerabilities, runtime errors, or undefined behavior when configuration files are malformed, missing, or contain malicious content. The current pattern shows direct JSON.parse usage without visible validation, creating potential attack vectors and reliability issues at the public API boundary where agents integrate with external configuration sources.

## Decision

1. SHOULD: Configuration parsing logic SHOULD be centralized in a shared utility module to ensure consistent validation across all agent implementations

## Policy Block

- SHOULD Configuration parsing logic SHOULD be centralized in a shared utility module to ensure consistent validation across all agent implementations

In scope:
- All agent implementations that extend IAgent interface
- Configuration files loaded via fs, fs/promises, or FileSystemUtils modules
- JSON.parse operations on external file content
- MCP configuration files (mcpPath) and similar external configuration sources

Out of scope:
- Internal JSON serialization/deserialization within the application runtime
- Configuration hardcoded in source code
- Environment variables or command-line arguments
- Database-stored configuration

Exceptions:
- EXC-001: Configuration files are generated and validated by the same process immediately before parsing

## Rationale

- The IR evidence shows JSON.parse operations on file content in 2 agent implementations (OpenCodeAgent.ts, ZedAgent.ts) with 91.20% confidence, indicating a consistent pattern that requires standardization
- External configuration files represent a trust boundary where malicious or malformed input can compromise agent behavior, making input validation a critical security control
- The presence of FileSystemUtils and path modules alongside parsing operations suggests filesystem-based configuration is an established integration pattern that should be hardened
- Centralizing validation logic reduces code duplication and ensures consistent security posture across all agent implementations that follow this pattern

## Consequences

Positive:
- Prevents runtime errors and crashes from malformed JSON configuration files
- Reduces attack surface by validating untrusted input at the trust boundary
- Improves debugging experience with clear validation error messages
- Enables safe evolution of configuration schemas with explicit validation rules

Negative:
- Adds initialization overhead for schema validation on every agent startup
- Requires additional dependencies for schema validation libraries
- Increases code complexity in agent initialization paths
- May break existing agents if current configuration files do not conform to new validation schemas

## Alternatives

- Use TypeScript type guards without schema validation libraries (rejected)
  Rejected because: Type guards only provide compile-time safety and do not validate runtime data from external files, leaving the security vulnerability unaddressed
  When valid: Only for internal type narrowing after validation has already occurred
- Implement custom validation functions for each agent configuration (rejected)
  Rejected because: Duplicates validation logic across multiple agent implementations, increasing maintenance burden and risk of inconsistent security controls
  When valid: When agent-specific validation requirements cannot be expressed in a schema language
- Move all configuration to environment variables or command-line arguments (deferred)
  Rejected because: Would require significant refactoring of existing agent architecture and may not be suitable for complex nested configuration structures
  When valid: For simple scalar configuration values or when filesystem access is not available

## Risks

- Existing agent configurations may fail validation when schema validation is introduced, breaking production deployments
  Mitigation: Implement validation in warning mode first, log violations without failing, then enforce after configuration files are updated
  Owner: Engineering team
- Schema validation library vulnerabilities could introduce new attack vectors
  Mitigation: Use well-maintained validation libraries (e.g., Zod, Ajv) with active security monitoring and regular dependency updates
  Owner: Security team
- Performance degradation during agent initialization if validation is computationally expensive
  Mitigation: Profile validation performance and cache parsed/validated configurations when appropriate
  Owner: Engineering team

## Implementation Notes

- Consider using Zod or Ajv for schema validation - both provide TypeScript integration and comprehensive validation capabilities
- Create a shared ConfigurationLoader utility in FileSystemUtils that encapsulates file reading, parsing, and validation
- Define configuration schemas as TypeScript types with corresponding runtime validators to maintain type safety
- Add unit tests that verify validation behavior with malformed, missing, and malicious configuration files

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' # Should return no matches after implementation
- grep -r 'readFile.*JSON\.parse' src/agents/ # Should be replaced with validated configuration loading
- npm test -- --grep 'configuration validation' # Should pass all validation test cases

Accept when:
- All JSON.parse operations on external file content are wrapped in try-catch blocks with appropriate error handling
- Schema validation is applied to all configuration objects before they are consumed by agent implementations
- Unit tests demonstrate that malformed JSON and invalid configuration schemas are rejected with descriptive error messages

## Enforcement

- Verified by: Static analysis tools (ESLint rules) to detect unvalidated JSON.parse usage
- Verified by: Code review checklist requiring validation for all external configuration loading
- Verified by: Integration tests that verify validation behavior in CI pipeline
- Violation handling: CI pipeline fails if static analysis detects unvalidated JSON.parse on external file content
- Violation handling: Code review blocks merge if validation is missing from new agent implementations
- Violation handling: Security audit flags unvalidated external input as high-priority vulnerability
- Exception process: Submit exception request to tech lead with justification for why validation cannot be applied
- Exception process: Document the exception in code comments with reference to approval
- Exception process: Add compensating security controls if validation is waived