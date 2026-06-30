# Standardize Core Library Imports for Subagent File Processing: Public Contracts Parsedfrontmatter

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all subagent file processing modules.

## Context

- The codebase processes subagent configuration files that require parsing YAML and TOML formats, necessitating consistent library choices across modules
- File system operations for discovering, loading, and writing subagent files are performed across multiple modules (SubagentsUtils.ts and SubagentsProcessor.ts)
- Frontmatter parsing and validation require standardized contracts (ParsedFrontmatter, CopilotToolMapping) to ensure consistent data structures
- Asynchronous file operations using fs/promises enable non-blocking I/O for subagent file discovery and processing workflows

## Problem Statement

Without standardized core library imports and consistent API contracts across subagent processing modules, the system risks fragmentation in file parsing logic, inconsistent error handling, and duplicated validation code, making maintenance and extension of subagent functionality more difficult.

## Decision

1. SHOULD: Public API contracts (ParsedFrontmatter, CopilotToolMapping) SHOULD be defined in shared type definitions to maintain consistency

## Policy Block

- SHOULD Public API contracts (ParsedFrontmatter, CopilotToolMapping) SHOULD be defined in shared type definitions to maintain consistency

In scope:
- All modules in src/core/ that process subagent files
- SubagentsUtils.ts and SubagentsProcessor.ts modules
- Functions that parse, validate, discover, or write subagent configuration files
- Type definitions for subagent data structures (ParsedFrontmatter, CopilotToolMapping)

Out of scope:
- Non-subagent file processing modules
- Client-side or UI components that consume subagent data but do not process files
- Test utilities that may use alternative mocking libraries
- Build-time scripts that operate outside the runtime module system

## Rationale

- The evidence shows consistent use of 'path', 'fs/promises', 'js-yaml', and '@iarna/toml' across SubagentsUtils.ts and SubagentsProcessor.ts, indicating an established pattern
- Standardizing on these libraries reduces dependency bloat and ensures consistent behavior across file parsing operations
- Shared API contracts (ParsedFrontmatter, CopilotToolMapping) and utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) demonstrate intentional code reuse
- The pattern supports concurrent file operations (discoverSubagents, loadSubagentFile) which aligns with the asynchronous nature of fs/promises

## Consequences

Positive:
- Consistent file parsing behavior across all subagent processing modules reduces debugging complexity
- Shared utility functions and type contracts reduce code duplication and maintenance burden
- Asynchronous file operations using fs/promises improve performance for bulk subagent discovery and loading
- Standardized library choices simplify dependency management and security auditing

Negative:
- Tight coupling to specific parsing libraries (js-yaml, @iarna/toml) makes migration to alternatives more difficult
- Developers must learn the specific APIs of the mandated libraries rather than choosing familiar alternatives
- Adding support for new configuration formats requires updating the standard library set and potentially all consuming modules

## Alternatives

- Allow each module to choose its own YAML/TOML parsing library (rejected)
  Rejected because: Would lead to inconsistent parsing behavior, increased dependency count, and fragmented error handling across modules
  When valid: In a microservices architecture where each service manages its own dependencies independently
- Use synchronous fs methods instead of fs/promises (rejected)
  Rejected because: Synchronous file operations would block the event loop during bulk subagent discovery, degrading performance
  When valid: In CLI tools or scripts where blocking I/O is acceptable and concurrency is not required
- Create an abstraction layer over file parsing libraries (deferred)
  Rejected because: Not rejected; could be considered if library migration becomes necessary, but adds complexity without current benefit
  When valid: When multiple parsing library migrations are anticipated or when supporting pluggable parsing backends

## Risks

- Security vulnerabilities in js-yaml or @iarna/toml could affect all subagent processing modules
  Mitigation: Implement automated dependency scanning in CI/CD pipeline and establish a process for rapid library updates
  Owner: engineering team
- Breaking changes in library APIs during major version updates could require widespread code changes
  Mitigation: Pin library versions in package.json and test updates in isolated branches before rolling out
  Owner: engineering team
- Performance bottlenecks in chosen libraries may not be discovered until production scale
  Mitigation: Implement performance benchmarks for file parsing operations and monitor in production
  Owner: engineering team

## Implementation Notes

- Import 'fs/promises' as a named import (e.g., import { readFile, writeFile } from 'fs/promises') for tree-shaking benefits
- Centralize error handling for YAML/TOML parsing failures in utility functions to provide consistent error messages
- Document the expected structure of ParsedFrontmatter and CopilotToolMapping in TypeScript interfaces with JSDoc comments
- Use path.join() and path.resolve() consistently to avoid hardcoded path separators that break on Windows

## Continuation Context


Verify commands:
- grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts
- grep -r "import.*from 'js-yaml'" src/core/Subagents*.ts
- grep -r "import.*from '@iarna/toml'" src/core/SubagentsProcessor.ts
- grep -r "parseFrontmatter\|validateFrontmatter\|loadSubagentFile" src/core/SubagentsUtils.ts

Accept when:
- All subagent processing modules import fs/promises, js-yaml, and path from the standard library set
- Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) are exported from SubagentsUtils.ts and imported by consuming modules
- No synchronous fs methods are used in SubagentsUtils.ts or SubagentsProcessor.ts

## Enforcement

- Verified by: Static analysis via grep commands in CI pipeline to detect non-standard library imports
- Verified by: Code review checklist requiring verification of library imports in subagent-related PRs
- Verified by: TypeScript compilation checks ensuring shared type contracts are used consistently
- Violation handling: CI pipeline fails if non-standard parsing libraries are detected in subagent processing modules
- Violation handling: Code review requires justification and architectural approval for deviations from standard library set
- Violation handling: Automated linting rules flag synchronous fs usage in async contexts
- Exception process: Developer submits exception request documenting the technical justification and scope
- Exception process: Architecture review board evaluates whether the exception warrants updating the standard or is truly exceptional
- Exception process: Approved exceptions are documented in code comments with ADR reference and expiration review date