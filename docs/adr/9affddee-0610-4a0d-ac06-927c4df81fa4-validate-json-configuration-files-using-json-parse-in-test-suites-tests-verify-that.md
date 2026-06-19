# Validate JSON Configuration Files Using JSON.parse in Test Suites: Tests Verify That

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all test suites that read and validate JSON configuration files.

## Context

- Test suites across the codebase validate JSON configuration files for multiple AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands) that use MCP (Model Context Protocol) server configurations
- Configuration files are read from filesystem paths using fs/promises and path modules, then parsed to verify structure, keys, and merge behavior
- Tests use describe/it frameworks to verify that configuration merging, agent-specific key usage (servers vs mcpServers), backup prevention, and idempotent re-runs work correctly
- The pattern emerged from the need to validate complex configuration transformations including TOML-to-JSON merging, agent-specific overrides, and type field handling per MCP specification updates
- JSON.parse serves as both a validation mechanism (throws on malformed JSON) and a data extraction tool for assertion-based testing

## Problem Statement

Test suites must reliably validate that JSON configuration files are syntactically correct, structurally sound, and contain expected keys and values after configuration transformations, merges, and agent-specific processing. Without explicit JSON parsing in tests, malformed configurations could pass undetected, leading to runtime failures in production agent integrations.

## Decision

1. SHOULD: Tests SHOULD verify that backup files (.bak) are not created during idempotent configuration operations

## Policy Block

- SHOULD Tests SHOULD verify that backup files (.bak) are not created during idempotent configuration operations

In scope:
- All test files validating JSON configuration files for AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode)
- Tests verifying MCP server configuration merging, overwriting, and agent-specific key usage
- Tests validating configuration file transformations from TOML to JSON
- Tests checking idempotent behavior and backup file prevention

Out of scope:
- Production code that reads configuration files (covered by separate runtime validation ADRs)
- Tests that validate non-JSON configuration formats (TOML, YAML, etc.) unless they verify JSON output
- Unit tests that mock file system operations without actual file I/O

Exceptions:
- EXC-001: Tests explicitly verify error handling for malformed JSON

## Rationale

- The evidence shows 13 test files consistently using JSON.parse after fs.readFile to validate configuration files, establishing a proven pattern with 93.30% confidence
- JSON.parse provides immediate syntax validation by throwing on malformed JSON, catching configuration errors at test time rather than runtime
- Parsing enables structural assertions on nested configuration objects (mcpServers, servers, contextFileName) that are critical to agent integration correctness
- The pattern supports testing complex scenarios including merge strategies, agent-specific key normalization, and specification compliance (e.g., type field handling per Nov 2025 MCP spec update)

## Consequences

Positive:
- Malformed JSON configuration files are detected immediately during test execution via JSON.parse exceptions
- Tests can make precise assertions on nested configuration structure, keys, and values after parsing
- Configuration merge behavior, agent-specific transformations, and idempotent operations are reliably validated
- The pattern provides consistent validation approach across 13+ test files covering multiple agent integrations

Negative:
- Tests become tightly coupled to exact JSON structure, requiring updates when configuration schemas evolve
- JSON.parse throws generic SyntaxError without detailed diagnostics, making debugging of malformed test fixtures harder
- Repeated file reads and parsing in test suites may impact test execution performance for large configuration files
- Tests must handle both JSON.parse exceptions and file system errors, increasing error handling complexity

## Alternatives

- Use JSON schema validation libraries (e.g., ajv, joi) instead of JSON.parse for configuration validation (rejected)
  Rejected because: Adds dependency overhead and complexity when JSON.parse provides sufficient syntax validation for current test needs; schema validation would be more appropriate for production runtime validation
  When valid: When tests need to validate complex schema constraints beyond structure (e.g., regex patterns, conditional fields, cross-field validation)
- Mock file system operations and return pre-parsed JavaScript objects in tests (rejected)
  Rejected because: Eliminates validation of actual file I/O and JSON serialization, which are critical integration points for agent configuration loading
  When valid: For pure unit tests of configuration transformation logic that do not need to verify file system integration
- Use require() or import to load JSON files directly without explicit parsing (rejected)
  Rejected because: Requires .json extension handling and does not work with dynamic file paths constructed during test execution; less explicit about validation intent
  When valid: For static test fixtures with known paths that do not change during test execution

## Risks

- JSON.parse throws generic SyntaxError without line/column information, making it difficult to diagnose malformed test fixtures
  Mitigation: Wrap JSON.parse calls in try-catch blocks with enhanced error messages that include file path and raw content preview
  Owner: Test infrastructure team
- Tests may pass with syntactically valid but semantically incorrect JSON configurations
  Mitigation: Combine JSON.parse with explicit assertions on required keys, value types, and structural constraints using expect() matchers
  Owner: Test authors
- Configuration schema changes may break multiple test files simultaneously due to shared validation pattern
  Mitigation: Centralize common configuration validation logic in test harness utilities; use shared fixtures for common configuration structures
  Owner: Engineering team

## Implementation Notes

- Always read configuration files with utf8 encoding: await fs.readFile(path, 'utf8') before parsing
- Use path.join(projectRoot, relativePath) to construct absolute paths to configuration files in test projects
- Combine JSON.parse with expect() assertions to verify both syntax validity and structural correctness
- For tests verifying merge behavior, parse both source and target files to compare keys using Object.keys().sort()
- When testing backup prevention, use fs.access() to verify .bak files do not exist after operations

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' tests/ --include='*.test.ts' | wc -l
- grep -r 'fs\.readFile.*utf8' tests/ --include='*.test.ts' | grep -c 'JSON\.parse'
- npm test -- --testPathPattern='mcp|gemini|agent' --passWithNoTests

Accept when:
- All test files that read JSON configuration files use JSON.parse to validate syntax before assertions
- Tests successfully detect malformed JSON by catching JSON.parse exceptions
- Configuration validation tests pass for all agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands)

## Enforcement

- Verified by: Automated test suite execution in CI pipeline
- Verified by: Code review verification that new configuration tests include JSON.parse validation
- Verified by: Grep-based verification commands checking for JSON.parse usage in test files
- Violation handling: CI pipeline fails if tests do not properly validate JSON configuration files
- Violation handling: Code review feedback requests addition of JSON.parse validation for configuration tests
- Violation handling: Test failures are investigated to determine if missing JSON validation caused false positives
- Exception process: Test author documents in test description why JSON.parse is not used (e.g., testing error handling)
- Exception process: Code reviewer approves exception based on documented justification
- Exception process: Exception is recorded in test comments with rationale