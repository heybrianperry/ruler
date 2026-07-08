# Validate JSON Input with Try-Catch Parsing in Integration Components: Duplicate Detection Mechanisms

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all integration components that parse external JSON configuration or data.

## Context

- The FirebenderAgent component integrates with external configuration files (firebender.json) that require parsing untrusted or potentially malformed JSON content
- Integration components must handle file system operations ('fs', 'path') and parse configuration data without crashing on invalid input
- The codebase uses Node.js core libraries for file operations and JSON parsing, requiring defensive error handling at integration boundaries
- Configuration loading operations (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) coordinate multiple file system and parsing steps that can fail independently
- The agent architecture exposes public contracts (FirebenderAgent) that must remain stable despite external configuration errors

## Problem Statement

Integration components that parse external JSON configuration files face reliability risks when encountering malformed input, missing files, or corrupted data. Without defensive parsing and error handling, these components can crash or propagate invalid state, compromising system stability and violating the contract expectations of dependent modules.

## Decision

1. SHOULD: Duplicate detection mechanisms (e.g., seen.add(key)) SHOULD be used to prevent processing the same configuration multiple times

## Policy Block

- SHOULD Duplicate detection mechanisms (e.g., seen.add(key)) SHOULD be used to prevent processing the same configuration multiple times

In scope:
- All JSON parsing operations on external configuration files (e.g., firebender.json)
- File system read operations using Node.js 'fs' and 'path' modules in integration components
- Configuration loading functions: loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration
- Public API contracts exposed by integration agents (e.g., FirebenderAgent)

Out of scope:
- JSON parsing of trusted internal data structures already validated
- Test fixtures and mock data in test suites
- JSON serialization (JSON.stringify) operations
- Non-JSON configuration formats (YAML, TOML, XML)

## Rationale

- The evidence shows JSON.parse(existingContent) operations combined with error logging patterns (console.warn with 'Failed to read/parse existing firebender.json'), indicating defensive parsing is already practiced in at least one integration component
- The presence of multiple configuration coordination functions (applyRulerConfig, loadExistingConfig, saveConfig, handleMcpConfiguration) suggests a pattern of complex configuration management that requires consistent error handling
- The use of Node.js core modules ('fs', 'path') and FileSystemUtils indicates file system integration boundaries where external input validation is critical
- The public contract exposure (FirebenderAgent) requires stability guarantees that can only be maintained through defensive parsing at integration boundaries

## Consequences

Positive:
- Integration components remain stable and operational even when external configuration files are malformed or corrupted
- Error logging provides clear diagnostic information for troubleshooting configuration issues without requiring debugger attachment
- Public API contracts maintain stability guarantees, preventing cascading failures to dependent modules
- Defensive parsing enables graceful degradation with fallback configurations rather than complete system failure

Negative:
- Try-catch blocks add code verbosity and nesting depth to configuration loading logic
- Silent error handling with fallbacks may mask configuration problems that should be surfaced to users
- Logging to console.warn may not integrate with structured logging systems or observability platforms
- Duplicate detection mechanisms (seen.add) add memory overhead for tracking processed configurations

## Alternatives

- Use JSON schema validation libraries (e.g., ajv, zod) to validate structure before parsing (rejected)
  Rejected because: Schema validation adds dependency weight and does not eliminate the need for try-catch around JSON.parse itself, which can still throw on malformed JSON syntax
  When valid: When configuration structure validation is needed beyond syntax checking, or when generating TypeScript types from schemas
- Fail fast and crash the process on any configuration parse error (rejected)
  Rejected because: Violates stability requirements for public API contracts and prevents graceful degradation with fallback configurations
  When valid: In initialization-only contexts where invalid configuration should prevent service startup entirely
- Use a safe JSON parser library that returns Result types instead of throwing (deferred)
  Rejected because: Not rejected, but deferred pending evaluation of Result-type libraries compatible with Node.js ecosystem
  When valid: When adopting functional error handling patterns across the codebase or when try-catch verbosity becomes a maintenance burden

## Risks

- Inconsistent error handling across integration components if pattern is not uniformly applied
  Mitigation: Establish linting rules or code review checklists to verify try-catch wrapping of JSON.parse on external input
  Owner: engineering team
- Silent failures with fallback configurations may hide critical configuration errors from operators
  Mitigation: Ensure all parse failures are logged at appropriate severity levels and consider metrics/alerting for parse failure rates
  Owner: engineering team
- Console.warn logging may not be captured in production observability systems
  Mitigation: Migrate to structured logging framework that integrates with production log aggregation and monitoring
  Owner: platform team

## Implementation Notes

- Wrap all JSON.parse() calls on external input with try-catch blocks, logging errors with file path and error message context
- Use FileSystemUtils helper module to centralize file read error handling and reduce duplication across integration components
- Implement duplicate detection with Set-based tracking (seen.add(key)) for configuration keys that may be processed multiple times
- Provide sensible fallback values or default configurations when parsing fails, ensuring the component can continue operating in degraded mode

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'test' | wc -l | awk '{if ($1 > 0) exit 1}'
- grep -A5 'JSON\.parse.*existingContent' src/agents/FirebenderAgent.ts | grep -q 'console.warn.*Failed to read/parse'
- grep -r 'loadExistingConfig\|saveConfig\|applyRulerConfig\|handleMcpConfiguration' src/ | xargs grep -l 'try.*catch'

Accept when:
- All JSON.parse operations on external input are wrapped in try-catch blocks with error logging
- Configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) handle parse failures without throwing unhandled exceptions
- Error logs include sufficient context (file path, error message) to diagnose configuration issues

## Enforcement

- Verified by: Code review checklist requiring try-catch verification for all JSON.parse operations on external input
- Verified by: Static analysis or linting rules detecting unwrapped JSON.parse calls outside test files
- Verified by: Integration tests that verify graceful handling of malformed JSON configuration files
- Violation handling: Pull requests with unwrapped JSON.parse on external input are blocked until defensive error handling is added
- Violation handling: Existing violations are tracked as technical debt items and prioritized based on component criticality
- Violation handling: Production incidents caused by unhandled parse exceptions trigger immediate remediation and post-mortem review
- Exception process: Exceptions may be granted for JSON.parse operations on trusted internal data structures with documented justification
- Exception process: Exception requests must demonstrate that input is validated through alternative means (e.g., schema validation, type guards)
- Exception process: All exceptions require approval from tech lead and are documented in code comments with rationale