# Standardize Node.js Core Module Integration for Subagent File System Operations: Toml Configuration Parsing

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all subagent file system operations and configuration parsing within the core module boundary.

## Context

- The codebase requires file system operations for discovering, loading, and propagating subagent configurations across the system
- Multiple structured data formats (YAML via js-yaml, TOML via @iarna/toml) must be parsed to support flexible subagent definition files
- Asynchronous file operations are necessary to handle recursive directory traversal and atomic write operations without blocking the event loop
- The SubagentsProcessor and SubagentsUtils modules coordinate subagent lifecycle management including discovery, validation, frontmatter parsing, and gitignore path generation
- Integration with Claude requires propagating subagent metadata through message queues and maintaining tool mapping contracts

## Problem Statement

The system needs a consistent integration pattern for file system operations and structured data parsing across subagent management components, ensuring reliable discovery, validation, and propagation of subagent configurations while maintaining clear API contracts and concurrency boundaries.

## Decision

1. MUST: TOML configuration parsing MUST use the '@iarna/toml' library as the standard parser

## Policy Block

- MUST TOML configuration parsing MUST use the '@iarna/toml' library as the standard parser

In scope:
- All subagent discovery and loading operations in src/core/SubagentsProcessor.ts
- All frontmatter parsing and validation in src/core/SubagentsUtils.ts
- File system operations for markdown file discovery and gitignore path generation
- Configuration parsing for YAML and TOML subagent definition files
- Message queue integration for subagent target propagation

Out of scope:
- Non-subagent file system operations outside the core module boundary
- Alternative structured data formats beyond YAML and TOML
- Synchronous file operations for non-critical paths
- Client-side or browser-based file handling
- External tool integrations beyond Claude propagation

Exceptions:
- EXC-001: Legacy synchronous file operations may be retained during migration period for non-critical utility functions

## Rationale

- The evidence shows consistent use of Node.js core modules ('path', 'fs/promises') across both SubagentsProcessor and SubagentsUtils, indicating an established integration pattern
- Multiple structured data parsers ('js-yaml', '@iarna/toml') are explicitly imported, demonstrating the need to support diverse subagent configuration formats
- Public API contracts are clearly defined with functions like discoverSubagents, parseFrontmatter, and propagateSubagentsForClaude, establishing integration boundaries
- Concurrency model evidence (discoverSubagents, listMarkdownFilesRecursive, loadSubagentFile) and message queue boundaries (targets.add, mapped.add) indicate asynchronous coordination patterns

## Consequences

Positive:
- Consistent asynchronous file operations prevent event loop blocking during subagent discovery and loading
- Standardized parsing libraries reduce configuration format ambiguity and improve interoperability
- Clear API contracts enable reliable integration between subagent management and Claude propagation
- Set-based collection operations guarantee uniqueness of subagent targets and tool mappings

Negative:
- Dependency on external parsing libraries (js-yaml, @iarna/toml) introduces third-party maintenance risk
- Asynchronous-only operations may complicate simple utility functions that could use synchronous APIs
- Multiple supported formats (YAML, TOML) increase parsing complexity and potential error surface area
- Atomic write operations add implementation complexity compared to simple file writes

## Alternatives

- Use synchronous fs module APIs for all file operations (rejected)
  Rejected because: Synchronous operations would block the event loop during recursive directory traversal and large file operations, degrading system responsiveness
  When valid: Only acceptable for one-time initialization scripts or CLI tools with no concurrency requirements
- Support only YAML format and eliminate TOML parsing dependency (rejected)
  Rejected because: Evidence shows @iarna/toml is actively imported, indicating existing TOML configuration files that must be supported
  When valid: Could be reconsidered if all TOML configurations are migrated to YAML and no external TOML dependencies exist
- Implement custom parsers for YAML and TOML instead of using third-party libraries (rejected)
  Rejected because: Custom parser implementation would introduce significant maintenance burden and potential spec compliance issues
  When valid: Only if third-party libraries introduce critical security vulnerabilities or licensing conflicts

## Risks

- Third-party parsing library vulnerabilities (js-yaml, @iarna/toml) could introduce security risks
  Mitigation: Implement automated dependency scanning in CI pipeline and establish update cadence for security patches
  Owner: Engineering team
- Asynchronous file operations may introduce race conditions during concurrent subagent discovery
  Mitigation: Use atomic write operations (writeAgentsDirectoryAtomic) and Set-based deduplication to ensure consistency
  Owner: Engineering team
- Breaking changes in public API contracts could disrupt external consumers and Claude integration
  Mitigation: Implement API versioning and maintain backward compatibility for at least one major version cycle
  Owner: Engineering team

## Implementation Notes

- Import 'fs/promises' as a named import to clearly distinguish async operations from legacy fs module usage
- Wrap all file system operations in try-catch blocks to handle ENOENT, EACCES, and other common file system errors gracefully
- Use path.join() and path.resolve() consistently to avoid hardcoded path separators that break cross-platform compatibility
- Document the expected structure of YAML frontmatter and TOML configuration files in schema definitions or JSDoc comments
- Implement comprehensive validation in validateFrontmatter to catch malformed configurations early in the loading pipeline

## Continuation Context


Verify commands:
- grep -r "require('fs')" src/core/Subagents*.ts | grep -v fs/promises
- grep -r "from 'path'" src/core/Subagents*.ts
- grep -r "from 'js-yaml'" src/core/Subagents*.ts
- grep -r "from '@iarna/toml'" src/core/Subagents*.ts
- grep -E "export.*(discoverSubagents|parseFrontmatter|loadSubagentFile|propagateSubagentsForClaude)" src/core/Subagents*.ts

Accept when:
- All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts use fs/promises API exclusively
- Path module is imported and used for all path manipulation operations
- js-yaml and @iarna/toml are the only parsing libraries used for structured configuration data
- All public API functions (discoverSubagents, parseFrontmatter, validateFrontmatter, loadSubagentFile, propagateSubagentsForClaude) are exported and documented

## Enforcement

- Verified by: Static analysis via ESLint rules prohibiting synchronous fs module usage in core subagent modules
- Verified by: Code review checklist requiring verification of fs/promises usage and proper error handling
- Verified by: Integration tests validating public API contracts and concurrency behavior
- Violation handling: CI pipeline fails on detection of synchronous fs operations in SubagentsProcessor or SubagentsUtils
- Violation handling: Pull requests introducing alternative parsing libraries require architecture review and justification
- Violation handling: Breaking changes to public API contracts trigger automated deprecation warnings and version bump requirements
- Exception process: Submit exception request to core team lead with technical justification and risk assessment
- Exception process: Document approved exceptions in code comments with reference to exception ID and expiration date
- Exception process: Review all active exceptions quarterly to determine if they can be resolved or require extension