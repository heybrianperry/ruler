# Standardize VSCode Settings Management with Typed Public Contracts: Transformation Functions Transformrulertoaugmentmcp

These rules are ALWAYS ACTIVE for all modules that read, write, or transform VSCode extension settings, including server configuration management, configuration transformation and migration logic, and settings validation operations.

### Rules

- **R-VSCODE-SETTINGS-001** SHOULD: Transformation functions (transformRulerToAugmentMcp) SHOULD be used when converting between internal and external configuration formats.
- **R-VSCODE-SETTINGS-002** MUST: All settings access MUST use typed contracts (VSCodeSettings, AugmentMcpServer) with no 'any' types.
- **R-VSCODE-SETTINGS-003** MUST: Direct JSON.parse operations on settings files MUST NOT exist outside src/vscode/settings.ts.
- **R-VSCODE-SETTINGS-004** MUST: All settings operations MUST use readVSCodeSettings/writeVSCodeSettings public contracts instead of direct filesystem access.
- **R-VSCODE-SETTINGS-005** MUST: TypeScript strict mode compilation MUST pass for all settings-related modules.
- **R-VSCODE-SETTINGS-006** SHOULD: Cache invalidation strategy SHOULD be documented and implemented to prevent stale data issues from external modifications.

### Verify

```bash
# Check for direct JSON.parse operations on settings outside the settings module
grep -r 'JSON\.parse.*settings' --include='*.ts' --exclude='src/vscode/settings.ts' | grep -v 'readVSCodeSettings' && echo 'FAIL: Direct JSON parsing found' || echo 'PASS'

# Check for direct fs module usage in settings-related code
grep -r "require.*'fs'" --include='*.ts' | grep -i settings | grep -v 'src/vscode/settings.ts' && echo 'FAIL: Direct fs usage in settings code' || echo 'PASS'

# Verify TypeScript strict mode compilation
npx tsc --noEmit --strict && echo 'PASS: Type checking passed' || echo 'FAIL: Type errors detected'
```

**Accept when:**
- No direct JSON.parse operations on settings files exist outside src/vscode/settings.ts
- All settings access uses typed contracts (VSCodeSettings, AugmentMcpServer) with no 'any' types
- TypeScript compilation passes with strict mode enabled for settings-related modules
- No direct filesystem access patterns (require 'fs') appear in settings-related code outside the designated module

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to settings-related code. Violations MUST block pull requests until resolved.
</enforcement>