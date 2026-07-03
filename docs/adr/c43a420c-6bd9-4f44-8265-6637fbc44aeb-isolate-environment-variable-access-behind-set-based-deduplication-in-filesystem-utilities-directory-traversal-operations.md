# Isolate Environment Variable Access Behind Set-Based Deduplication in Filesystem Utilities: Directory Traversal Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all filesystem utility modules that access environment variables for configuration path resolution.

## Context

- The FileSystemUtils module accesses process.env.XDG_CONFIG_HOME to resolve global configuration directories, creating a direct dependency on environment variables that may contain sensitive path information
- Directory traversal operations use a visitedDirectories.add(realDir) pattern to track visited paths, implementing set-based deduplication to prevent infinite loops in symbolic link resolution
- The module combines security-sensitive operations (symbolic link validation, root containment checks) with environment variable access, requiring isolation of credential-like configuration sources
- Error logging via console.error exposes configuration directory paths and environment-derived values, creating potential information disclosure vectors if not properly bounded

## Problem Statement

When filesystem utilities directly access environment variables for configuration path resolution while simultaneously performing security-critical operations like symbolic link validation and directory traversal, there is no clear boundary between credential sources and operational logic, increasing the risk of credential leakage through error messages, logs, or unvalidated path expansion.

## Decision

1. MUST: Directory traversal operations that track visited paths MUST use Set-based deduplication (visitedDirectories.add) to prevent infinite loops and ensure deterministic path resolution

## Policy Block

- MUST Directory traversal operations that track visited paths MUST use Set-based deduplication (visitedDirectories.add) to prevent infinite loops and ensure deterministic path resolution

In scope:
- All filesystem utility modules that access process.env for configuration path resolution
- Directory traversal functions that resolve symbolic links or validate path containment
- Error logging and diagnostic output that may expose configuration paths
- Functions that combine environment variable access with security-critical filesystem operations

Out of scope:
- Environment variables used for non-path configuration (feature flags, log levels)
- Hardcoded default paths that do not derive from environment variables
- Third-party library filesystem operations outside the core utility module
- Test fixtures and mock filesystem implementations

Exceptions:
- EXC-001: Development and debugging environments where full path disclosure is required for troubleshooting

## Rationale

- The evidence shows direct process.env.XDG_CONFIG_HOME access within a module that performs security-critical operations (symbolic link validation, root containment checks), creating a coupling between credential sources and operational logic
- The visitedDirectories.add(realDir) pattern demonstrates an existing architectural commitment to set-based deduplication for preventing traversal loops, which should be extended to credential handling boundaries
- Console.error logging of configuration directory paths (${globalConfigDir}) creates information disclosure risk if environment-derived paths contain sensitive information or reveal system structure
- Isolating environment variable access behind validation layers reduces the attack surface for path traversal vulnerabilities and credential leakage while maintaining the existing deduplication semantics

## Consequences

Positive:
- Reduced risk of credential leakage through error messages and logs by centralizing environment variable access and validation
- Clearer separation of concerns between configuration retrieval and filesystem operations, improving testability and maintainability
- Consistent application of set-based deduplication patterns across both path tracking and credential source isolation
- Easier audit trail for environment variable access, enabling security reviews to focus on isolated accessor functions

Negative:
- Additional abstraction layer increases code complexity and may introduce performance overhead for repeated configuration path resolution
- Existing code must be refactored to route environment access through new accessor functions, requiring migration effort
- Overly aggressive redaction in error messages may hinder legitimate debugging and troubleshooting activities
- Set-based tracking of visited directories adds memory overhead proportional to traversal depth in deeply nested filesystem structures

## Alternatives

- Continue direct process.env access throughout filesystem utilities without isolation (rejected)
  Rejected because: Maintains tight coupling between credential sources and operational logic, increasing risk of credential leakage through error paths and making security audits more difficult
  When valid: Only acceptable in throwaway prototypes or single-use scripts with no security requirements
- Use a global configuration singleton that caches all environment variables at startup (rejected)
  Rejected because: Creates a single point of failure and prevents runtime configuration updates; does not address the core issue of isolating credential access from filesystem operations
  When valid: May be appropriate for containerized environments with immutable configuration where startup-time validation is sufficient
- Implement dependency injection for configuration sources, passing validated paths as function parameters (deferred)
  Rejected because: Requires more extensive refactoring than accessor functions but provides better testability and inversion of control
  When valid: Should be reconsidered if the module evolves to support multiple configuration sources or requires more sophisticated testing strategies

## Risks

- Incomplete migration leaves some environment variable access points unprotected, creating inconsistent security boundaries
  Mitigation: Use static analysis (grep for 'process.env') to identify all access points and track migration progress; enforce through linting rules
  Owner: Engineering team
- Performance degradation from repeated validation of environment-derived paths in hot code paths
  Mitigation: Implement caching for validated configuration paths with appropriate invalidation strategies; profile critical paths to measure impact
  Owner: Engineering team
- Redacted error messages may obscure root causes during incident response, increasing mean time to resolution
  Mitigation: Implement structured logging with separate channels for detailed diagnostics (restricted access) and sanitized user-facing errors; document redaction policies
  Owner: Security team and SRE team

## Implementation Notes

- Create a dedicated ConfigPathResolver module with functions like getXdgConfigHome() that validate and sanitize environment variable values before returning them
- Refactor existing process.env.XDG_CONFIG_HOME access in FileSystemUtils.ts to use the new accessor functions, maintaining the existing visitedDirectories.add(realDir) deduplication pattern
- Update error logging to use path redaction utilities that mask sensitive portions of configuration paths while preserving debugging utility (e.g., show basename but mask parent directories)
- Add unit tests that verify environment variable access only occurs through designated accessor functions and that error messages do not leak raw environment values

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' src/core/FileSystemUtils.ts | grep -v 'ConfigPathResolver' | wc -l | grep -q '^0$'
- grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts | wc -l | grep -q -v '^0$'
- grep -r 'console\.error.*\${.*env' src/core/ | wc -l | grep -q '^0$'

Accept when:
- All process.env access in FileSystemUtils.ts routes through dedicated accessor functions (zero direct access outside ConfigPathResolver)
- Set-based deduplication via visitedDirectories.add remains present and functional for directory traversal
- Error logging does not expose raw environment variable values in console.error calls

## Enforcement

- Verified by: Static analysis via ESLint rule prohibiting direct process.env access outside designated modules
- Verified by: Code review checklist requiring verification of environment variable isolation in filesystem utilities
- Verified by: Automated grep-based verification in CI pipeline checking for process.env patterns outside accessor functions
- Violation handling: CI build fails if direct process.env access is detected outside ConfigPathResolver module
- Violation handling: Pull requests introducing new environment variable access must include security review justification
- Violation handling: Existing violations flagged in technical debt backlog with priority based on exposure risk
- Exception process: Developer submits exception request documenting the specific use case and why accessor functions are insufficient
- Exception process: Security team reviews exception for credential leakage risk and path traversal implications
- Exception process: Approved exceptions must include inline comments with exception ID and expiration date for future review