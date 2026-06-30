# Standardize VSCode Settings Management with Typed Public Contracts: Vscode Settings Access

These rules are ALWAYS ACTIVE for all modules that read or write VSCode extension settings, server configuration management, configuration transformation and migration logic, and settings validation and type checking operations.

### Rules

- **R-VSCODE-SETTINGS-001** MUST: All VSCode settings access MUST use the typed public contracts (VSCodeSettings, AugmentMcpServer) defined in src/vscode/settings.ts rather than direct filesystem operations.

### Verify

```bash
# Check for direct JSON parsing on settings outside the settings module
grep -r 'JSON\.parse.*settings' --include='*.ts' --exclude='src/vscode/settings.ts' | grep -v 'readVSCodeSettings' && echo 'FAIL: Direct JSON parsing found' || echo 'PASS'

# Check for direct fs usage in settings-related code
grep -r "require.*'fs'" --include='*.ts' | grep -i settings | grep -v 'src/vscode/settings.ts' && echo 'FAIL: Direct fs usage in settings code' || echo 'PASS'

# Verify TypeScript strict mode compilation
npx tsc --noEmit --strict && echo 'PASS: Type checking passed' || echo 'FAIL: Type errors detected'
```

**Accept when:**
- No direct JSON.parse operations on settings files exist outside src/vscode/settings.ts
- All settings access uses typed contracts (VSCodeSettings, AugmentMcpServer) with no 'any' types
- TypeScript compilation passes with strict mode enabled for settings-related modules

<enforcement>
Clause R-VSCODE-SETTINGS-001 is verified by TypeScript strict mode compilation in CI pipeline, static analysis rules checking for direct filesystem access patterns, and code review checklist requiring typed contract usage. Violations block CI builds and pull requests until resolved. Exceptions require tech lead approval and SETTINGS-EXCEPTION tag documentation.
</enforcement>