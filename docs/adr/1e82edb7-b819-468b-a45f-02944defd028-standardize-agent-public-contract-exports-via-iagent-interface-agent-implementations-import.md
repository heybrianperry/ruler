# Standardize Agent Public Contract Exports via IAgent Interface: Agent Implementations Import

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations and agent-related utility modules within the codebase.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, CopilotAgent, WindsurfAgent, PiAgent, AmpAgent, TraeAgent, WarpAgent) that require consistent public interfaces for interoperability
- Agent modules export typed contracts (api.public.contracts) to enable type-safe integration with core systems like UnifiedConfigLoader, config-utils, and MCP capabilities
- The IAgent interface and AbstractAgent/AgentsMdAgent base classes serve as the architectural boundary between agent implementations and the configuration/orchestration layer
- Agent implementations integrate with external configuration formats (TOML, JSON) and require standardized input validation patterns (parseTOML, JSON.parse) and concurrency models (applyRulerConfig)
- The pattern emerged across 20 files with 90.30% confidence, indicating strong architectural consistency in how agent contracts are defined and exposed

## Problem Statement

Without standardized public contract exports, agent implementations risk interface fragmentation, making it difficult to maintain type safety, enable polymorphic agent handling, and ensure consistent integration with configuration loaders, MCP capability detection, and orchestration utilities. The absence of a unified contract pattern would require custom integration logic for each agent type, increasing maintenance burden and reducing system extensibility.

## Decision

1. SHOULD: Agent implementations SHOULD import core dependencies (fs, path, fs/promises) and domain interfaces (IAgent, FileSystemUtils) using relative paths to maintain module boundaries

## Policy Block

- SHOULD Agent implementations SHOULD import core dependencies (fs, path, fs/promises) and domain interfaces (IAgent, FileSystemUtils) using relative paths to maintain module boundaries

In scope:
- All TypeScript modules in src/agents/ directory that implement agent behavior
- Core utility modules (src/core/UnifiedConfigLoader.ts, src/core/config-utils.ts) that load or transform agent configurations
- MCP integration modules (src/mcp/capabilities.ts) that query agent capabilities
- Agent utility functions (src/agents/agent-utils.ts) that operate polymorphically on agent instances

Out of scope:
- Test fixtures or mock agent implementations used exclusively in test suites
- Internal helper classes that are not exposed as public agent contracts
- Configuration type definitions (UnifiedConfigTypes.ts) that describe data structures rather than behavioral contracts

Exceptions:
- EXC-001: An agent implementation is experimental or deprecated and explicitly marked as non-standard in its module documentation

## Rationale

- The IAgent interface provides a stable architectural boundary that decouples agent implementations from orchestration logic, enabling polymorphic agent handling across 20+ files
- Standardized public contracts enable type-safe integration with UnifiedConfigLoader, MCP capabilities detection (getAgentMcpCapabilities, filterMcpConfigForAgent), and agent utility functions (getAgentOutputPaths, mapRawAgentConfigs)
- Consistent input validation patterns (JSON.parse, parseTOML) across agent implementations reduce security vulnerabilities and parsing inconsistencies when loading external configuration
- The 90.30% confidence score across 20 files demonstrates that this pattern is already the de facto standard, and formalizing it as an ADR prevents architectural drift

## Consequences

Positive:
- Enables polymorphic agent handling in core utilities, reducing code duplication and simplifying addition of new agent types
- Provides compile-time type safety for agent interactions, catching integration errors early in development
- Facilitates automated capability detection and configuration filtering through standardized contract inspection
- Reduces cognitive load for developers by establishing predictable patterns for agent implementation and integration

Negative:
- Requires all new agent implementations to conform to the IAgent interface contract, potentially constraining implementation flexibility for specialized agents
- May require refactoring of existing non-conformant agent implementations to align with the standard contract
- Introduces coupling between agent implementations and the IAgent interface definition, requiring coordinated updates when the interface evolves
- Adds overhead for simple agent implementations that might not require the full contract surface area

## Alternatives

- Use duck typing without formal interface contracts, relying on runtime property checks (rejected)
  Rejected because: Duck typing eliminates compile-time type safety, increases runtime error risk, and makes contract expectations implicit rather than explicit. The evidence shows explicit contract exports (api.public.contracts) are already the established pattern.
  When valid: Only appropriate for prototyping or experimental code not intended for production integration
- Define separate interface contracts for each agent category (CLI agents, MCP agents, etc.) rather than a unified IAgent interface (rejected)
  Rejected because: Multiple interface hierarchies would fragment the agent ecosystem, preventing polymorphic handling in utilities like config-utils and capabilities.ts. Evidence shows a single IAgent interface is used across diverse agent types.
  When valid: Could be reconsidered if agent categories diverge significantly in their behavioral contracts and shared utilities become impractical
- Use abstract base classes exclusively (AbstractAgent, AgentsMdAgent) without a separate IAgent interface (rejected)
  Rejected because: Abstract classes enforce implementation inheritance rather than contract compliance, limiting flexibility for agents that need different base implementations. Evidence shows IAgent interface is imported independently of base classes.
  When valid: Appropriate for agents that share substantial implementation logic and can commit to a single inheritance hierarchy

## Risks

- Interface evolution may break existing agent implementations if contract changes are not backward compatible
  Mitigation: Use semantic versioning for IAgent interface changes, provide deprecation warnings for breaking changes, and maintain adapter patterns for legacy agents during transition periods
  Owner: Architecture team and agent module maintainers
- Over-specification of the IAgent contract may constrain innovation in agent implementations or force unnecessary method implementations
  Mitigation: Keep the IAgent interface minimal and focused on essential integration points. Use optional methods or capability flags for specialized behaviors. Review interface additions through architecture review process.
  Owner: Engineering team
- Inconsistent input validation implementations across agents may introduce security vulnerabilities despite standardized parsing functions
  Mitigation: Provide shared validation utilities that wrap JSON.parse and parseTOML with consistent error handling and security checks. Include validation compliance in code review checklist.
  Owner: Security and engineering teams

## Implementation Notes

- New agent implementations should start by importing IAgent from './IAgent' and implementing all required methods before adding agent-specific logic
- Use AbstractAgent or AgentsMdAgent base classes when agents share common functionality (configuration application, file system operations), but ensure the base class itself implements IAgent
- Wrap all external configuration parsing (JSON.parse, parseTOML) in try-catch blocks with meaningful error messages that include the configuration source path
- When implementing applyRulerConfig, ensure the method is async and handles concurrent configuration updates safely to prevent race conditions
- Export the agent class as the default export and ensure it is listed in the api.public.contracts facet for automated discovery by tools like capabilities.ts and config-utils.ts

## Continuation Context


Verify commands:
- grep -r "implements.*IAgent\|extends.*AbstractAgent\|extends.*AgentsMdAgent" src/agents/*.ts | wc -l
- grep -r "api.public.contracts=" src/agents/*.ts | wc -l
- grep -r "import.*IAgent" src/agents/*.ts src/core/*.ts src/mcp/*.ts | wc -l
- find src/agents -name "*Agent.ts" -exec grep -L "IAgent\|AbstractAgent\|AgentsMdAgent" {} \;

Accept when:
- All agent implementation files in src/agents/ export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Core utility modules (config-utils.ts, capabilities.ts, agent-utils.ts) import and depend on the IAgent interface rather than concrete agent classes
- No agent implementation files exist that lack the api.public.contracts export pattern or IAgent interface compliance
- All agent implementations that parse external configuration use standard validation patterns (JSON.parse, parseTOML) with error handling

## Enforcement

- Verified by: TypeScript compiler type checking enforces IAgent interface compliance at build time
- Verified by: Code review checklist includes verification of api.public.contracts export and IAgent implementation
- Verified by: Automated linting rules detect agent files that do not import or implement IAgent interface
- Verified by: CI pipeline runs grep-based verification commands to count contract-compliant agent implementations
- Violation handling: Build failures occur when agent implementations do not satisfy IAgent interface type constraints
- Violation handling: Code review blocks merge requests for new agents that lack proper contract exports or interface compliance
- Violation handling: Linting warnings are escalated to errors for agent modules missing IAgent imports or implementations
- Violation handling: Quarterly architecture audits identify and prioritize refactoring of non-compliant legacy agent implementations
- Exception process: Submit exception request to architecture review board with justification for why IAgent contract cannot be satisfied
- Exception process: Document the exception in the agent module header with EXC-001 reference and migration plan
- Exception process: Exceptions are reviewed quarterly and must demonstrate progress toward contract compliance or provide updated justification
- Exception process: Experimental agents may receive temporary exceptions (max 2 quarters) before requiring full compliance or deprecation