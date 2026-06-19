# Standardize Agent Implementation Using Core Interface Contracts and Configuration Loaders: Agent Implementations Implement

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent implementations and configuration loading subsystems within the codebase.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, CopilotAgent, WindsurfAgent, PiAgent, AmpAgent, TraeAgent, WarpAgent) that require consistent interface contracts and configuration management
- Agent implementations depend on core abstractions (IAgent, AbstractAgent, AgentsMdAgent) to establish public API contracts and ensure interoperability across the system
- Configuration loading involves multiple formats (JSON, TOML) requiring unified parsing through UnifiedConfigLoader with validation via JSON.parse and parseTOML operations
- The system coordinates agent behavior through applyRulerConfig operations and MCP (Model Context Protocol) capabilities, requiring standardized integration patterns
- File system utilities, path resolution, and configuration merging are shared concerns across agent implementations, necessitating consistent dependency patterns

## Problem Statement

Agent implementations across the codebase lack a unified approach to interface contracts, configuration loading, and dependency management, leading to inconsistent public API surfaces, duplicated validation logic, and fragile integration points between agents and the core configuration system.

## Decision

1. MUST: All agent implementations MUST implement or extend one of the core interface contracts: IAgent, AbstractAgent, or AgentsMdAgent

## Policy Block

- MUST All agent implementations MUST implement or extend one of the core interface contracts: IAgent, AbstractAgent, or AgentsMdAgent

In scope:
- All agent implementation files in src/agents/ directory
- Configuration loading modules (UnifiedConfigLoader, config-utils)
- Core interface contracts (IAgent, AbstractAgent, AgentsMdAgent)
- MCP capability coordination modules
- Agent utility functions and shared helpers

Out of scope:
- Internal agent implementation details not exposed through public contracts
- Agent-specific business logic unrelated to configuration or interface contracts
- Test fixtures and mock implementations
- Legacy agent implementations marked for deprecation

Exceptions:
- EXC-001: Agent requires custom configuration format not supported by UnifiedConfigLoader (e.g., binary formats, proprietary schemas)
- EXC-002: Agent operates in isolated runtime environment without access to core file system utilities

## Rationale

- Evidence shows 20 files with consistent import patterns for core interfaces (IAgent, AbstractAgent, AgentsMdAgent) and configuration utilities, indicating an established architectural pattern
- Multiple agents (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent) demonstrate identical security.input_validation patterns using JSON.parse and parseTOML, validating the need for standardized parsing
- The presence of UnifiedConfigLoader, config-utils, and mcp/capabilities modules indicates intentional separation of configuration concerns from agent business logic
- Pattern confidence of 90.30% across 20 files with consistent api.public.contracts exports demonstrates mature adoption of interface-based design

## Consequences

Positive:
- Consistent public API surface across all agent implementations enables predictable integration and testing
- Centralized configuration loading through UnifiedConfigLoader reduces validation logic duplication and parsing errors
- Shared core dependencies (fs, path, @iarna/toml) establish clear dependency boundaries and simplify maintenance
- MCP capability coordination through standardized functions enables uniform protocol support across agents

Negative:
- Agent implementations must conform to core interface contracts even when specific agent requirements differ from common patterns
- UnifiedConfigLoader becomes a critical dependency point; failures impact all agents simultaneously
- Adding new configuration formats requires changes to core loader rather than agent-local extensions
- Abstraction overhead may reduce performance for simple agents that don't require full configuration machinery

## Alternatives

- Allow each agent to implement custom configuration loading and validation logic without shared abstractions (rejected)
  Rejected because: Evidence shows duplicated validation patterns (JSON.parse, parseTOML) across multiple agents, indicating maintenance burden and inconsistent error handling
  When valid: Only for agents with truly unique configuration requirements that cannot be accommodated by extensible core loaders
- Use duck typing without explicit interface contracts (IAgent, AbstractAgent) for agent implementations (rejected)
  Rejected because: 20 files consistently export api.public.contracts, demonstrating intentional contract-based design for type safety and integration guarantees
  When valid: In prototyping phases before agent patterns stabilize, or for internal-only agents never exposed through public APIs
- Implement agent-specific utility modules rather than shared agent-utils and config-utils (rejected)
  Rejected because: Evidence shows common operations (getAgentOutputPaths, mapRawAgentConfigs) already extracted to shared utilities, indicating successful abstraction of cross-cutting concerns
  When valid: When agent-specific operations have no overlap with other agents and shared utilities would create unnecessary coupling

## Risks

- Core interface contracts (IAgent, AbstractAgent) may not accommodate future agent types with fundamentally different operational models
  Mitigation: Design interfaces with extension points and version core contracts to allow backward-compatible evolution; document extension patterns in interface documentation
  Owner: Architecture team and agent subsystem maintainers
- UnifiedConfigLoader centralization creates single point of failure for all agent configuration loading
  Mitigation: Implement comprehensive error handling, fallback mechanisms, and circuit breakers in loader; maintain high test coverage for all supported formats
  Owner: Core configuration team
- Dependency on external parsing libraries (@iarna/toml) introduces supply chain and compatibility risks
  Mitigation: Pin dependency versions, monitor for security advisories, and document migration path to alternative parsers if needed
  Owner: Security and dependency management team

## Implementation Notes

- New agent implementations should extend AbstractAgent or AgentsMdAgent rather than implementing IAgent directly unless specific requirements dictate otherwise
- Use UnifiedConfigLoader.loadUnifiedConfig with appropriate format hints (TOML vs JSON) based on configuration file extension
- Import file system utilities from core/FileSystemUtils rather than using fs/fs.promises directly to benefit from error handling and path normalization
- For agents requiring MCP support, call agentSupportsMcp early in initialization to determine capability availability before attempting MCP operations
- Export agent classes as named exports matching the agent name pattern (e.g., export class OpenCodeAgent) to maintain consistent public API discovery

## Continuation Context


Verify commands:
- grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/ | wc -l
- grep -r 'import.*UnifiedConfigLoader' src/agents/ src/core/ | grep -v test
- grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -E '(OpenCodeAgent|MistralVibeAgent|CodexCliAgent|AmazonQCliAgent|UnifiedConfigLoader)'
- grep -r 'export class.*Agent' src/agents/ | grep -v test

Accept when:
- All agent implementation files in src/agents/ extend or implement one of the core interface contracts (IAgent, AbstractAgent, AgentsMdAgent)
- Configuration loading operations use UnifiedConfigLoader with JSON.parse or parseTOML for validation, with no custom parsing logic duplicated across agents
- Agent classes are exported as named exports matching established naming patterns and can be imported by integration modules

## Enforcement

- Verified by: Static analysis tools checking for interface implementation compliance in agent files
- Verified by: Code review checklist requiring UnifiedConfigLoader usage for new agent configurations
- Verified by: Integration tests validating agent public API contracts match expected interfaces
- Verified by: Dependency graph analysis ensuring agents import from approved core modules
- Violation handling: CI pipeline fails if agent implementations lack required interface contracts
- Violation handling: Pull requests blocked if custom configuration parsing duplicates UnifiedConfigLoader functionality
- Violation handling: Architecture review required for agents not using standard core dependencies
- Violation handling: Automated linting rules flag direct fs operations without FileSystemUtils wrapper
- Exception process: Submit exception request documenting specific agent requirements incompatible with core contracts
- Exception process: Architecture team reviews technical justification and evaluates core interface extension options
- Exception process: Approved exceptions documented in agent configuration metadata with migration timeline
- Exception process: Quarterly review of exceptions to identify patterns requiring core interface evolution