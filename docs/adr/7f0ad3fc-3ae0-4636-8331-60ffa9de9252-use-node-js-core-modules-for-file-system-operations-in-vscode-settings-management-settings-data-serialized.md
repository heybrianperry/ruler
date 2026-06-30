# Use Node.js Core Modules for File System Operations in VSCode Settings Management: Settings Data Serialized

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The VSCode settings management module (src/vscode/settings.ts) requires persistent storage and retrieval of configuration data for MCP server definitions
- File system operations are needed to read and write JSON-formatted settings files that store server configurations including names and properties
- The module maintains an in-memory cache layer using Map data structures (existingServerMap) to track server instances and avoid redundant file operations
- Node.js core modules 'fs' and 'path' provide zero-dependency file system access suitable for local configuration persistence without external database requirements
- The codebase exposes public contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings) that abstract file system operations from consumers

## Problem Statement

Configuration data for VSCode MCP servers must be persisted locally, retrieved reliably, and cached efficiently without introducing external database dependencies or heavyweight storage solutions that would complicate deployment and increase the attack surface.

## Decision

1. MUST: Settings data MUST be serialized and deserialized using JSON.parse for reading configuration content

## Policy Block

- MUST Settings data MUST be serialized and deserialized using JSON.parse for reading configuration content

In scope:
- VSCode settings files containing MCP server configurations
- JSON-formatted configuration data read from and written to the file system
- In-memory Map-based caching of server instances by name
- File system operations within src/vscode/settings.ts module

Out of scope:
- Remote configuration storage or cloud-based settings synchronization
- Binary or non-JSON serialization formats
- Database systems (SQL or NoSQL) for configuration persistence
- Settings unrelated to VSCode MCP server management

## Rationale

- Node.js core modules provide zero-dependency file system access, eliminating external database requirements and simplifying deployment for local configuration storage
- JSON serialization offers human-readable configuration files that can be version-controlled and manually edited when necessary
- Map-based caching reduces redundant file I/O operations by maintaining server instances in memory, improving read performance for frequently accessed configurations
- The FileSystemUtils abstraction centralizes file operations, enabling consistent error handling and potential future migration to alternative storage backends

## Consequences

Positive:
- Zero external dependencies for configuration persistence reduces deployment complexity and attack surface
- Human-readable JSON files enable manual inspection, editing, and version control of settings
- In-memory Map cache improves read performance by avoiding repeated file system access
- Core module usage ensures long-term stability and compatibility across Node.js versions

Negative:
- File system operations introduce I/O latency compared to in-memory-only solutions
- JSON.parse lacks schema validation, requiring additional input validation logic to prevent malformed data
- Local file storage does not support multi-user concurrent access or distributed configuration scenarios
- Cache invalidation logic must be manually implemented to maintain consistency between file and memory state

## Alternatives

- Use SQLite database for configuration persistence with structured schema and query capabilities (rejected)
  Rejected because: Introduces external dependency and binary file format that complicates version control and manual editing of configurations
  When valid: When configuration data requires complex querying, relationships, or transactional guarantees beyond simple key-value storage
- Store configuration in environment variables or command-line arguments only (rejected)
  Rejected because: Does not provide persistence across sessions and limits configuration complexity to flat key-value pairs
  When valid: For ephemeral or containerized environments where configuration is injected at runtime and persistence is not required
- Use third-party configuration management library (e.g., conf, configstore) (rejected)
  Rejected because: Adds external dependency when Node.js core modules provide sufficient functionality for the use case
  When valid: When advanced features like atomic writes, schema validation, or cross-platform path handling are required beyond core module capabilities

## Risks

- Concurrent writes to settings files from multiple processes could corrupt configuration data
  Mitigation: Implement file locking mechanism or atomic write-rename pattern in FileSystemUtils; document single-writer constraint
  Owner: engineering team
- JSON.parse on untrusted input could enable injection attacks or cause runtime errors on malformed data
  Mitigation: Implement input validation and schema checking after JSON.parse; wrap in try-catch with error handling
  Owner: engineering team
- Cache staleness if external processes modify settings files without invalidating in-memory Map
  Mitigation: Implement file watching with fs.watch or document cache refresh requirements; consider TTL-based invalidation
  Owner: engineering team

## Implementation Notes

- Centralize all file system operations in ../core/FileSystemUtils to enable consistent error handling and future storage backend changes
- Wrap JSON.parse calls in try-catch blocks and validate parsed objects against expected schema before populating cache
- Use atomic write patterns (write to temp file, then rename) to prevent partial writes from corrupting configuration files
- Document the cache invalidation strategy and provide explicit refresh methods if external modification of settings files is supported

## Continuation Context


Verify commands:
- grep -r "require.*['\"]fs['\"]\|import.*from.*['\"]fs['\"]" src/vscode/settings.ts
- grep -r "JSON\.parse" src/vscode/settings.ts
- grep -r "existingServerMap\.set\|Map" src/vscode/settings.ts
- grep -r "readVSCodeSettings\|writeVSCodeSettings" src/vscode/settings.ts

Accept when:
- Grep commands confirm presence of 'fs' and 'path' core module imports in src/vscode/settings.ts
- JSON.parse usage is detected for deserializing configuration content
- Map data structure usage with .set() method is confirmed for cache layer implementation
- Public API functions readVSCodeSettings and writeVSCodeSettings are exported from the module

## Enforcement

- Verified by: Automated grep-based verification in CI pipeline checking for core module imports and API contracts
- Verified by: Code review ensuring FileSystemUtils abstraction is used for all file operations
- Verified by: Static analysis detecting direct file system access outside approved modules
- Violation handling: CI build fails if verification commands do not detect required patterns in src/vscode/settings.ts
- Violation handling: Code review rejects PRs introducing alternative storage mechanisms without architectural discussion
- Violation handling: Linting rules flag direct fs module usage outside FileSystemUtils abstraction layer
- Exception process: Document architectural justification for alternative storage approach in ADR amendment
- Exception process: Obtain approval from technical lead for exceptions requiring external database dependencies
- Exception process: Update verification commands and acceptance criteria to reflect approved exceptions