# gh-flox

Go CLI tool and AWS Lambda function that measures flox ecosystem adoption by querying GitHub for repositories using flox.

## Build & Run

```bash
flox activate              # Enter dev environment (provides go, awscli2, gnumake, golint)
make local                 # Build native binary (runs go fmt first)
make lambda                # Build ARM64 binary for AWS Lambda, zip, and deploy
make ready                 # Build linux binary and deploy to hubot server via scp
make clean                 # Remove build artifacts
```

## Project Structure

- `main.go` — Single-file CLI app (Cobra framework). All commands, GitHub API logic, caching, Lambda handler.
- `post-processing/main.go` — Separate tool that loads JSON export into SQLite for historical tracking.
- `additional_repos.json` — Hand-maintained repo list embedded into the binary at compile time.
- `.flox/env/manifest.toml` — Flox dev environment definition.

## Key Commands

```bash
./gh-flox repos [-v] [-f]      # List repos with .flox/env/manifest.toml
./gh-flox readmes [-v] [-f]    # List repos with "flox install" in README
./gh-flox stars                 # Star count for flox/flox
./gh-flox floxindex [-f]       # Sum of stars across all discovered repos
./gh-flox export [-f]          # JSON export of all data
./gh-flox version              # Git SHA and dirty state
./gh-flox clearcache           # Clear in-memory/file cache
```

Flags: `-v` verbose (include star counts), `-f` full (include flox org repos), `--no-cache` disable caching.

## Environment Variables

- `GITHUB_TOKEN` (required) — GitHub API token
- `SLACK_MODE` — Enable Slack-formatted output
- `DEBUG` — Enable debug logging
- `S3_BUCKET_NAME`, `S3_OBJECT_KEY`, `AWS_REGION` — Lambda S3 upload config
- `LAMBDA_TASK_ROOT` — Auto-set by AWS Lambda runtime

## Architecture Notes

- Caching: In-memory (patrickmn/go-cache, 4h TTL) persisted to `/tmp/cache.gob` via gob encoding.
- Lambda: When `LAMBDA_TASK_ROOT` is set, runs `export` command, captures stdout via pipe, uploads to S3.
- Version info embedded via LDFLAGS (`-X main.GitSHA=... -X main.GitDirty=...`).
- Go 1.21.8, module path `github.com/stahnma/gh-flox`.
