# Standardize VSCode Settings Management with Typed Public Contracts: Transformation Functions Transformrulertoaugmentmcp

These rules are ALWAYS ACTIVE for all VSCode settings read/write operations in `src/vscode/settings.ts`, server configuration management involving `AugmentMcpServer` instances, type definitions exported from the types module, and JSON parsing/validation of settings files.

### Rules

- **R-VSCODE-001** MUST: Use `readVSCodeSettings` and `writeVSCodeSettings` public contract functions for all settings access outside `src/vscode/settings.ts`.
- **R-VSCODE-002** MUST: Wrap all `JSON.parse` operations in settings module with try-catch blocks and provide appropriate error logging and fallback to default `VSCodeSettings` structure.
- **R-VSCODE-003** MUST: Ensure all new server configuration types extend or implement the `AugmentMcpServer` interface for type compatibility.
- **R-VSCODE-004** SHOULD: Use `transformRulerToAugmentMcp` transformation function when converting between internal ruler format and external MCP server configuration format.
- **R-VSCODE-005** SHOULD: Always use `existingServerMap.set(server.name, server)` pattern for cache updates, ensuring name-based keying consistency.
- **R-VSCODE-006** SHOULD: Import settings operations exclusively through public contracts: `import { readVSCodeSettings, writeVSCodeSettings } from './vscode/settings'`.

### Verify

```bash
# Verify all settings access outside settings.ts uses public contracts
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
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes to VSCode settings management code.
</enforcement>