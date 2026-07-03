# Standardize VSCode Settings Management with Typed Public Contracts: Settings Modules Import

These rules are ALWAYS ACTIVE for all VSCode settings management code in `src/vscode/settings.ts` and any code that accesses server configuration state, type definitions, or persistent settings storage.

### Rules

- **R-SETTINGS-001** MUST: Settings modules MUST import core dependencies ('fs', 'path', '../types') through explicit relative or absolute paths.
- **R-SETTINGS-002** MUST: All settings access outside `src/vscode/settings.ts` MUST use the public contract functions `readVSCodeSettings` or `writeVSCodeSettings` rather than direct filesystem operations.
- **R-SETTINGS-003** MUST: All `JSON.parse` operations in the settings module MUST be wrapped in try-catch blocks with appropriate error logging and fallback to default `VSCodeSettings` structure.
- **R-SETTINGS-004** MUST: All new server configuration types MUST extend or implement the `AugmentMcpServer` interface for type compatibility.
- **R-SETTINGS-005** SHOULD: Cache updates SHOULD follow the `existingServerMap.set(server.name, server)` pattern to ensure name-based keying consistency.
- **R-SETTINGS-006** SHOULD: Use `transformRulerToAugmentMcp` when converting between internal ruler format and external MCP server configuration format.

### Verify

```bash
# Verify all settings access uses public contracts
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
- All server configuration types implement or extend `AugmentMcpServer` interface

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler strict mode checks MUST pass during CI build. ESLint rules prohibiting direct `fs.readFileSync`/`writeFileSync` in non-settings modules MUST be enforced. Code review MUST verify use of public contracts for all settings access.
</enforcement>