# Changelog

## v0.1.0 - 2026-10-14

### Added

- Initial release. Renders a `glint/help.Tree` (from `glint.document/1`) to Markdown reference documentation. Patterned after [oclif's readme generator][oclif] — supports single-file output (one README with TOC + every command) and multi-file output (one file per top-level subcommand plus an index). Sentinel-comment injection via `glint_markdown.inject/3` mirrors oclif's `replaceTag`. Anchor slugs delegated to the [`glugify`](https://hex.pm/packages/glugify) Hex package. [oclif]: https://github.com/oclif/oclif/blob/main/src/readme-generator.ts
- New `glint_markdown/cli` module exposing a ready-made `gen-docs` subcommand that callers mount into their own glint CLI with one line: `app |> glint_markdown_cli.mount(at: ["gen-docs"])`. Supports `--mode single|multi`, `--out`, `--readme` (sentinel injection in place), `--repo-prefix`, `--include-hidden`, `--no-toc`, `--check` (CI-friendly diff mode that exits 1 on drift), and `--quiet`. `mount` is restricted to `Glint(Nil)` host apps; non-`Nil` hosts use the lower-level `glint_markdown_cli.command/1` with `glint.map_command`. Adds `simplifile` as a direct dependency for filesystem IO.
- Add README root sentinel injection via `to_root_body` and `gen-docs --readme`.

### Changed

- Use Trellis to manage changelogs and Git-only releases. Release tags and GitHub Releases replace Hex publishing.

### Fixed

- `to_topics_index_body` now emits its section heading as an H2 (`## Subcommands`) instead of an H1, matching its docstring and preserving heading hierarchy when injected into a README.
- `to_files` no longer duplicates each topic's heading and description. The per-file `#` title now serves as the topic section, and only descendant subcommands are rendered as `##` sections.
- Use explicit unique command anchors, encode and relativize topic links, reject unsafe topic paths and conflicting destinations, and keep CLI status output on stderr.
- Preflight destination aliases and replace files atomically, preserving permissions and symbolic links. Refuse changed hardlinked files and compare exact bytes, including UTF-8 byte-order marks, on both targets.
