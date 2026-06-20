# Standardize VSCode Settings Management with Typed Public Contracts: Vscode Settings Access

These rules are ALWAYS ACTIVE for all VSCode settings access operations in src/vscode/settings.ts and any code that reads or writes server configuration state.

### Rules

- **R-VSCODE-001** MUST: All VSCode settings access MUST use the exported public contracts (readVSCodeSettings, writeVSCodeSettings) rather than direct filesystem operations.
- **R-VSCODE-002** MUST: All JSON.parse operations in the settings module MUST be wrapped in try-catch blocks with appropriate error logging and fallback to default VSCodeSettings structure.
- **R-VSCODE-003** MUST: All new server configuration types MUST extend or implement the AugmentMcpServer interface for type compatibility.
- **R-VSCODE-004** MUST: Cache updates MUST use the existingServerMap.set(server.name, server) pattern to ensure name-based keying consistency.
- **R-VSCODE-005** SHOULD: Use transformRulerToAugmentMcp when converting between internal ruler format and external MCP server configuration format.
- **R-VSCODE-006** SHOULD: Import settings operations exclusively through public contracts: import { readVSCodeSettings, writeVSCodeSettings } from './vscode/settings'.

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
- All settings access outside src/vscode/settings.ts uses readVSCodeSettings or writeVSCodeSettings functions
- All JSON.parse operations in settings module are wrapped in error handling blocks
- TypeScript compilation passes with strict mode enabled for settings module
- No direct fs.readFileSync or fs.writeFileSync calls exist in non-settings modules for configuration access

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to VSCode settings access patterns.
</enforcement>