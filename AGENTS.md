# AGENTS.md — Proyecto4

This is a **fresh repository** with no source code, build system, tests, or configuration yet. The only content is:

- `.codegraph/` — local codegraph metadata (gitignored, per-machine).

## What to expect

- There are no frameworks, languages, or toolchains set up.
- No `README`, `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, or equivalent.
- No CI, linter, formatter, or typechecker config.
- No tests, fixtures, or test runners.

## For agents working here

- **Do not search for build/test/lint commands** — they don't exist yet.
- **Do not assume any language or toolchain** until the user adds one.
- The `.codegraph/` directory is gitignored; treat it as local-only metadata.
- Treat this file as a placeholder that should be updated once the project has meaningful structure.

## Useful defaults to start with

When the user begins adding code, consider this checklist:

1. Choose a language/runtime and add the appropriate manifest (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, etc.).
2. Set up a type checker (TypeScript, basedpyright, mypy, etc.).
3. Set up a linter and formatter (ESLint+Prettier, ruff, clippy+rustfmt, golangci-lint, etc.).
4. Add at least a root `README.md`.
5. Add a meaningful `.gitignore` for the chosen stack.
