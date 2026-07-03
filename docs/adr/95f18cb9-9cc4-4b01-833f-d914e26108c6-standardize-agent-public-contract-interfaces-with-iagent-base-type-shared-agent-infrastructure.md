# Standardize Agent Public Contract Interfaces with IAgent Base Type: Shared Agent Infrastructure

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent implementations and agent-related utilities within the codebase.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, WindsurfAgent, PiAgent, AmpAgent, TraeAgent, WarpAgent) that require a consistent interface for interoperability
- Agent implementations need to integrate with shared infrastructure including configuration loading (UnifiedConfigLoader), MCP capabilities (mcp/capabilities.ts), and utility functions (agent-utils.ts)
- Configuration sources vary across agents (JSON for OpenCodeAgent and AmazonQCliAgent, TOML for MistralVibeAgent and CodexCliAgent) requiring standardized parsing and validation patterns
- The IAgent interface and AbstractAgent base class serve as the contract boundary between agent implementations and the broader system, enabling polymorphic agent handling
- Evidence shows 21 files with consistent import patterns of './IAgent' and './AbstractAgent', indicating widespread adoption of contract-based agent architecture

## Problem Statement

Without a standardized public contract interface for agent implementations, the system would face inconsistent agent behavior, duplicated configuration logic, inability to generically process agents through utilities like getAgentOutputPaths and filterMcpConfigForAgent, and fragile coupling between agent implementations and infrastructure components. The pattern addresses the need for a stable API boundary that allows agent implementations to vary independently while maintaining system-wide interoperability.

## Decision

1. SHOULD: Shared agent infrastructure (capabilities, config-utils, agent-utils) SHOULD depend only on IAgent interface, not concrete implementations

## Policy Block

- SHOULD Shared agent infrastructure (capabilities, config-utils, agent-utils) SHOULD depend only on IAgent interface, not concrete implementations

In scope:
- All TypeScript classes in src/agents/ directory that represent agent implementations
- Utility modules that operate on agents (agent-utils.ts, capabilities.ts, config-utils.ts)
- Configuration loading and validation logic in UnifiedConfigLoader.ts
- MCP capability detection and filtering functions

Out of scope:
- Internal implementation details of specific agents beyond the public contract
- Agent-specific configuration file formats (TOML, JSON) as long as validation occurs
- Non-agent modules that do not interact with the agent abstraction layer

Exceptions:
- EXC-001: Legacy agent implementations during migration period

## Rationale

- The pattern enables polymorphic handling of 9+ distinct agent implementations through a single interface contract, as evidenced by getAgentOutputPaths, getAgentMcpCapabilities, and filterMcpConfigForAgent utilities
- Standardized input validation through JSON.parse and parseTOML across agent configuration loaders reduces security vulnerabilities and parsing inconsistencies
- The IAgent contract serves as a stable API boundary with 21 file references, allowing agent implementations to evolve independently while maintaining system integration
- Confidence of 89.81% across 21 files demonstrates strong pattern consistency and architectural commitment to contract-based agent design

## Consequences

Positive:
- Enables addition of new agent implementations without modifying shared infrastructure code
- Provides type safety and compile-time verification of agent contract compliance through TypeScript interfaces
- Facilitates testing through interface mocking and reduces coupling between agent implementations
- Allows generic agent processing utilities to operate uniformly across all agent types

Negative:
- Introduces abstraction overhead and requires all agent implementations to conform to interface constraints
- May limit agent-specific optimizations that don't fit the common interface contract
- Requires coordination when evolving the IAgent interface to avoid breaking existing implementations
- Adds learning curve for developers who must understand both the interface contract and specific agent implementations

## Alternatives

- Use duck typing without explicit IAgent interface, relying on structural compatibility (rejected)
  Rejected because: TypeScript's structural typing alone provides no compile-time contract enforcement and makes the expected agent API implicit rather than explicit, reducing maintainability
  When valid: In dynamically-typed languages or prototyping phases where interface rigidity hinders exploration
- Implement agent-specific utility functions rather than generic polymorphic utilities (rejected)
  Rejected because: Would result in code duplication across 9+ agent implementations and prevent uniform agent handling in infrastructure code
  When valid: When agent behaviors are fundamentally incompatible and cannot share common abstractions
- Use composition with agent capabilities rather than inheritance from AbstractAgent (deferred)
  When valid: Could be adopted alongside IAgent interface for agents that prefer composition over inheritance patterns

## Risks

- IAgent interface evolution may break existing agent implementations if not managed through versioning or deprecation cycles
  Mitigation: Implement interface versioning strategy and maintain backward compatibility through optional properties and default implementations in AbstractAgent
  Owner: engineering team
- Agent implementations may partially implement the interface or bypass validation, creating runtime failures despite compile-time success
  Mitigation: Implement runtime contract validation in AbstractAgent constructor and add integration tests that verify full interface compliance
  Owner: engineering team
- Configuration validation patterns (JSON.parse, parseTOML) may not catch all malformed inputs, leading to runtime errors in agent initialization
  Mitigation: Add schema validation using JSON Schema or Zod after parsing, and implement comprehensive error handling in UnifiedConfigLoader
  Owner: engineering team

## Implementation Notes

- New agent implementations should extend AbstractAgent or AgentsMdAgent rather than implementing IAgent directly to inherit common functionality
- Use UnifiedConfigLoader.loadUnifiedConfig for configuration loading to ensure consistent validation and parsing patterns
- Agent utility functions should type parameters as IAgent and use type guards when agent-specific behavior is required
- Configuration validation should occur immediately after parsing (JSON.parse/parseTOML) and before passing to agent constructors

## Continuation Context


Verify commands:
- grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/*.ts | wc -l
- grep -r 'export class.*Agent' src/agents/*.ts | grep -v 'IAgent\|AbstractAgent' | wc -l
- grep -r 'JSON.parse\|parseTOML' src/agents/*.ts src/core/UnifiedConfigLoader.ts | wc -l

Accept when:
- All agent implementation files in src/agents/ export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Agent utility functions (agent-utils.ts, capabilities.ts, config-utils.ts) accept IAgent interface types in their function signatures
- Configuration loading code uses JSON.parse or parseTOML with validation before constructing agent instances

## Enforcement

- Verified by: TypeScript compiler type checking enforces IAgent interface compliance at build time
- Verified by: Code review process verifies new agent implementations extend AbstractAgent or implement IAgent
- Verified by: Integration tests validate that agent instances satisfy the full IAgent contract
- Violation handling: TypeScript compilation fails if agent classes do not implement required IAgent interface methods
- Violation handling: Code review rejects pull requests introducing agent implementations without proper interface compliance
- Violation handling: Runtime validation in AbstractAgent constructor throws errors for incomplete implementations
- Exception process: Request architecture review for agents that cannot conform to IAgent interface
- Exception process: Document technical justification for why standard contract cannot be satisfied
- Exception process: Obtain approval from engineering lead and document exception in agent implementation file