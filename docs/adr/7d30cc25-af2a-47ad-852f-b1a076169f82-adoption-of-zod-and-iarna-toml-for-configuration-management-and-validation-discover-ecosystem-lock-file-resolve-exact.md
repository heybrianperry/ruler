# Adoption of Zod and Iarna/TOML for Configuration Management and Validation: Discover Ecosystem Lock File Resolve Exact

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- The application requires robust and type-safe configuration loading from various sources.
- Configuration needs to be validated against predefined schemas to ensure correctness and prevent runtime errors.
- Support for different configuration formats, specifically TOML, is necessary.
- Platform-agnostic file system and operating system interactions are fundamental to the application's core utilities.

## Problem Statement

Ensuring consistent, validated, and type-safe loading of application configuration from diverse sources, including TOML files, while handling environment-specific settings and system interactions reliably.

## Decision

1. MUST: MUST discover the ecosystem's lock file and resolve the exact locked version of all third-party dependencies before implementation.

## Policy Block

- MUST MUST discover the ecosystem's lock file and resolve the exact locked version of all third-party dependencies before implementation.

In scope:
- Modules responsible for loading, parsing, or validating application configuration.
- Core utility modules interacting with the file system or operating system.

Out of scope:
- Modules not involved in configuration management or core system interactions.
- User interface components.

## Rationale

- `zod` provides a powerful, declarative, and type-safe way to define schemas and validate data, significantly reducing runtime errors related to malformed configuration.
- `@iarna/toml` offers a reliable and widely adopted solution for parsing TOML configuration files, aligning with existing project needs.
- Leveraging core Node.js modules ensures direct and efficient interaction with the underlying system, which is essential for foundational utilities.
- Centralizing configuration validation with `zod` improves maintainability and consistency across the codebase.

## Consequences

Positive:
- Increased reliability and stability due to robust configuration validation.
- Improved developer experience with type-safe configuration access.
- Consistent approach to handling different configuration formats.
- Reduced boilerplate for configuration parsing and validation.

Negative:
- Introduction of additional third-party dependencies (`zod`, `@iarna/toml`).
- Learning curve for developers unfamiliar with `zod`'s schema definition language.
- Potential for increased bundle size if not properly tree-shaken (though less critical for backend/CLI).

## Alternatives

- Use a different schema validation library (e.g., Joi, Yup, custom validation). (rejected)
  Rejected because: `zod` offers superior type inference and integration with TypeScript, which is a significant advantage for developer experience and maintainability.
  When valid: For projects not using TypeScript or with simpler validation needs.
- Use a different TOML parser or implement a custom one. (rejected)
  Rejected because: `@iarna/toml` is a well-maintained and performant library, and a custom implementation would introduce unnecessary complexity and maintenance overhead.
  When valid: If specific, highly custom TOML parsing behavior is required that `@iarna/toml` cannot provide.

## Risks

- Dependency on external libraries (`zod`, `@iarna/toml`) introduces potential for supply chain vulnerabilities or breaking changes.
  Mitigation: Regularly audit dependencies and pin exact versions in lock files.
  Owner: Engineering team.
- Over-complex `zod` schemas could become difficult to read and maintain.
  Mitigation: Encourage modular schema definitions and clear documentation.
  Owner: Engineering team.

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When defining `zod` schemas for configuration, prioritize clear naming and logical grouping of related fields.
- Ensure that environment variables used for configuration are clearly documented and their precedence is well-defined.

## Continuation Context


Verify commands:
- Inspect the project's dependency manifest to confirm the presence of the specified libraries.
- Examine configuration loading modules to verify `zod` schema definitions and `@iarna/toml` usage.
- Run the project's test suite to ensure configuration loading and validation behave as expected.

Accept when:
- All configuration loading paths successfully validate against their respective `zod` schemas.
- TOML configuration files are correctly parsed by the designated library.
- File system and OS interactions function as intended without errors.

## Enforcement

- Verified by: Automated CI checks for dependency usage and code patterns.
- Verified by: Code reviews for adherence to `zod` schema definition and configuration loading practices.
- Violation handling: CI pipeline failure for non-compliant code.
- Violation handling: Code review comments requiring remediation.
- Exception process: Exceptions require explicit approval from a lead architect, documented with a new ADR outlining the specific justification and alternative approach.