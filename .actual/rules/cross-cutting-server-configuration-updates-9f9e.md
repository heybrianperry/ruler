# Standardize VSCode Settings Management with Typed Public Contracts: Server Configuration Updates

These rules are ALWAYS ACTIVE for all modules that read or write VSCode extension settings, server configuration management, configuration transformation and migration logic, and settings validation and type checking operations.

### Rules

- **R-VSCODE-CONFIG-001** SHOULD: Server configuration updates SHOULD utilize the cache layer (existingServerMap) to minimize filesystem I/O.

### Verify

```bash
# Check for direct JSON.parse operations on settings files outside the settings module
grep -r 'JSON\.parse.*settings' --include='*.ts' --exclude='src/vscode/settings.ts' | grep -v 'readVSCodeSettings' && echo 'FAIL: Direct JSON parsing found' || echo 'PASS'

# Check for direct fs usage in settings-related code outside the settings module
grep -r "require.*'fs'" --include='*.ts' | grep -i settings | grep -v 'src/vscode/settings.ts' && echo 'FAIL: Direct fs usage in settings code' || echo 'PASS'

# Verify TypeScript strict mode compilation
npx tsc --noEmit --strict && echo 'PASS: Type checking passed' || echo 'FAIL: Type errors detected'
```

**Accept when:**
- No direct JSON.parse operations on settings files exist outside src/vscode/settings.ts
- All settings access uses typed contracts (VSCodeSettings, AugmentMcpServer) with no 'any' types
- TypeScript compilation passes with strict mode enabled for settings-related modules
- Cache layer (existingServerMap) is utilized for server configuration updates to minimize filesystem I/O

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to settings-related code. Type safety violations MUST be resolved before merge.
</enforcement>