# Standardize applyRulerConfig and loadSubagentFile as Concurrency Coordination Points in Agent Initialization: Agent Implementations Validate

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent implementations and subagent loading workflows within the system.

## Context

- Agent implementations (OpenCodeAgent, ZedAgent) require configuration application during initialization, with applyRulerConfig serving as a synchronization point for rule-based configuration
- Subagent discovery and loading workflows (SubagentsUtils, SubagentsProcessor) coordinate file system operations through loadSubagentFile and discoverSubagents functions
- The codebase processes external configuration files (JSON, YAML, TOML) requiring validation and parsing before agent instantiation
- Multiple agent types share common initialization patterns involving file system access, JSON parsing, and configuration application
- The system exposes public contracts (OpenCodeAgent, ZedAgent, ParsedFrontmatter, CopilotToolMapping) that depend on consistent concurrency coordination during setup

## Problem Statement

Agent initialization and subagent loading involve asynchronous file system operations, JSON/YAML parsing, and configuration application that must be coordinated to ensure agents are fully configured before use. Without standardized concurrency coordination points, initialization sequences may exhibit race conditions, incomplete configuration states, or inconsistent agent behavior across different agent types.

## Decision

1. SHOULD: Agent implementations SHOULD validate parsed configuration data using security.input_validation patterns before applying configuration

## Policy Block

- SHOULD Agent implementations SHOULD validate parsed configuration data using security.input_validation patterns before applying configuration

In scope:
- All agent implementations in src/agents/ (OpenCodeAgent, ZedAgent, AgentsMdAgent)
- Subagent utility modules (SubagentsUtils, SubagentsProcessor)
- Configuration loading and parsing workflows involving fs/promises, js-yaml, @iarna/toml
- Public API contract exports from agent and utility modules

Out of scope:
- Runtime agent execution after initialization is complete
- Synchronous utility functions that do not involve file system or configuration operations
- Test utilities that mock or stub coordination points
- Third-party library internal concurrency mechanisms

Exceptions:
- EXC-001: Agent implementations require hot-reload or dynamic reconfiguration capabilities

## Rationale

- The evidence shows consistent use of applyRulerConfig across OpenCodeAgent and ZedAgent (significance 0.91), indicating an established pattern for configuration coordination
- SubagentsUtils and SubagentsProcessor both use loadSubagentFile as a coordination point (significance 0.90), demonstrating a parallel pattern for subagent loading workflows
- All detected files perform JSON.parse or YAML parsing operations before coordination points, establishing a clear sequencing requirement for initialization
- The pattern appears in 4 files with 90.60% confidence, providing strong evidence of an intentional architectural approach to concurrency coordination in agent initialization

## Consequences

Positive:
- Consistent initialization sequencing across all agent types reduces race conditions and configuration-related bugs
- Clear coordination points provide explicit locations for debugging initialization issues and adding instrumentation
- Standardized patterns simplify onboarding for developers implementing new agent types
- Explicit coordination points enable future optimization opportunities (e.g., parallel initialization, caching) without breaking existing contracts

Negative:
- Additional coordination points introduce slight initialization latency compared to uncoordinated approaches
- Rigid coordination requirements may complicate hot-reload or dynamic reconfiguration scenarios
- Developers must understand and correctly implement coordination patterns, increasing cognitive load for agent development
- Coordination point changes require coordinated updates across multiple agent implementations

## Alternatives

- Use constructor-based synchronous initialization without explicit coordination points (rejected)
  Rejected because: Agent initialization requires asynchronous file system operations (fs/promises) and parsing, which cannot be performed synchronously in constructors without blocking
  When valid: Only applicable for agents with purely in-memory configuration that requires no I/O
- Implement per-agent custom initialization patterns without standardization (rejected)
  Rejected because: Lack of standardization increases maintenance burden, makes initialization behavior unpredictable across agent types, and complicates testing and debugging
  When valid: May be acceptable for experimental or prototype agents not intended for production use
- Use dependency injection framework to manage initialization lifecycle (deferred)
  Rejected because: Would require significant refactoring of existing agent implementations and introduce external framework dependency
  When valid: Could be reconsidered if the system scales to dozens of agent types requiring complex dependency graphs

## Risks

- Coordination points become performance bottlenecks if initialization involves expensive operations or many agents
  Mitigation: Profile initialization performance, implement caching for parsed configuration, consider parallel initialization for independent agents
  Owner: engineering team
- Future agent types may have initialization requirements incompatible with current coordination patterns
  Mitigation: Document extension points for custom coordination, maintain policy exception process for justified deviations, review pattern quarterly
  Owner: architecture team
- Implicit dependencies between coordination points may create subtle ordering bugs
  Mitigation: Document coordination point dependencies explicitly, add initialization sequence validation in tests, use TypeScript types to enforce ordering
  Owner: engineering team

## Implementation Notes

- When implementing new agent types, follow the pattern in OpenCodeAgent.ts and ZedAgent.ts: parse configuration files first, then call applyRulerConfig before exposing the agent instance
- For subagent loading, use loadSubagentFile from SubagentsUtils.ts as the single entry point for file access, ensuring consistent error handling and validation
- Add TypeScript return type annotations to coordination functions to make initialization sequencing explicit and type-safe
- Consider adding initialization state tracking (e.g., isInitialized flag) to agent classes to prevent premature use before coordination points complete

## Continuation Context


Verify commands:
- grep -r 'applyRulerConfig' src/agents/ | wc -l  # Should match number of agent implementations
- grep -r 'loadSubagentFile' src/core/Subagents*.ts | grep -v 'export' | wc -l  # Should show usage in coordination contexts
- grep -r 'JSON.parse.*await.*readFile' src/agents/ | wc -l  # Should show parsing before coordination

Accept when:
- All agent implementations in src/agents/ use applyRulerConfig as a coordination point after configuration parsing
- All subagent loading workflows use loadSubagentFile for file system access and parsing
- No public API contracts are exported before their respective coordination points complete

## Enforcement

- Verified by: Code review checklist requiring verification of coordination point usage in new agent implementations
- Verified by: Static analysis rules detecting async file operations not coordinated through designated functions
- Verified by: Integration tests validating initialization sequencing and configuration application
- Violation handling: Code review rejection for new agent implementations missing coordination points
- Violation handling: Required refactoring for existing agents found to bypass coordination patterns during maintenance
- Violation handling: Incident review for production issues traced to initialization race conditions
- Exception process: Submit exception request to architecture team with justification for alternative coordination approach
- Exception process: Document exception rationale and alternative coordination mechanism in agent module comments
- Exception process: Add exception to policy_exceptions section with approval record and monitoring requirements