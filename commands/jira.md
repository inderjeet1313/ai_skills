# /jira - Jira Ticket Implementation Pipeline

## Usage
```
/project:jira MIT-3881
/project:jira MIT-3881 --analyze-only
/project:jira MIT-3881 --plan-only
```

## What This Does

Triggers the full Jira implementation pipeline via the `jira-agent` orchestrator:

```
Fetch ticket → ticket-analyzer → codebase-scanner → implementation-agent → quality gates
```

## Flags

| Flag | Behaviour |
|------|-----------|
| _(none)_ | Full pipeline: fetch → analyze → scan → implement → quality gates |
| `--analyze-only` | Stop after ticket-analyzer. Output analysis, no code written. |
| `--plan-only` | Stop after codebase-scanner. Output plan, no code written. |

## Credentials Required

Must be set in `~/.zshrc`:
```bash
export JIRA_EMAIL="your.email@company.com"
export JIRA_API_TOKEN="your-atlassian-api-token"
export JIRA_BASE_URL="https://your-domain.atlassian.net"
```

## Agent Pipeline

Use the `jira-agent` for the ticket: $ARGUMENTS

The jira-agent will:
1. Fetch the ticket from Jira using environment credentials
2. Spawn `ticket-analyzer` — parses requirements into structured engineering analysis
3. Spawn `codebase-scanner` — maps exactly which files to touch and in what order
4. Spawn `implementation-agent` — writes all code following CLAUDE.md standards
5. Run quality gates:
   - `security-analyzer` — if ticket touches auth/crypto/payments/storage
   - `rtl-reviewer` — if ticket touches any UI
   - `test-writer` — always
   - `code-reviewer` — always

## Output

Each agent outputs a structured report. The jira-agent produces a final summary showing:
- All files created/modified
- Acceptance criteria status
- Quality gate results
- Issues to resolve before PR
