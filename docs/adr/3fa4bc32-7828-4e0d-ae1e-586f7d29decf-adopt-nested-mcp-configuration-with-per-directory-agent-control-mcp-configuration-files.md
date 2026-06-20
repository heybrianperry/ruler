# Adopt Nested MCP Configuration with Per-Directory Agent Control: Mcp Configuration Files

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent configuration systems that support MCP (Model Context Protocol) integration with nested directory structures.

## Context

- The system manages multiple AI agents (Claude, Copilot, Windsurf) with per-agent MCP server configurations that must be independently controlled across nested directory hierarchies
- Agent configurations are distributed across project root, module, and submodule directories, each with independent .ruler/ruler.toml files defining agent enablement and MCP server definitions
- MCP server configurations include both stdio-based commands and remote URL endpoints that must be scoped to specific directory contexts without inheritance pollution
- The integration test framework validates that per-directory MCP configs are written correctly, agent-specific enablement flags are honored, and gitignore entries are updated to exclude generated configuration files
- Backup mechanisms preserve existing MCP configuration files before overwriting, ensuring recovery paths for configuration rollback

## Problem Statement

Multi-agent AI development environments require fine-grained control over which agents have MCP capabilities enabled in different parts of a monorepo or nested project structure, but a single global configuration creates conflicts when different modules need different MCP server sets or when specific agents should be disabled in certain contexts while remaining active elsewhere.

## Decision

1. MUST: MCP configuration files MUST be written to the directory level where the corresponding .ruler/ruler.toml file defines agent and MCP server settings

## Policy Block

- MUST MCP configuration files MUST be written to the directory level where the corresponding .ruler/ruler.toml file defines agent and MCP server settings

In scope:
- All AI agent configuration systems supporting MCP protocol integration
- Projects with nested directory structures requiring independent agent configurations
- MCP server definitions using stdio commands or remote URLs
- Agent identifiers: claude, copilot, windsurf, and future MCP-capable agents
- Configuration files: .ruler/ruler.toml, .ruler/AGENTS.md, and agent-specific MCP JSON files

Out of scope:
- Non-MCP agent configuration mechanisms
- Global agent settings that do not vary by directory context
- Agent instruction content within AGENTS.md files (content is out of scope; only file presence matters)
- MCP server implementation details or protocol specifications

Exceptions:
- EXC-001: A single-directory project with no nested modules may use global MCP configuration without per-directory isolation
- EXC-002: Legacy projects migrating to nested MCP may temporarily disable backup creation if existing configurations are already version-controlled

## Rationale

- The evidence shows explicit testing of nested directory structures (projectRoot, moduleDir, submoduleDir) with independent ruler.toml files, demonstrating intentional support for per-directory configuration isolation
- The test validates that agents.copilot.mcp.enabled=false in module directory prevents Copilot MCP file creation there while allowing it in root and submodule, proving fine-grained agent control is a core requirement
- MCP server definitions (root_stdio, module_remote, sub_stdio) are verified to remain scoped to their defining directory without leaking to parent or child contexts, indicating strict boundary enforcement
- The backup mechanism (.mcp.json.bak) and gitignore updates demonstrate production-readiness concerns for safe configuration management and version control hygiene

## Consequences

Positive:
- Different modules within a monorepo can configure different sets of MCP servers without conflicts or inheritance issues
- Teams can disable specific agents in sensitive directories while keeping them enabled elsewhere, supporting security and compliance requirements
- Configuration changes are reversible through automatic backup creation, reducing risk of accidental data loss
- Generated configuration files are automatically excluded from version control, preventing merge conflicts and repository bloat

Negative:
- Configuration complexity increases as each directory level requires independent ruler.toml maintenance and understanding of inheritance rules
- Debugging configuration issues requires examining multiple directory levels to understand which settings apply in a given context
- The lack of inheritance means common MCP servers must be duplicated across multiple ruler.toml files if needed in multiple directories
- Testing nested configuration behavior requires complex integration test fixtures with multi-level directory structures and state management

## Alternatives

- Use a single global MCP configuration file at project root with all agents and servers defined centrally (rejected)
  Rejected because: Cannot support per-directory agent enablement or module-specific MCP server sets, which the evidence shows are explicit requirements (e.g., copilot disabled in module but enabled in root)
  When valid: Single-directory projects with uniform agent requirements across all code
- Implement configuration inheritance where child directories merge parent MCP servers with local definitions (rejected)
  Rejected because: Test assertions explicitly verify that servers do NOT propagate (e.g., root_stdio must not appear in module MCP config), indicating inheritance would violate isolation requirements
  When valid: Projects where MCP servers represent shared infrastructure that should be available throughout the tree
- Use environment variables or command-line flags to control MCP configuration instead of per-directory files (rejected)
  Rejected because: Would not support persistent, directory-scoped configuration that different team members and CI environments can rely on without runtime coordination
  When valid: Development environments where configuration is ephemeral and developer-specific rather than project-structural

## Risks

- Configuration drift where different directories have inconsistent agent enablement patterns that developers are unaware of, leading to unexpected agent behavior
  Mitigation: Implement configuration validation tooling that reports all active agents and MCP servers across the directory tree; add documentation generation for configuration topology
  Owner: Engineering team
- Backup files accumulate over time consuming disk space and creating confusion about which configuration is current
  Mitigation: Implement backup rotation policy (keep only N most recent backups); add cleanup commands to development tooling; document backup file naming conventions
  Owner: DevOps team
- Integration test complexity grows as more agents and nesting levels are added, making test maintenance difficult and execution slow
  Mitigation: Extract test fixture creation into reusable harness functions; parameterize tests by agent and directory depth; implement parallel test execution for independent scenarios
  Owner: QA and testing team

## Implementation Notes

- Use the applyAllAgentConfigs function with nested=true flag to enable per-directory MCP configuration mode
- Structure test fixtures with setupTestProject/teardownTestProject harness functions to ensure clean state between test runs
- Implement getNativeMcpPath helper to resolve agent-specific MCP configuration file locations across different directory contexts
- Parse MCP JSON files using extractServers helper that handles both mcpServers and servers key variations for cross-agent compatibility
- Validate gitignore updates by checking for both the MCP config path and its .bak backup path using relative paths from project root

## Continuation Context


Verify commands:
- grep -r 'applyAllAgentConfigs.*nested.*true' tests/integration/ || echo 'No nested MCP tests found'
- find . -name 'ruler.toml' -exec grep -l '\[agents\..*\.mcp\]' {} \; | wc -l
- find . -name '.mcp.json' -o -name '.mcp.json.bak' | while read f; do jq -e '.mcpServers // .servers' "$f" > /dev/null && echo "$f: valid" || echo "$f: invalid"; done

Accept when:
- Integration tests validate that MCP configuration files are created only in directories where the corresponding agent has mcp.enabled=true in ruler.toml
- Tests confirm that MCP server definitions remain isolated to their defining directory without appearing in parent or child directory configurations
- Backup files with .bak extension are created before overwriting existing MCP configurations, and both original and backup paths are added to .gitignore

## Enforcement

- Verified by: Integration test suite using describe/it framework with beforeAll/afterAll lifecycle validation
- Verified by: Automated verification of MCP JSON file structure and server key presence using extractServers helper
- Verified by: File system assertions checking for expected MCP config presence/absence based on agent enablement flags
- Violation handling: Integration test failures block PR merges when nested MCP propagation behavior deviates from expected patterns
- Violation handling: Configuration validation errors are logged with specific directory and agent context to aid debugging
- Violation handling: Backup restoration procedures are documented for recovery when MCP configuration corruption is detected
- Exception process: Exceptions to nested configuration requirements must be documented in project-level README with architectural justification
- Exception process: Alternative configuration approaches require approval from project architect and must include migration path documentation
- Exception process: Test coverage waivers for new agents require demonstration that existing nested propagation patterns are preserved