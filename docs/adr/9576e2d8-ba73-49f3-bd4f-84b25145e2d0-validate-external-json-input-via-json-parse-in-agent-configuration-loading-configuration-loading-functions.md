# Validate External JSON Input via JSON.parse in Agent Configuration Loading: Configuration Loading Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent implementations that load external configuration files.

## Context

- Agent implementations (GeminiCliAgent, OpenCodeAgent, ZedAgent) load configuration from external JSON files using fs.readFile or fs/promises
- Configuration files contain settings, MCP paths, and agent-specific parameters that control runtime behavior
- JSON.parse is consistently applied to file content strings before consumption, establishing a validation boundary
- The pattern appears in both test harnesses (GeminiCliAgent.test.ts) and production agent code (OpenCodeAgent.ts, ZedAgent.ts)
- Input validation through JSON.parse provides syntactic validation and prevents malformed configuration from propagating into agent logic

## Problem Statement

Agent implementations must safely consume external configuration files without allowing malformed, syntactically invalid, or corrupted JSON to compromise agent initialization or runtime behavior. Without explicit validation at the file-to-object boundary, agents risk runtime errors, undefined behavior, or security vulnerabilities from untrusted or corrupted configuration sources.

## Decision

1. SHOULD: Configuration loading functions (readGeminiSettings, applyRulerConfig) SHOULD fail fast when JSON.parse throws, preventing agents from initializing with invalid state

## Policy Block

- SHOULD Configuration loading functions (readGeminiSettings, applyRulerConfig) SHOULD fail fast when JSON.parse throws, preventing agents from initializing with invalid state

In scope:
- All agent classes extending AgentsMdAgent or implementing IAgent
- Configuration loading functions (readGeminiSettings, applyRulerConfig)
- Test harnesses that load agent settings from JSON files
- MCP configuration loading in OpenCodeAgent
- Settings file parsing in GeminiCliAgent and ZedAgent

Out of scope:
- JSON responses from external APIs or network sources (covered by separate API validation rules)
- In-memory JSON serialization/deserialization within agent logic
- JSON configuration embedded directly in source code as literals
- Non-JSON configuration formats (YAML, TOML, INI)

## Rationale

- The pattern is observed consistently across 3 agent implementations with 91.63% confidence, indicating an established architectural convention
- JSON.parse provides a clear validation boundary that prevents syntactically invalid data from entering agent state, reducing the attack surface for configuration-based vulnerabilities
- The pattern aligns with the security.input_validation facet by treating external file content as untrusted input requiring explicit validation
- Consistent use of JSON.parse across agent implementations establishes a predictable error-handling contract for configuration loading failures

## Consequences

Positive:
- Syntactic validation at the file-to-object boundary prevents malformed JSON from causing undefined behavior in agent initialization
- JSON.parse throws SyntaxError on invalid input, providing a clear failure mode that can be caught and handled explicitly
- Consistent validation pattern across agent implementations reduces cognitive load and improves maintainability
- Early validation failure prevents agents from initializing with corrupted or incomplete configuration state

Negative:
- JSON.parse only validates syntax, not schema or semantic correctness—agents may still receive structurally valid but semantically incorrect configuration
- Exception handling for JSON.parse failures adds boilerplate to configuration loading functions
- Performance overhead of parsing large configuration files on every agent initialization (though typically negligible for config files)
- No protection against valid JSON containing malicious or unexpected values without additional schema validation

## Alternatives

- Use a JSON schema validator (e.g., ajv, zod) to enforce both syntactic and semantic validation (rejected)
  Rejected because: Evidence shows only JSON.parse usage without schema validation libraries; adding schema validation would require additional dependencies and is not reflected in the detected pattern
  When valid: When configuration complexity grows and semantic validation becomes critical for preventing runtime errors from valid-but-incorrect JSON
- Trust file system content without explicit validation, relying on TypeScript type assertions (rejected)
  Rejected because: Eliminates the validation boundary and exposes agents to runtime errors from malformed configuration; contradicts the observed security.input_validation pattern
  When valid: Never—external file content should always be treated as untrusted input
- Use require() for JSON files to leverage Node.js built-in JSON parsing (rejected)
  Rejected because: Evidence shows explicit fs.readFile + JSON.parse pattern, not require(); require() also caches modules and doesn't support async loading patterns used in the codebase
  When valid: For static configuration files that don't change at runtime and where synchronous loading is acceptable

## Risks

- JSON.parse alone does not prevent semantically invalid configuration (e.g., missing required fields, wrong types) from reaching agent logic
  Mitigation: Document expected configuration schema and consider adding runtime type validation using TypeScript type guards or validation libraries for critical configuration fields
  Owner: engineering team
- Inconsistent error handling across agent implementations may lead to different failure modes for the same configuration error
  Mitigation: Establish a shared configuration loading utility that standardizes JSON.parse error handling and provides consistent error messages across all agents
  Owner: engineering team
- Large or deeply nested JSON configuration files may cause performance degradation or stack overflow during parsing
  Mitigation: Establish configuration file size limits and monitor parsing performance; consider streaming parsers for exceptionally large configuration files
  Owner: engineering team

## Implementation Notes

- Wrap JSON.parse calls in try-catch blocks to handle SyntaxError exceptions and provide context about which configuration file failed to parse
- Use async/await with fs/promises for configuration loading to maintain consistency with the observed pattern in OpenCodeAgent and other implementations
- Consider creating a shared utility function (e.g., loadJsonConfig) that encapsulates the fs.readFile + JSON.parse pattern with standardized error handling
- Log the configuration file path in error messages to aid debugging when JSON.parse fails in production environments

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*await fs\.readFile' src/agents/ tests/unit/agents/
- grep -r 'readGeminiSettings\|applyRulerConfig' src/agents/ --include='*.ts' -A 5 | grep 'JSON\.parse'
- npm test -- --testPathPattern='GeminiCliAgent|OpenCodeAgent|ZedAgent' --testNamePattern='configuration|settings'

Accept when:
- All agent implementations that load external JSON configuration files use JSON.parse on the raw file content string
- Configuration loading functions (readGeminiSettings, applyRulerConfig) contain explicit JSON.parse calls as evidenced by grep results
- Tests for GeminiCliAgent, OpenCodeAgent, and ZedAgent pass, confirming configuration loading behavior is preserved

## Enforcement

- Verified by: Code review checklist requiring JSON.parse for all new agent configuration loading code
- Verified by: Automated grep-based verification in CI pipeline checking for JSON.parse usage in agent configuration functions
- Verified by: Unit tests validating that configuration loading functions handle malformed JSON appropriately
- Violation handling: CI pipeline fails if new agent implementations load JSON configuration without JSON.parse validation
- Violation handling: Code review blocks merge requests that introduce configuration loading without explicit parsing
- Violation handling: Runtime errors from malformed configuration trigger alerts and are tracked as P1 incidents
- Exception process: Exceptions require architectural review and must document why JSON.parse is inappropriate for the specific use case
- Exception process: Alternative validation mechanisms must be explicitly documented and provide equivalent or stronger guarantees
- Exception process: Exception approval requires sign-off from security and architecture teams