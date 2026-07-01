# Windows PowerShell Safety

A Codex skill for safer command-line work on Windows through PowerShell.

Use this skill before running Windows or PowerShell commands, especially when a task involves Chinese or other UTF-8 output, Node package-manager wrappers, Windows paths, sandbox approvals, ACLs, network restrictions, or execution-policy errors.

## Contents

- `SKILL.md` - the skill instructions Codex reads when the skill is invoked.
- `agents/openai.yaml` - UI metadata and implicit invocation policy.

## What It Covers

- Sets UTF-8 console input and output before commands that may print non-ASCII text.
- Uses `.cmd` wrappers for `npm`, `npx`, `pnpm`, and `yarn` in PowerShell.
- Prefers `-LiteralPath` and absolute paths for Windows file operations.
- Avoids Bash syntax in PowerShell commands.
- Handles sandbox, ACL, network, and execution-policy failures through scoped approval requests.

## Install

Place this repository content under your Codex personal skills directory as a skill folder, for example:

```text
%USERPROFILE%\.codex\skills\windows-powershell-safety\SKILL.md
%USERPROFILE%\.codex\skills\windows-powershell-safety\agents\openai.yaml
```

After installation, restart Codex or open a new thread so the skill list refreshes.
