# Standardize VSCode Settings Management with Typed Public Contracts: Json Content Parsed

These rules are ALWAYS ACTIVE for all VSCode settings read/write operations in `src/vscode/settings.ts`, server configuration management involving `AugmentMcpServer` instances, type definitions exported from the types module, and all JSON parsing and validation of settings files.

### Rules

- **R-VSCODE-001** MUST: JSON content parsed from filesystem MUST be validated through JSON.parse with appropriate error handling.
- **R-VSCODE-002** MUST: All settings access outside `src/vscode/settings.ts` MUST use `readVSCodeSettings` or `writeVSCodeSettings` public contract functions exclusively.
- **R-VSCODE-003** MUST: All JSON.parse operations in the settings module MUST be wrapped in try-catch blocks with appropriate error logging and fallback to default `VSCodeSettings` structure.
- **R-VSCODE-004** MUST: All new server configuration types MUST extend or implement the `AugmentMcpServer` interface for type compatibility.
- **R-VSCODE-005** MUST: Cache updates MUST use the `existingServerMap.set(server.name, server)` pattern to ensure name-based keying consistency.
- **R-VSCODE-006** SHOULD: Use `transformRulerToAugmentMcp` when converting between internal ruler format and external MCP server configuration format.

### Verify

```bash
# Verify all settings access outside settings.ts uses public contracts
grep -r 'readVSCodeSettings\|writeVSCodeSettings' src/ --include='*.ts' | grep -v 'src/vscode/settings.ts' | wc -l

# Verify all JSON.parse operations are wrapped in error handling
grep -r 'JSON\.parse' src/vscode/settings.ts | grep -c 'try\|catch'

# Verify TypeScript strict mode compilation passes
tsc --noEmit --strict src/vscode/settings.ts
```

**Accept when:**
- All settings access outside `src/vscode/settings.ts` uses `readVSCodeSettings` or `writeVSCodeSettings` functions
- All JSON.parse operations in settings module are wrapped in error handling blocks
- TypeScript compilation passes with strict mode enabled for settings module
- No direct `fs.readFileSync` or `fs.writeFileSync` calls bypass public contracts in settings-related code

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler strict mode checks during CI build are mandatory. ESLint rules prohibiting direct filesystem access in non-settings modules are mandatory. Code review checklist requiring use of public contracts for settings access is mandatory.
</enforcement>