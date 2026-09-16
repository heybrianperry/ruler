# Use of process.env.XDG_CONFIG_HOME for Secret Handling: Sensitive Configuration Secrets Retrieved Environment Variables

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Sensitive configuration and secrets must be externalized from the codebase to prevent accidental exposure in version control.
- The application requires a mechanism to load configuration dynamically based on the execution environment.
- Adherence to standard environment variable conventions for user-specific configuration is desired.
- Configuration loading involves parsing various formats, necessitating robust input validation.

## Problem Statement

The application needs a secure and standardized method for loading sensitive configuration and secrets that are external to the application's source code, while also ensuring the integrity of the loaded data.

## Decision

1. MUST: Sensitive configuration and secrets MUST be retrieved from environment variables, specifically utilizing process.env.XDG_CONFIG_HOME for user-specific configuration paths.

## Policy Block

- MUST Sensitive configuration and secrets MUST be retrieved from environment variables, specifically utilizing process.env.XDG_CONFIG_HOME for user-specific configuration paths.

In scope:
- Modules responsible for application configuration loading.
- Components handling sensitive data or credentials.

Out of scope:
- Non-sensitive application configuration that does not require environment variable externalization.
- Temporary or transient configuration values not persisted across restarts.

## Rationale

- Using process.env.XDG_CONFIG_HOME provides a standardized and secure way to inject secrets and configuration at runtime, preventing them from being hardcoded or committed to source control.
- Externalizing secrets via environment variables aligns with the Twelve-Factor App methodology for configuration management.
- Explicit input validation ensures that loaded configuration data conforms to expected formats and prevents potential injection vulnerabilities or application errors from malformed input.
- The use of XDG_CONFIG_HOME specifically aligns with common Unix-like system conventions for user configuration.

## Consequences

Positive:
- Improved security posture by keeping secrets out of version control.
- Enhanced flexibility in deploying the application across different environments without code changes.
- Standardized approach to configuration loading, reducing ambiguity for developers.
- Reduced risk of runtime errors due to malformed configuration through explicit validation.

Negative:
- Requires careful environment setup and management to ensure necessary environment variables are present.
- Potential for misconfiguration if XDG_CONFIG_HOME is not correctly set or if the referenced files are missing/malformed.
- Debugging configuration issues might be more complex as values are external to the codebase.

## Alternatives

- Hardcoding secrets directly in source code. (rejected)
  Rejected because: This practice is a significant security risk, leading to potential exposure of sensitive information in version control.
  When valid: Never for production secrets.
- Using a dedicated secrets management service (e.g., HashiCorp Vault, AWS Secrets Manager). (rejected)
  Rejected because: While more robust, the current evidence does not indicate the adoption of such a service, and it introduces additional operational overhead not currently observed.
  When valid: For highly sensitive, frequently rotated, or centrally managed secrets in larger, more complex deployments.
- Storing secrets in encrypted files within the repository. (rejected)
  Rejected because: Requires a mechanism for decryption at runtime and still carries the risk of the encryption key being compromised if not managed separately.
  When valid: For environments where environment variables are not feasible and a dedicated secrets manager is overkill, but with careful key management.

## Risks

- Accidental exposure of XDG_CONFIG_HOME content if file permissions are too permissive.
  Mitigation: Implement strict file system permissions for configuration files.
  Owner: engineering team
- Application failure if XDG_CONFIG_HOME is not set or points to an invalid/empty location.
  Mitigation: Implement robust error handling and provide clear fallback mechanisms or default values where appropriate.
  Owner: engineering team
- Inconsistent parsing behavior across different configuration formats or versions of parsing libraries.
  Mitigation: Standardize on specific parsing libraries and versions, and implement comprehensive unit and integration tests for configuration loading.
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
- Ensure that the application's deployment process includes steps for setting the XDG_CONFIG_HOME environment variable and populating the referenced configuration files.
- Consider using a configuration schema validation library in addition to basic parsing to enforce stricter data integrity.

## Continuation Context


Verify commands:
- Discover the project's build scripts and execute the configuration loading tests.
- Inspect the project's environment setup documentation for XDG_CONFIG_HOME configuration.
- Run the application with intentionally malformed configuration files to verify error handling.

Accept when:
- All configuration loading tests pass successfully.
- The application correctly loads sensitive configuration from XDG_CONFIG_HOME at runtime.
- Invalid configuration inputs are gracefully handled, and appropriate errors are reported.

## Enforcement

- Verified by: CI/CD pipelines
- Verified by: code reviews
- Verified by: security audits
- Violation handling: Automated build failures
- Violation handling: code review rejection
- Violation handling: security incident reports
- Exception process: Formal review by architectural review board
- Exception process: documented justification
- Exception process: and approval by security lead.