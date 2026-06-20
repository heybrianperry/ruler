# Standardize VSCode Settings Management with Typed Public Contracts: Settings Data Structures

These rules are ALWAYS ACTIVE for all VSCode settings read/write operations in `src/vscode/settings.ts`, server configuration management involving `AugmentMcpServer` instances, type definitions exported from the `../types` module, and JSON parsing/validation of settings files.

### Rules

- **R-SETTINGS-001** MUST: Settings data structures MUST conform to the `VSCodeSettings` and `AugmentMcpServer` type definitions.
- **R-SETTINGS-002** MUST: All settings access outside `src/vscode/settings.ts` MUST use `readVSCodeSettings` or `writeVSCodeSettings` functions exclusively.
- **R-SETTINGS-003** MUST: All `JSON.parse` operations in the settings module MUST be wrapped in try-catch blocks with appropriate error logging and fallback to default `VSCodeSettings` structure.
- **R-SETTINGS-004** MUST: Cache updates MUST follow the `existingServerMap.set(server.name, server)` pattern, ensuring name-based keying consistency.
- **R-SETTINGS-005** SHOULD: Use `transformRulerToAugmentMcp` when converting between internal ruler format and external MCP server configuration format.
- **R-SETTINGS-006** MUST: All new server configuration types MUST extend or implement the `AugmentMcpServer` interface for type compatibility.

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
- All `JSON.parse` operations in settings module are wrapped in error handling blocks
- TypeScript compilation passes with strict mode enabled for settings module

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler strict mode checks during CI build, ESLint rules prohibiting direct `fs.readFileSync`/`writeFileSync` in non-settings modules, and code review checklists are mandatory enforcement gates.
</enforcement>