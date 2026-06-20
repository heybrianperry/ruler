# Use Set-Based Message Queue Boundaries for Agent and Skill Processing: Agent Skill Processing

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent and skill processing components that handle configuration, discovery, and propagation workflows.

## Context

- The codebase implements agent configuration management (FirebenderAgent) and skill discovery/propagation (SkillsProcessor) as public API contracts exposed through TypeScript modules
- Both components process collections of items (configuration keys, skill targets) where uniqueness and deduplication are critical to prevent duplicate processing or configuration conflicts
- The system uses Set data structures as in-memory message queue boundaries to accumulate and deduplicate items during asynchronous processing workflows
- Evidence shows explicit Set.add() operations (seen.add(key), targets.add(target)) coordinating state across concurrent operations like applyRulerConfig, loadExistingConfig, discoverSkills, and propagateSkills

## Problem Statement

When processing agent configurations and skill propagation across multiple asynchronous operations, the system must prevent duplicate entries, maintain processing boundaries, and coordinate state without external message queue infrastructure. The challenge is ensuring idempotent operations and clear boundaries between processing stages while keeping the implementation lightweight and embedded within the application logic.

## Decision

1. MUST: Agent and skill processing components MUST use Set data structures to establish message queue boundaries for deduplication of configuration keys and processing targets

## Policy Block

- MUST Agent and skill processing components MUST use Set data structures to establish message queue boundaries for deduplication of configuration keys and processing targets

In scope:
- FirebenderAgent configuration management (applyRulerConfig, loadExistingConfig, saveConfig, handleMcpConfiguration)
- SkillsProcessor discovery and propagation (discoverSkills, getSkillsGitignorePaths, propagateSkills, propagateSkillsForClaude, propagateSkillsForCodex)
- Any public API contract that processes collections requiring deduplication
- Asynchronous operations coordinating state across multiple concurrent functions

Out of scope:
- External message queue systems (RabbitMQ, Kafka, SQS)
- Database-backed queue implementations
- Persistent queue storage mechanisms
- Inter-process or distributed queue coordination
- Components that require ordered processing guarantees beyond Set iteration order

## Rationale

- The evidence shows consistent use of Set.add() operations (seen.add(key), targets.add(target)) across two independent modules, indicating an established pattern for managing processing boundaries
- Set data structures provide O(1) deduplication and membership testing, making them efficient for coordinating concurrent operations without external dependencies
- The pattern appears in public API contracts (FirebenderAgent, discoverSkills, propagateSkills) that serve as integration points, suggesting this is an intentional architectural boundary mechanism
- Using in-memory Sets as lightweight message queue boundaries reduces infrastructure complexity while maintaining clear separation between processing stages

## Consequences

Positive:
- Automatic deduplication prevents duplicate processing of configuration keys and skill targets without additional validation logic
- Lightweight in-memory implementation eliminates external message queue infrastructure dependencies and operational complexity
- Clear processing boundaries improve code readability and make concurrent operation coordination explicit
- O(1) Set operations provide efficient performance for typical workload sizes in agent and skill processing

Negative:
- In-memory Sets do not persist across process restarts, requiring reprocessing on failure or restart scenarios
- Set iteration order is not guaranteed to be stable across JavaScript engine versions, potentially affecting processing order
- Memory usage scales linearly with the number of unique items, which could become problematic for very large configuration or skill sets
- Lack of external queue visibility makes monitoring and debugging processing boundaries more difficult compared to dedicated queue infrastructure

## Alternatives

- Use external message queue system (RabbitMQ, Redis, SQS) for processing boundaries (rejected)
  Rejected because: Adds significant infrastructure complexity and operational overhead for a pattern that currently operates efficiently in-memory with small to medium workload sizes. The evidence shows no distributed processing requirements that would justify external queue infrastructure.
  When valid: Valid when processing must survive process restarts, when distributed processing across multiple nodes is required, or when workload sizes exceed available memory
- Use Array with manual deduplication logic (filter, includes checks) (rejected)
  Rejected because: Arrays require O(n) deduplication checks, creating performance bottlenecks for concurrent operations. Set data structures provide O(1) operations and automatic deduplication, making them architecturally superior for this use case.
  When valid: Valid only when processing order must be strictly maintained and deduplication is infrequent
- Use Map data structure with boolean values for tracking processed items (deferred)
  When valid: Valid when additional metadata about processed items (timestamps, status, retry counts) needs to be tracked alongside the deduplication boundary

## Risks

- Memory exhaustion if configuration keys or skill targets grow unbounded without cleanup
  Mitigation: Implement Set size monitoring and periodic cleanup for long-running processes. Consider adding maximum size limits with overflow handling for production deployments.
  Owner: engineering team
- Loss of processing state on process crash or restart requires full reprocessing
  Mitigation: Document restart behavior clearly. For critical workflows, consider periodic checkpointing or transition to persistent queue if reliability requirements increase.
  Owner: engineering team
- Non-deterministic Set iteration order may cause inconsistent processing sequences in edge cases
  Mitigation: Convert Sets to sorted arrays before iteration when processing order matters. Document that Set-based boundaries do not guarantee order.
  Owner: engineering team

## Implementation Notes

- Initialize Sets at the beginning of processing functions (e.g., const seen = new Set(), const targets = new Set()) to establish clear boundary scope
- Use descriptive variable names that indicate the boundary purpose: 'seen' for tracking processed items, 'targets' for accumulating destinations, 'pending' for queued work
- When converting Sets to arrays for iteration or output, use Array.from(set) or [...set] spread syntax for clarity
- For debugging, log Set sizes at key processing milestones to track boundary growth and identify potential memory issues early

## Continuation Context


Verify commands:
- grep -r 'seen\.add\|targets\.add' src/ --include='*.ts' | wc -l
- grep -r 'new Set<' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l
- npm test -- --grep 'deduplication|Set.*boundary' 2>/dev/null || echo 'No specific tests found'

Accept when:
- Grep commands identify at least 2 instances of Set.add() operations in agent or skill processing modules
- Code review confirms Set data structures are used for deduplication in public API contracts
- No instances of manual array-based deduplication logic (filter, includes) exist in the same processing contexts where Sets are appropriate

## Enforcement

- Verified by: Code review checklist requiring Set-based boundaries for new agent and skill processing features
- Verified by: Automated grep-based verification in CI pipeline checking for Set.add() patterns in relevant modules
- Verified by: Architecture review for new public API contracts ensuring deduplication strategy is documented
- Violation handling: Code review feedback requesting refactoring to Set-based boundaries when manual deduplication is detected
- Violation handling: Architecture review escalation for new components that introduce alternative queue boundary mechanisms without documented rationale
- Violation handling: Performance regression investigation if O(n) deduplication patterns are introduced in hot paths
- Exception process: Document specific requirements (ordering guarantees, persistence, distributed coordination) that Set-based boundaries cannot satisfy
- Exception process: Propose alternative approach with performance and complexity tradeoff analysis
- Exception process: Obtain architecture review approval before implementing alternative queue boundary mechanism