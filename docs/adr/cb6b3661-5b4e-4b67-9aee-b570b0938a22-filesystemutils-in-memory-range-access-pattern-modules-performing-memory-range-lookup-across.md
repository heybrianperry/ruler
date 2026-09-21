# FileSystemUtils In-Memory Range Access Pattern: Modules Performing Memory Range Lookup Across

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core utility operations require indexing and boundary identification for range segments during file system path filtering and pattern evaluation.
- Internal modules utilize local data structures alongside core platform runtime utilities to manage block offsets and range definitions.
- Static inspection demonstrates direct linear array traversal using predicate matching for offset resolution within utility boundaries.

## Problem Statement

Utility routines managing file system pattern matching require deterministic boundary and range lookup for block structures without incurring unnecessary dependency weight or indexing overhead for localized data sets.

## Decision

1. MUST: Modules performing in-memory range lookup across internal FileSystemUtils structures MUST evaluate the cardinality of collections before adopting linear predicate searches.

## Policy Block

- MUST Modules performing in-memory range lookup across internal FileSystemUtils structures MUST evaluate the cardinality of collections before adopting linear predicate searches.

In scope:
- In-memory collection querying within internal utility modules.
- Sequential boundary evaluation for localized range structures.

Out of scope:
- Persistent data access layers requiring secondary indexing or distributed storage.
- High-throughput data pipelines operating on unbounded collections.

## Rationale

- Evidence demonstrates localized array predicate searches over range blocks within internal utility components alongside core runtime utilities.
- Linear traversal represents a minimal implementation footprint with zero memory overhead for small, localized collections.
- Because the observed usage is confined to a single utility context and relies on standard language capabilities, full architectural formalization is not warranted.

## Consequences

Positive:
- Eliminates external dependency overhead and custom data structure maintenance for small in-memory collections.
- Maintains minimal memory footprint by avoiding indexing structures or persistent cache state.
- Ensures straightforward, readable logic within internal utility modules.

Negative:
- Yields linear time complexity that degrades performance if range collection sizes increase significantly.
- Lacks centralized query abstractions across utility boundaries.

## Alternatives

- Map-based key lookup index (rejected)
  Rejected because: Constructing and maintaining a key-value hash map introduces unnecessary allocation overhead for ephemeral small collections.
  When valid: Valid when range lookups are executed repeatedly across high-cardinality collections with static keys.
- Binary search interval tree (rejected)
  Rejected because: Implementing tree structures adds architectural complexity disproportionate to the scale of local utility operations.
  When valid: Valid when querying overlapping ranges over large datasets with strict logarithmic search time requirements.

## Risks

- Performance degradation if range collection sizes expand beyond initial utility assumptions.
  Mitigation: Establish performance benchmarks and transition to indexed data structures if collection cardinality exceeds linear efficiency thresholds.
  Owner: Core Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Profile collection sizes at runtime to verify that linear scanning remains within acceptable latency budgets for utility routines.
- Preserve immutability of range collections during iteration to prevent predicate evaluation inconsistencies.

## Continuation Context


Verify commands:
- Discover the repository test execution script and run automated test suites covering utility data access routines.
- Discover and run static analysis linters to confirm adherence to architectural data access boundaries.

Accept when:
- Automated unit test suites for utility range resolution execute without failure.
- Static verification confirms data access logic does not introduce unapproved indexing dependencies into utility modules.

## Enforcement

- Verified by: Automated continuous integration pipeline test execution.
- Verified by: Peer code review for pull requests modifying utility data access patterns.
- Violation handling: Pull requests introducing unapproved data access libraries or unbounded linear scans will be blocked until remediated.
- Exception process: Exceptions for specialized data structures must be submitted via an architecture review proposal detailing cardinality and performance benchmarks.