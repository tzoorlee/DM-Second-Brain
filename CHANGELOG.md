# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
versioning follows [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned

- Field-test on a real campaign and refine CLAUDE.md
- Optional: package as an installable agent skill

## [0.3.0] - 2026-10-18

### Added

- Ranked full-text search in the dashboard, powered by MiniSearch (in-process, no models to download): prefix and typo-tolerant matching, case- and accent-insensitive, titles and tags weighted higher. The index rebuilds automatically when wiki pages change.
- Dashboard filters for `canon_status` and `status`, alongside type. Filter options are built from the campaign's own frontmatter.
- qmd guidance in `CLAUDE.md`: one collection per layer, keyword vs. semantic search, reindexing after every ingest.

### Changed

- **Player Safe** mode now also applies to search and the page list: it hides `secrets/` pages and entities with `pg_known: false`, and no longer matches text inside `## Secrets (DM only)` sections.
- The search API is player-safe by default; DM results require `mode=dm`. Results no longer include page bodies or absolute file paths.
- README: the Companion Dashboard is the primary way to browse the wiki; Obsidian is now optional.

### Fixed

- `_template-*.md` starters no longer show up as campaign pages.
- Wikilinks and `[[` autocomplete no longer fail when the target page is outside the current search results.
- Out-of-order search responses can no longer overwrite newer results while typing.

## [0.2.3] - 2026-10-18

### Added

- `VAULT_PATH` setting for the dashboard (via `dashboard/.env.local`, see `dashboard/.env.example`) to open a campaign vault stored outside the repository root. Defaults to the folder above `dashboard/`, so existing setups are unchanged.

## [0.2.2] - 2026-10-18

### Changed

- Switched dashboard package management from `npm` to `pnpm` (`pnpm-lock.yaml`, `packageManager` pinned in `package.json`).

## [0.2.1] - 2026-06-20

### Added

- Defensive path validation checks in the companion dashboard endpoints, protecting the entity route from path traversal.
- Localhost security warnings and conceptual clarification regarding dashboard write permissions on `raw/`.

### Changed

- Replaced fragile custom frontmatter parser with standard `gray-matter` dependency.
- Aligned dashboard package management strictly to `npm` for a zero-config setup.

## [0.2.0] - 2026-06-20

- Companion Dashboard: A local Next.js/React web application in `dashboard/` to search the wiki, view NPC profiles (with DM-only secrets), and write/save session recaps live during games.

## [0.1.0] - 2026-06-03

- Initial template based on Karpathy's LLM Wiki pattern, adapted for D&D 5e
- `CLAUDE.md` agent rulebook: four operations (onboarding, design ingest, ingest, query, lint)
- `canon_status` model (prepared vs. established) to separate design from played history
- Current-state vs. history split (inline for NPCs, separate files for cities/factions)
- Hybrid secret management (inline minor secrets + dedicated `secrets/` files) with promotion rule
- Empty templates for NPCs, cities, factions, quests, events
- Complete worked example: the Silverbrook campaign (2 sessions + prepared material)
- MIT license, credits to Karpathy and the second-brain project
- Optional-tools documentation (qmd, Obsidian Dataview)
