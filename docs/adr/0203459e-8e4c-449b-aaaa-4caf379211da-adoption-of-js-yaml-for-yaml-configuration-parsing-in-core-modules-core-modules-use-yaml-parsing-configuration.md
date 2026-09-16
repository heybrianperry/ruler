# Adoption of `js-yaml` for YAML Configuration Parsing in Core Modules: Core Modules Use Yaml Parsing Configuration

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Core modules require robust mechanisms for reading and interpreting configuration from YAML files.
- YAML is a chosen format for external configuration due to its readability and structured nature.
- File system interactions are necessary to load these configuration files.
- Ensuring the integrity and security of parsed configuration data is critical for subagent operation, as evidenced by `security.input_validation=ON.parse(content)`.
- The project also uses `@iarna/toml` for TOML parsing, indicating a need for multiple configuration formats.

## Problem Statement

Core subagent modules need a standardized and secure approach to parse YAML configuration files, ensuring data integrity and preventing vulnerabilities from malformed input.

## Decision

1. MUST: Core modules MUST use `js-yaml` for parsing YAML configuration files.

## Policy Block

- MUST Core modules MUST use `js-yaml` for parsing YAML configuration files.

In scope:
- Code within `src/core/SubagentsProcessor.ts` and `src/core/SubagentsUtils.ts` that handles YAML configuration.
- Any new or existing core module requiring YAML configuration parsing.

Out of scope:
- Modules not part of the core subagent functionality.
- Parsing of configuration formats other than YAML (e.g., TOML parsing, which uses `@iarna/toml`).

## Rationale

- The observed usage of `js-yaml` in core subagent files indicates an established pattern for handling YAML configuration.
- `js-yaml` provides reliable and efficient parsing capabilities for structured YAML data.
- Explicit input validation during parsing, as observed with `ON.parse(content)`, is crucial for maintaining application stability and security against malformed configuration.
- Standardizing on `js-yaml` for YAML parsing reduces cognitive load and promotes consistency across core modules.

## Consequences

Positive:
- Consistent and reliable parsing of YAML configurations across core modules.
- Enhanced security posture due to mandatory input validation during YAML parsing.
- Reduced development effort by leveraging an established, well-maintained YAML parsing library.

Negative:
- Dependency on an external library (`js-yaml`), requiring maintenance and updates.
- Potential for increased bundle size if not properly managed.
- Developers must be familiar with the `js-yaml` API.

## Alternatives

- Implement custom parsing logic for YAML. (rejected)
  Rejected because: Custom implementations are prone to errors, security vulnerabilities, and require significant maintenance overhead compared to battle-tested libraries.
  When valid: For extremely simple, highly constrained configuration formats where external dependencies are strictly forbidden.
- Use `@iarna/toml` for YAML parsing. (rejected)
  Rejected because: `@iarna/toml` is specifically designed for TOML, not YAML, and would not be suitable for this purpose.
  When valid: When parsing TOML files.
- Use a different third-party YAML parsing library. (rejected)
  Rejected because: `js-yaml` is already established in the codebase, and switching would introduce unnecessary churn without clear benefits.
  When valid: If `js-yaml` proves to have critical unresolvable issues or a superior alternative emerges.

## Risks

- Vulnerabilities discovered in `js-yaml`.
  Mitigation: Regularly update dependencies and monitor security advisories for `js-yaml`.
  Owner: engineering team
- Breaking changes in new versions of `js-yaml`.
  Mitigation: Pin exact versions in lock files and thoroughly test updates before deployment.
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When implementing YAML parsing logic, ensure that the `ON.parse(content)` pattern observed for input validation is consistently applied.
- Consider creating wrapper functions or utility modules to encapsulate `js-yaml` parsing logic and validation for reusability.

## Continuation Context


Verify commands:
- Inspect the project's dependency manifest to identify the declared version of `js-yaml`.
- Examine the project's lock file to confirm the exact resolved version of `js-yaml`.
- Review relevant source files (e.g., those in `src/core/`) to confirm the usage of `js-yaml` for YAML parsing and the presence of input validation.

Accept when:
- The project's dependency manifest explicitly lists `js-yaml`.
- The lock file confirms a specific, resolved version of `js-yaml`.
- Code reviews confirm that core modules use `js-yaml` for YAML parsing and include input validation.

## Enforcement

- Verified by: Automated CI checks for dependency usage and manual code reviews.
- Violation handling: Code failing CI checks will block merges. Code review feedback will require remediation.
- Exception process: Exceptions require approval from a lead architect, documented with a clear rationale and alternative mitigation strategies.