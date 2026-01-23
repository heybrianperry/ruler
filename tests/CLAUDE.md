## ADR 1: Adopt Comprehensive Unit Testing Strategy for CI/CD Pipeline Integration

1. Implement a comprehensive unit testing framework as a foundational component of the CI/CD pipeline. All core utilities, agents, configuration parsers, and CLI features must have corresponding unit tests organized in a parallel test directory structure (tests/unit/ and tests/integration/). Tests are written in TypeScript/JavaScript using a consistent testing framework, covering critical paths including file system operations, rule processing, configuration merging, agent behaviors, and error handling scenarios. The testing strategy includes both isolated unit tests for individual components and integration tests for end-to-end workflows.

---

## ADR 2: Adopt Comprehensive Unit Testing with Mocking for Agent Implementations

1. Implement a standardized unit testing approach with mocking capabilities for all agent implementations. Each agent receives dedicated test files that use mocking frameworks to isolate the agent logic from external dependencies. This includes testing valid agent behaviors (JunieAgent, OpenCodeAgent, JulesAgent) as well as invalid agent scenarios to ensure comprehensive coverage of both happy paths and error conditions. The testing pattern is consistently applied across all agent types, creating a uniform testing architecture.