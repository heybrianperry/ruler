# Standardize VSCode Settings Management with Typed Public Contracts: Additional Caching Layers

These rules are ALWAYS ACTIVE for all VSCode settings management code in `src/vscode/settings.ts` and any code that reads or writes server configuration state.

### Rules

- **R-VSCODE-SETTINGS-001** MUST: Import settings operations exclusively through public contracts: `readVSCodeSettings`, `writeVSCodeSettings`, `transformRulerToAugmentMcp`, and use types `VSCodeSettings` and `AugmentMcpServer`.
- **R-VSCODE-SETTINGS-002** MUST: Wrap all `JSON.parse` operations in try-catch blocks with appropriate error logging and fallback to default `VSCodeSettings` structure.
- **R-VSCODE-SETTINGS-003** MUST: Use `existingServerMap.set(server.name, server)` pattern for cache updates, ensuring name-based keying consistency.
- **R-VSCODE-SETTINGS-004** MUST: Ensure all new server configuration types extend or implement `AugmentMcpServer` interface for type compatibility.
- **R-VSCODE-SETTINGS-005** MAY: Additional caching layers MAY be introduced for frequently accessed server configurations, provided cache invalidation is implemented on write operations.

### Verify

```bash
# Verify all settings access outside settings.ts uses public contracts
grep -r 'readVSCodeSettings\|writeVSCodeSettings' src/ --include='*.ts' | grep -v 'src/vscode/settings.ts' | wc -l

# Verify all JSON.parse operations in settings module are wrapped in error handling
grep -r 'JSON\.parse' src/vscode/settings.ts | grep -c 'try\|catch'

# Verify TypeScript compilation passes with strict mode
tsc --noEmit --strict src/vscode/settings.ts
```

**Accept when:**
- All settings access outside `src/vscode/settings.ts` uses `readVSCodeSettings` or `writeVSCodeSettings` functions
- All `JSON.parse` operations in settings module are wrapped in error handling blocks
- TypeScript compilation passes with strict mode enabled for settings module
- Cache synchronization between `existingServerMap` and persistent storage includes invalidation logic on write operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules must be checked before approving changes to settings management code.
</enforcement>