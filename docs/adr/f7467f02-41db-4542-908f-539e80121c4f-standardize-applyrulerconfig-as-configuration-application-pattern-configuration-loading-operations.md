# Standardize applyRulerConfig as Configuration Application Pattern: Configuration Loading Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent implementations and configuration loading modules within the Ruler system.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, CopilotAgent, AiderAgent) that require consistent configuration application mechanisms
- Agent configuration files are stored in multiple formats (JSON, TOML) and require parsing with validation before application
- The system uses a unified configuration loading pattern (UnifiedConfigLoader) that coordinates with agent-specific configuration application
- Configuration reversion capabilities (revert-engine.ts) require predictable configuration application patterns to enable rollback operations
- The applyRulerConfig pattern appears consistently across 8 files with 90.75% confidence, indicating an established architectural convention

## Problem Statement

Agent implementations require a consistent, predictable mechanism for applying Ruler configuration that coordinates with input validation, file system operations, and reversion capabilities while supporting multiple configuration formats and agent-specific customization requirements.

## Decision

1. MUST: Configuration loading operations MUST use FileSystemUtils or fs/promises for file system access to maintain consistency with revert-engine expectations

## Policy Block

- MUST Configuration loading operations MUST use FileSystemUtils or fs/promises for file system access to maintain consistency with revert-engine expectations

In scope:
- All agent implementation modules in src/agents/ directory
- Configuration loading modules (UnifiedConfigLoader, ConfigLoader)
- Reversion and cleanup operations in revert-engine.ts
- Agent utility functions that manipulate configuration state

Out of scope:
- Third-party agent tools that do not integrate with Ruler configuration system
- Runtime agent behavior after configuration has been applied
- Agent-specific business logic unrelated to configuration application
- User-facing CLI commands that invoke but do not implement configuration application

Exceptions:
- EXC-001: Legacy agent implementations that predate the applyRulerConfig pattern and are scheduled for deprecation
- EXC-002: Experimental agent prototypes in development that have not reached integration phase

## Rationale

- The applyRulerConfig pattern appears in 8 files across agent implementations and configuration modules, demonstrating an established architectural convention with 90.75% confidence
- Consistent configuration application enables the revert-engine to reliably restore previous states by depending on predictable file system operations and configuration structure
- Centralized input validation through JSON.parse and parseTOML at configuration application boundaries reduces security risks from malformed configuration files
- The pattern coordinates with IAgent interface contracts and AgentsMdAgent base class to enforce consistent public APIs across heterogeneous agent implementations

## Consequences

Positive:
- Predictable configuration application enables reliable reversion and rollback capabilities across all agent types
- Consistent input validation at configuration boundaries reduces security vulnerabilities from malformed or malicious configuration files
- Standardized pattern reduces cognitive load for developers implementing new agent types or maintaining existing ones
- Coordination with UnifiedConfigLoader enables centralized hash computation and rule processing for configuration provenance tracking

Negative:
- Agent implementations with unique configuration requirements must adapt to the applyRulerConfig interface, potentially requiring wrapper logic
- The pattern creates coupling between agent implementations and core library modules (IAgent, FileSystemUtils), reducing agent portability
- Changes to the applyRulerConfig interface contract require coordinated updates across 8+ files
- Additional abstraction layer may introduce performance overhead for simple configuration operations

## Alternatives

- Allow each agent to implement configuration application using agent-specific patterns without standardization (rejected)
  Rejected because: Lack of standardization prevents revert-engine from reliably restoring configuration state and increases maintenance burden across heterogeneous agent implementations
  When valid: Only valid for isolated agent prototypes that do not require integration with Ruler configuration management or reversion capabilities
- Use event-driven configuration application with pub-sub pattern instead of direct method invocation (rejected)
  Rejected because: Event-driven approach introduces asynchronous complexity and makes configuration application order non-deterministic, complicating error handling and rollback scenarios
  When valid: Could be reconsidered if the system evolves to require real-time configuration updates across distributed agent instances
- Implement configuration application as a separate service layer rather than agent instance methods (deferred)
  Rejected because: Would require significant refactoring of existing agent implementations and may provide benefits for future microservice architecture
  When valid: Should be reconsidered if the system transitions to a distributed architecture with centralized configuration management service

## Risks

- Breaking changes to applyRulerConfig interface require coordinated updates across 8+ agent implementations, creating deployment risk
  Mitigation: Implement interface versioning and deprecation warnings; use adapter pattern for backward compatibility during transitions; require comprehensive integration tests before interface changes
  Owner: Engineering team with architecture review board oversight
- Input validation vulnerabilities in JSON.parse or parseTOML could affect all agent implementations simultaneously
  Mitigation: Centralize validation logic in shared utility module; implement schema validation using JSON Schema or equivalent; add fuzzing tests for configuration parsers
  Owner: Security team with engineering implementation support
- Tight coupling to FileSystemUtils and core libraries reduces agent portability and complicates testing
  Mitigation: Use dependency injection for file system operations; provide mock implementations for testing; document clear interface boundaries in IAgent contract
  Owner: Engineering team

## Implementation Notes

- New agent implementations should extend IAgent interface or AgentsMdAgent base class and implement applyRulerConfig as the primary configuration entry point
- Use JSON.parse for JSON configuration files and parseTOML from @iarna/toml for TOML files, wrapping parse operations in try-catch blocks with meaningful error messages
- Coordinate with UnifiedConfigLoader by accepting configuration objects that have already been processed through hash computation and rule processing pipelines
- Ensure file system operations use FileSystemUtils or fs/promises to maintain compatibility with revert-engine backup and restore operations
- Document agent-specific configuration extensions in module headers and ensure they do not break the core applyRulerConfig contract

## Continuation Context


Verify commands:
- grep -r 'applyRulerConfig' src/agents/ --include='*.ts' | wc -l
- grep -r 'JSON\.parse\|parseTOML' src/agents/ --include='*.ts' -A 2 | grep -c 'applyRulerConfig'
- grep -r "from.*['\"]\./IAgent" src/agents/ --include='*.ts' | wc -l

Accept when:
- All agent implementation files in src/agents/ contain applyRulerConfig method or function definition
- Configuration parsing operations (JSON.parse, parseTOML) are present in agent files that implement applyRulerConfig
- Agent implementations import from './IAgent' or './AgentsMdAgent' to maintain interface contract compliance

## Enforcement

- Verified by: Automated grep-based verification in CI pipeline checking for applyRulerConfig presence in agent implementations
- Verified by: Code review checklist requiring verification of IAgent interface compliance for new agent implementations
- Verified by: Integration tests validating configuration application and reversion workflows for each agent type
- Violation handling: CI pipeline fails if new agent implementations lack applyRulerConfig method
- Violation handling: Code review blocks merge if agent does not import from IAgent or AgentsMdAgent base classes
- Violation handling: Runtime warnings logged when configuration application bypasses standard validation patterns
- Exception process: Submit exception request to architecture review board with justification and migration timeline
- Exception process: Document exception in agent module header with JIRA ticket reference and expiration date
- Exception process: Quarterly review of active exceptions to assess migration progress or extend approval