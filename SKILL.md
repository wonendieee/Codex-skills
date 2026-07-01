---
name: windows-powershell-safety
description: "Mandatory prerequisite before Codex runs commands on Windows through PowerShell or prepares command-line work in Windows projects. Use for every Windows/PowerShell command-line task, especially when output may include Chinese or other UTF-8 text, when invoking Node/npm/npx/pnpm/yarn wrappers, when handling Windows paths, when reading or writing text with encodings, when considering parallel shell reads/searches, or when diagnosing sandbox, ACL, network, execution-policy, or permission failures."
---

# Windows PowerShell Safety

## Core Rule

Before any non-trivial Windows PowerShell command, assume encoding, script wrappers, paths, and sandbox permissions can fail in Windows-specific ways. Build the command so those failures are avoided or diagnosed directly.

## Command Preamble

Use this preamble for commands that may read or print non-ASCII text, invoke native tools, or inspect files with Chinese/CJK content:

```powershell
$OutputEncoding = [System.Text.UTF8Encoding]::new($false)
[Console]::OutputEncoding = $OutputEncoding
[Console]::InputEncoding = $OutputEncoding
```

When reading text files, add explicit encodings where the cmdlet supports them:

```powershell
Get-Content -LiteralPath 'C:\path\file.txt' -Encoding UTF8
```

For Python child processes that may print Unicode, prefer setting UTF-8 mode in the same command:

```powershell
$env:PYTHONUTF8 = '1'
python script.py
```

## Node And Script Wrappers

PowerShell may resolve `npx`, `npm`, `pnpm`, and `yarn` to blocked `.ps1` shims. Call the Windows command wrappers instead:

```powershell
npx.cmd --yes <package-or-command>
npm.cmd run <script>
pnpm.cmd <command>
yarn.cmd <command>
```

Do not download or execute third-party packages just to answer whether something exists. Prefer web search or official indexes first. If external code execution is genuinely required, request approval and state the network and local execution risk plainly.

## Paths And Files

Use absolute paths and `-LiteralPath` for user, repo, and generated paths. This avoids wildcard expansion, bracket parsing, and accidental path interpretation.

```powershell
Get-ChildItem -LiteralPath 'C:\Users\99733\.codex'
Remove-Item -LiteralPath $target -Recurse
```

Before recursive delete or move operations, verify the resolved absolute target remains inside the intended workspace. Never compose destructive filesystem operations across shells.

## Parallel PowerShell Guardrails

Avoid parallel PowerShell subprocess fan-out for routine file reads, searches, and diagnostics. Prefer one focused command, such as a single `rg` invocation or one `Get-Content -LiteralPath ... -Encoding UTF8`, because each `exec_command` runs as an independently sandboxed process.

If parallel shell calls are genuinely useful, each command must be independently safe:

- Include the UTF-8 preamble when output may include non-ASCII text.
- Use absolute paths and `-LiteralPath` where available.
- Avoid package-manager shims, network access, writes, and permission-changing operations.
- Keep each command useful even if the other parallel commands fail.

If any parallel PowerShell subprocess is rejected, access denied, blocked by execution policy, or otherwise sandbox-denied, treat the whole parallel batch as partially failed. Do not infer success from the commands that happened to complete. Stop issuing additional parallel shell calls, report the partial failure, and continue with one narrow serial command at a time. If the rejected command is required for the user's task, rerun that specific command with `sandbox_permissions: "require_escalated"` and a narrow justification.

## Shell Dialect Checks

Do not paste Bash idioms into PowerShell commands. Replace them with PowerShell equivalents or use the appropriate shell intentionally.

Common mismatches:

- `export NAME=value` -> `$env:NAME = 'value'`
- `VAR=value command` -> `$env:VAR = 'value'; command`
- `cat <<EOF` -> use `apply_patch` for file edits or a proper PowerShell here-string only when patching is impossible
- `rm -rf` -> `Remove-Item -LiteralPath ... -Recurse -Force` after path verification
- `sed -i` -> use a structured editor, `apply_patch`, or a language-specific formatter

## Sandbox And Approval Handling

When a command fails with access denied, execution policy, npm cache, registry, network, or ACL errors, identify the likely cause before retrying.

- If PowerShell blocks a `.ps1` shim, retry with the matching `.cmd` wrapper.
- If output is mojibake, retry with the UTF-8 preamble and explicit `-Encoding UTF8` where possible.
- If sandboxing or network restriction blocks an important command, rerun with `sandbox_permissions: "require_escalated"` and a narrow justification.
- If an escalation is rejected, do not work around it. Use a safer alternative or ask the user for explicit approval after explaining the risk.
- Do not request broad persistent permission changes unless the user explicitly asks for them.
- If a parallel command batch partially fails, treat the denial as a real failure signal and switch to serial diagnostics before drawing conclusions.

## Quick Checklist

Before executing, confirm:

1. UTF-8 preamble is present when text output matters.
2. `.cmd` wrappers are used for Node package-manager commands.
3. Paths are quoted and passed through `-LiteralPath` when available.
4. Bash syntax has not leaked into PowerShell.
5. Parallel PowerShell fan-out is avoided unless every subprocess is independently safe.
6. Any approval request is narrow, truthful, and tied to the user's task.
