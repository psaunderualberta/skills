# define-verification — worked example

A single end-to-end run on a hypothetical Go web service with a Postgres migrations dir and a React frontend.

## 1. Repo inspection findings

```
test command      : go test ./...  (fast, ~8s)
language/framework : Go (net/http) + React (Vite) frontend
UI surface         : yes — web/ (React), needs a browser to verify
migrations         : db/migrations/*.sql, applied via `make migrate-up`
CLI/API entrypoint : HTTP API (openapi.yaml present)
formatter/linter   : gofmt + golangci-lint present; prettier in web/
CI                 : .github/workflows/ci.yml runs go test + lint
project verify skill: none yet (→ recommend /verify to bootstrap)
```

## 2. Gate check

- **Bar 1 (≥2 change-types, different methods):** ✅ migration vs UI vs backend-logic all verify differently.
- **Bar 2 (surface beyond default test):** ✅ browser for UI, scratch-DB apply/rollback for migrations.

Clears both → **build the policy** (do not bail).

## 3. Drafted hook snippet (opt-in)

A formatter exists, so a PostToolUse auto-format hook fits. Drafted for the user, installed via `update-config` (never written directly):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "gofmt -w \"$CLAUDE_FILE_PATH\" 2>/dev/null; true" }
        ]
      }
    ]
  }
}
```

The `go test ./...` suite is cheap (~8s), so a **Stop hook** running it was also offered and accepted.

## 4. Resulting `## Verification` section in CLAUDE.md

```md
## Verification

| Change type        | What "verified" means                                  | How to run it                                  |
|--------------------|--------------------------------------------------------|------------------------------------------------|
| Backend logic      | `go test ./...` passes; affected endpoint driven once  | /verify (drives the HTTP endpoint)             |
| Schema migration   | applies cleanly on a scratch DB and rolls back         | make migrate-up then migrate-down on a tmp DB  |
| UI change          | component renders and the affected flow works          | /verify (drives the browser via Chrome ext)    |
| API contract       | openapi.yaml regenerated and matches handlers          | make openapi-check                             |
| docs/config-only   | no runtime verification required                       | —                                              |

Surfaces: UI changes require driving the browser; migrations require a scratch Postgres.
Hooks: PostToolUse auto-runs `gofmt` on edited files; a Stop hook runs `go test ./...`.
```

## Bail example (contrasting run)

A small pure-library repo: one `pytest` command, no UI, no migrations, no special surface.

- Bar 1: ❌ every change verifies the same way. Bar 2: ❌ nothing beyond `pytest`.

→ **Bail.** Write the single line and stop:

```md
Verification: run `/verify` before committing; no domain-specific policy needed.
```
