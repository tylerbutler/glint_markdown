# AGENTS.md — instructions for AI coding agents working on `glint_markdown`

## Build, test, and lint commands

- Use `just` recipes for normal development; each recipe already runs via `mise exec --` with the pinned toolchain.
- Install deps: `just deps`
- Build: `just build` (or `just build-strict` for warnings-as-errors)
- Type-check: `just check`
- Run tests (Erlang target): `just test`
- Run tests (JavaScript target via Node): `just test-js`
- Format: `just format`
- Format check: `just format-check`
- Lint: `just glint` (glinter; configured under `[tools.glinter]` in `gleam.toml`)
- Format-check + lint: `just lint`
- Full local CI pass: `just ci` (format-check + glint + check + test + test-js + build-strict)
- Create a changelog fragment: `just change`
- Preview unreleased changelog: `just changelog-preview`

## High-level architecture

- `src/glint_markdown.gleam` contains the pure renderer API. `src/glint_markdown/cli.gleam` provides CLI integration; `internal/plan` and `internal/io` separate planning from execution.
- Input: `glint/help.Tree` produced by `glint.document/1` upstream.
- Output: `String` Markdown, or `Dict(String, String)` of filename → contents in multi-file mode.
- Modes (`Mode`):
  - `Single` — one `# bin` document with TOC + every command as a `##` section.
  - `Multi(output_dir)` — one Markdown file per top-level subcommand, plus an index body for injecting into a parent README.
- Sentinel-comment injection (`inject/3`) mirrors oclif's `replaceTag`:
  - Both markers present → replace block in place.
  - Only start marker present → insert body + new stop marker.
  - No start marker → return readme unchanged.

## Key repository conventions

- Conventional Commits are enforced on PR titles (`.commitlintrc.json` + `.github/workflows/pr.yml`).
- Every PR that touches `src/` should add a changie fragment via `just change`. The Changelog job will comment on PRs lacking one.
- Anchor slug generation is delegated to the [`glugify`](https://hex.pm/packages/glugify) Hex package. Allocate unique explicit `command-` anchors per document and reuse them for navigation; do not hand-roll slug helpers.
- The `glint` dependency is pinned to a git ref on `tylerbutler/glint` until the upstream `glint.document/1` API is merged + released to Hex. When updating `gleam.toml`, prefer a commit SHA over a branch ref (per Gleam's own docs).
- Public API never re-exports types from `glint/internal/help`; downstream consumers should only need to import `glint`, `glint/help`, and `glint_markdown`.
- Gleeunit tests live under `test/`; `test/cli_integration.mjs` checks CLI subprocesses on both targets. Annotate all module functions, including tests, and add a focused test for any new public function.
