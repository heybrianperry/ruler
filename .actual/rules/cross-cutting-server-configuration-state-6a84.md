# Standardize VSCode Settings Management with Typed Public Contracts: Server Configuration State

These rules are ALWAYS ACTIVE for all VSCode settings read/write operations in `src/vscode/settings.ts`, server configuration management involving `AugmentMcpServer` instances, type definitions exported from the types module, and JSON parsing/validation of settings files.

### Rules

- **R-VSCODE-001** MUST: Server configuration state MUST be cached using Map data structures (existingServerMap.set) to maintain in-memory consistency between persistent storage and runtime state.
- **R-VSCODE-002** MUST: All settings access outside `src/vscode/settings.ts` MUST use the public contract functions `readVSCodeSettings` or `writeVSCodeSettings` rather than direct filesystem operations.
- **R-VSCODE-003** MUST: All `JSON.parse` operations in the settings module MUST be wrapped in try-catch blocks with appropriate error logging and fallback to default `VSCodeSettings` structure.
- **R-VSCODE-004** MUST: All new server configuration types MUST extend or implement the `AugmentMcpServer` interface for type compatibility.
- **R-VSCODE-005** SHOULD: Use `transformRulerToAugmentMcp` when converting between internal ruler format and external MCP server configuration format.

### Verify

```bash
# Check that settings access outside settings.ts uses public contracts
grep -r 'readVSCodeSettings\|writeVSCodeSettings' src/ --include='*.ts' | grep -v 'src/vscode/settings.ts' | wc -l

# Verify JSON.parse operations are wrapped in error handling
grep -r 'JSON\.parse' src/vscode/settings.ts | grep -c 'try\|catch'

# Verify TypeScript strict mode compilation
tsc --noEmit --strict src/vscode/settings.ts
```

**Accept when:**
- All settings access outside `src/vscode/settings.ts` uses `readVSCodeSettings` or `writeVSCodeSettings` functions
- All `JSON.parse` operations in the settings module are wrapped in error handling blocks
- TypeScript compilation passes with strict mode enabled for the settings module
- No direct `fs.readFileSync` or `fs.writeFileSync` calls exist in non-settings modules for configuration access

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler strict mode checks during CI build, ESLint rules prohibiting direct filesystem access in non-settings modules, and code review checklists are mandatory enforcement gates.
</enforcement>