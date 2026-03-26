# Architecture Decision Records

This document contains policies derived from Architecture Decision Records (ADRs) for the ruler project.

---

## Configuration Management

### Environment Variable Access

1. All environment variable access MUST be mediated through a centralized configuration module or service
2. Components MUST NOT directly access process.env or other platform-specific environment APIs outside the configuration layer
3. Environment variable access MUST be encapsulated through typed configuration interfaces rather than direct process.env access
4. Configuration modules MUST provide type-safe accessors with explicit return types for all configuration values
5. Configuration modules MUST document all available configuration keys, their types, default values, and purposes
6. Configuration modules SHOULD provide default values for optional settings to reduce configuration burden
7. Configuration modules MAY provide default values for optional settings with clear documentation of fallback behavior
8. Configuration accessors SHOULD include validation logic to ensure values meet expected formats and constraints
9. Configuration modules MAY support multiple configuration sources (environment variables, config files, command-line arguments) with defined precedence

### Environment Variable Validation

10. All environment variables consumed by the application MUST be validated before use, checking for presence, type correctness, and format constraints
11. Required environment variables MUST be validated at application startup or module initialization, failing fast before business logic execution
12. Configuration validation failures MUST result in explicit error messages that identify the invalid variable and expected format without exposing sensitive values
13. Configuration modules MUST validate required environment variables at initialization time and fail fast with descriptive error messages
14. Validation logic SHOULD be centralized in dedicated configuration or settings modules rather than scattered across business logic
15. Environment variable values MUST NOT be used directly in security-sensitive operations (authentication, authorization, command execution) without validation and sanitization
16. Configuration modules MAY implement schema validation using libraries like Zod, Joi, or JSON Schema for complex configuration structures

### Agent Configuration

17. All agent implementations MUST initialize configuration through a centralized environment management module before executing any business logic
18. Agent configuration SHOULD support both environment variable and settings file sources with clear precedence rules
19. Configuration values SHOULD be immutable after initialization to prevent race conditions in concurrent execution contexts
20. Agent implementations MUST NOT cache environment variables in global mutable state without proper synchronization mechanisms

---

## Logging Standards

### Logging Infrastructure

21. Logging configuration MUST be centralized in a constants or configuration module to ensure consistency across the application
22. All modules performing I/O operations, external integrations, or user-facing actions MUST implement logging at appropriate severity levels
23. Modules MAY implement debug-level logging for detailed operational traces when verbose mode is enabled
24. Logging implementations MUST NOT expose sensitive information such as credentials, tokens, or personally identifiable information in log messages
25. Log messages SHOULD include contextual information such as operation type, file paths (sanitized), and relevant identifiers to aid troubleshooting

### Public API Logging

26. All public API contract implementations MUST include structured logging at entry and exit points
27. Public API contracts MAY implement debug-level logging for detailed internal state inspection when explicitly enabled
28. Public API logging MUST NOT include sensitive data such as passwords, tokens, or personally identifiable information (PII) in plain text
29. Log entries for public API contracts MUST include correlation identifiers to enable request tracing across system boundaries
30. Error conditions in public API contracts MUST be logged with ERROR level severity and include stack traces where applicable
31. Public API contracts SHOULD log performance metrics including execution time and resource utilization
32. Logging implementations SHOULD use structured logging formats (JSON) to facilitate automated parsing and analysis

### Component-Specific Logging

33. File system operations SHOULD log both successful operations and error conditions with sufficient context for debugging
34. CLI handlers SHOULD log command execution start, completion, and any errors encountered during processing

---

## Module Structure and Organization

35. Configuration files (ESLint, TypeScript) MUST be maintained at the project root to enforce consistent code quality across all modules
36. Module boundaries SHOULD follow single responsibility principle with clear separation between agents, CLI, utilities, and tests
37. Abstract base classes SHOULD define common interfaces and behaviors that concrete implementations extend
38. Additional library modules MAY be introduced under src/ following the established organizational pattern

---

## Testing Framework

### Test Framework and Language

39. All test suites MUST use Jest as the testing framework
40. All test files MUST use TypeScript with the .test.ts file extension
41. Test code MUST leverage TypeScript's type system to ensure type safety in test assertions and mocks
42. Unit tests MUST be written in TypeScript with .ts extension

### Test File Organization

43. All test files MUST be placed within the tests/ directory at the project root
44. Test files MUST use the naming pattern {feature-name}.test.{ext} where feature-name uses kebab-case and clearly describes the component or feature under test
45. Unit tests MUST be organized under tests/unit/ with subdirectories matching the source code structure (e.g., tests/unit/agents/, tests/unit/mcp/)
46. Integration tests MUST be placed in tests/integration/ and end-to-end tests MUST be placed in tests/e2e/
47. Test files MUST be organized into appropriate directories: tests/unit/ for unit tests, tests/integration/ for integration tests, and tests/e2e/ for end-to-end tests
48. Test file names SHOULD include the test type as a suffix when it provides clarity (e.g., .integration.test.ts, .e2e.test.ts)
49. Test files SHOULD be named to reflect the specific behavior or scenario being tested rather than just the class name (e.g., mcp-backup-prevention.test.ts instead of mcp.test.ts)
50. Test files MAY include additional context in the name to distinguish between different test aspects of the same component (e.g., claude-http-type.test.ts, gemini-no-backup.test.ts)
51. Test file names SHOULD mirror the source file being tested (e.g., ClaudeAgent.test.ts for ClaudeAgent.ts)
52. Test files MAY include descriptive suffixes beyond .test.ts to indicate specific test scenarios (e.g., -anchored.test.ts, -no-backup.test.ts)

### Test Types and Scope

53. Unit tests SHOULD focus on testing individual components in isolation with mocked dependencies
54. Integration tests SHOULD verify interactions between multiple components or external systems

---

## Test Execution

### Parallel Test Execution

55. Unit tests in CI/CD pipelines MUST be designed to support parallel execution without shared mutable state
56. Test frameworks MUST be configured to execute test suites concurrently when running in CI/CD environments
57. Test files MUST be isolated such that execution order does not affect test outcomes
58. CI/CD pipeline configurations SHOULD specify the degree of parallelism based on available compute resources
59. Test suites SHOULD be balanced across parallel workers to optimize total execution time
60. Tests MUST NOT depend on execution order or timing of other tests in the suite
61. Integration tests MAY use sequential execution when external resource constraints require it

---

## Test Mocking Patterns

### Mock-Based Isolation

62. All unit tests in the CI/CD pipeline MUST mock external dependencies including network calls, file system operations, and third-party services to ensure test isolation and deterministic behavior
63. Integration tests MUST use mocking for components outside the scope of the integration being tested to maintain test reliability and execution speed
64. Test files SHOULD use a consistent mocking framework or library across the test suite to reduce cognitive overhead and improve maintainability
65. Mocks SHOULD be reset or cleared between test cases to prevent test pollution and ensure each test runs in a clean state
66. Mock implementations SHOULD verify that expected interactions occurred (e.g., function calls, parameter values) to validate component behavior
67. Tests MAY use spy patterns to observe real implementations while still maintaining the ability to verify interactions
68. Tests MUST NOT rely on real external services or network connectivity that could cause non-deterministic failures in the CI/CD pipeline

---

## Agent Testing Standards

### Agent Unit Test Organization

69. All agent implementations MUST have corresponding unit test files located in tests/unit/agents/ directory
70. Agent unit test files MUST follow the naming convention {AgentName}.test.ts
71. Each agent implementation MUST have a corresponding unit test file following the pattern tests/unit/agents/{AgentName}.test.ts

### Agent Unit Test Requirements

72. All agent unit tests MUST use mocking frameworks to isolate the agent under test from external dependencies
73. Agent unit tests MUST NOT make real network calls, file system operations, or database queries
74. Agent unit tests SHOULD cover core agent functionality including initialization, message handling, and state management
75. Agent test suites SHOULD achieve at least 80% code coverage for the agent class under test
76. Mock implementations SHOULD verify interaction patterns (method calls, argument passing) rather than just stubbing return values
77. Test setup SHOULD use factory functions or test fixtures to create consistent mock configurations across test cases
78. Tests MAY use spy objects to verify internal state changes when behavior verification is insufficient
79. Test files SHOULD maintain consistent structure and organization patterns across all agent implementations
80. Shared testing utilities and mocks SHOULD be extracted to common test helpers to avoid duplication
81. Agent tests MAY include integration-style tests within the unit test suite if they remain fast and isolated