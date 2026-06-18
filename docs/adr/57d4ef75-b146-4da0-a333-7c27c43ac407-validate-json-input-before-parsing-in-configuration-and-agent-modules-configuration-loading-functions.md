# Validate JSON Input Before Parsing in Configuration and Agent Modules: Configuration Loading Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent and configuration loading modules that parse external JSON or TOML content.

## Context

- The codebase includes configuration loading from external files (firebender.json, TOML config files) that require parsing untrusted or user-provided content
- FirebenderAgent and ConfigLoader modules both parse JSON/TOML content and use console.warn for error logging when parsing fails
- The system uses Zod schema validation (z.object, z.string, z.boolean, z.array, z.record) in ConfigLoader to validate configuration structure after parsing
- Runtime configuration sources include process.env variables (XDG_CONFIG_HOME) and file system reads that may contain malformed or malicious content
- The pattern combines input validation with defensive logging to handle parse failures gracefully while maintaining observability

## Problem Statement

Configuration and agent modules must safely parse external JSON and TOML content from file system sources and environment variables without crashing on malformed input, while providing visibility into parsing failures through structured logging. Without validation and error handling, the system is vulnerable to runtime exceptions from malformed configuration files and lacks observability when configuration loading fails.

## Decision

1. SHOULD: Configuration loading functions SHOULD return graceful defaults or null values when parsing fails rather than propagating exceptions

## Policy Block

- SHOULD Configuration loading functions SHOULD return graceful defaults or null values when parsing fails rather than propagating exceptions

In scope:
- FirebenderAgent configuration loading (firebender.json)
- ConfigLoader TOML and JSON parsing
- Agent configuration schema validation
- MCP configuration parsing
- Environment variable processing (process.env.XDG_CONFIG_HOME)

Out of scope:
- Runtime data validation unrelated to configuration loading
- API request/response validation
- Database query result validation
- Internal data structure validation between trusted modules

## Rationale

- The evidence shows consistent use of JSON.parse with error handling and console.warn logging across FirebenderAgent and ConfigLoader, indicating an established pattern for safe configuration parsing
- Zod schema validation provides runtime type safety and structural validation for complex configuration objects with nested properties (agents, mcp, gitignore, skills)
- Console.warn logging enables debugging and monitoring of configuration issues without terminating the application, supporting operational resilience
- The pattern addresses security concerns by validating untrusted input from file system and environment variables before use in application logic

## Consequences

Positive:
- Prevents application crashes from malformed configuration files through defensive parsing and validation
- Provides operational visibility into configuration loading failures through structured console logging
- Enables type-safe configuration handling with Zod schema validation catching structural errors at runtime
- Supports graceful degradation when configuration files are missing or invalid

Negative:
- Adds runtime overhead for schema validation on every configuration load operation
- Console.warn logging may not integrate with structured logging systems or log aggregation platforms
- Silent failure modes (returning defaults on parse errors) may mask configuration issues in production
- Zod schema definitions require maintenance as configuration structure evolves

## Alternatives

- Use JSON Schema validation instead of Zod for configuration validation (rejected)
  Rejected because: Evidence shows Zod is already integrated throughout ConfigLoader with comprehensive schema definitions; switching would require significant refactoring without clear benefit
  When valid: For new projects without existing Zod dependencies or when JSON Schema tooling is required for cross-language validation
- Fail fast and terminate application on configuration parse errors (rejected)
  Rejected because: Current pattern prioritizes resilience and graceful degradation; hard failures would reduce operational stability for non-critical configuration errors
  When valid: For critical configuration where running with defaults is more dangerous than failing to start
- Use structured logging library (Winston, Pino) instead of console.warn (deferred)
  Rejected because: Would improve log management and integration but requires additional dependencies and configuration; current console.warn provides basic observability
  When valid: When log aggregation, structured querying, or log level management becomes a requirement

## Risks

- Silent configuration failures may go unnoticed in production if console.warn logs are not monitored
  Mitigation: Implement log monitoring and alerting for configuration parse warnings; consider adding metrics for configuration load failures
  Owner: engineering team
- Zod schema drift may occur if schemas are not updated when configuration structure changes
  Mitigation: Include schema validation in unit tests; document schema update requirements in configuration change procedures
  Owner: engineering team
- Performance degradation if complex Zod schemas are validated on every configuration access rather than once at load time
  Mitigation: Validate configuration once during loading and cache validated results; avoid re-parsing on every access
  Owner: engineering team

## Implementation Notes

- Wrap all JSON.parse and TOML.parse operations in try-catch blocks with console.warn logging that includes file path and error details
- Define Zod schemas for all configuration objects using z.object() with appropriate field types (z.string, z.boolean, z.array, z.record, z.enum)
- Use Zod's .optional() modifier for non-required configuration fields to support partial configurations
- Return null or default configuration objects when parsing fails to enable graceful degradation
- Include module-specific prefixes in log messages (e.g., '[ruler]') to facilitate log filtering and debugging

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/ | grep -v 'try\|catch' || echo 'All JSON.parse calls are wrapped'
- grep -r 'console\.warn' src/agents/FirebenderAgent.ts src/core/ConfigLoader.ts | grep -c 'Failed\|Warning'
- grep -r 'z\.object\|z\.string\|z\.boolean' src/core/ConfigLoader.ts | wc -l

Accept when:
- All JSON.parse operations in configuration loading code are wrapped in try-catch blocks with error logging
- ConfigLoader uses Zod schema validation for all configuration objects before accepting values
- Parse failures generate console.warn log entries with file path and error message context

## Enforcement

- Verified by: Code review checklist requiring try-catch wrappers for all external content parsing
- Verified by: Unit tests validating error handling for malformed JSON/TOML input
- Verified by: Static analysis rules detecting unwrapped JSON.parse calls in configuration modules
- Violation handling: Pull requests with unwrapped JSON.parse calls in configuration code are rejected
- Violation handling: Missing Zod validation for new configuration fields triggers review comments
- Violation handling: Runtime exceptions from configuration parsing are treated as P1 bugs
- Exception process: Exceptions require architectural review if parsing trusted internal data structures
- Exception process: Document rationale for any configuration parsing without validation in code comments
- Exception process: Security review required for any changes to input validation patterns