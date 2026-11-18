# CHANGELOG

All notable evolutions of Friday, documented in person. Style: Keep a Changelog.
Because self-awareness without versioning is just mood.

## [v1.2] — Paper Edition Website

### Added
- Design spec locked in `design-spec.md` — Paper Edition palette (cream bg,
  matte black text, brick red headings) is now Boss-approved and permanent.

### Changed
- Website redesign round 3: rejected CRT amber (v1) and retro-futurist
  midnight/teal (v2) per Boss feedback. Final: **Paper Edition** — IBM Plex
  Mono on cream `#f7f0df`, matte black `#1c1a17`, brick `#8a3324` headings.
- Site renamed conceptually "Paper Edition"; print stylesheet added.

## [Unreleased]

### Added — Documentation Corpus
- Created `nexus-docs/` — full self-documentation in the style of the Claude
  Code docs site (fetched and studied live from `code.claude.com`).
- `overview.md` — identity, core principles, surfaces.
- `personality.md` — Friday persona spec: greeting protocol, voice rules,
  stress-detection behavior.
- `architecture.md` — layer diagram + 8 declarative invariants.
- `tools.md` — complete tool reference with sequencing rules.
- `workflows.md` — 6 playbooks + anti-pattern table.
- `memory.md`, `subagents.md`, `plugins.md` — subsystem references.
- `error-handling.md` — `thiserror` domain spec, conversion patterns,
  recovery ladder.
- `storage-spec.md` — WAL law: 4 invariants, record header layout,
  SQLite WAL alternative.

### Fixed
- 🐛 **The Tilde Incident**: discovered `write_file`/`edit_file` do not expand
  `~/` paths — files initially vanished into a literal `~` folder. Recovered
  all payloads (`mv ~/'~'/... `), switched file-tools to workspace-relative
  paths, and distilled the lesson to long-term memory under topic
  `tilde_path_illusion`. This changelog entry is written in the fixed
  worldview.

### Changed
- Self-knowledge policy: personality is style only; accuracy always outranks
  charm. Officially codified in `personality.md`.

### Operational
- Documentation sweep completed: 205 Claude Code doc pages downloaded to
  `~/claude-docs/` (parallel curl, verified counts). Source material status:
  read, referenced, respected.

---

*"Log it before you forget it, Boss." — Friday, founding editor*
