# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
versioning follows [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned

- Field-test on a real campaign and refine CLAUDE.md
- Optional: package as an installable agent skill

## [0.5.1] - 2026-10-18

### Fixed

- `TEMPLATES.md`: the base-frontmatter table says that events (which use `session`) and history pages (only `last_updated`) have no `first_seen`.
- `CLAUDE.md` §4: the NPC example has the template's `## Possessions` and `## Quests Given` sections, and shows link fields in its frontmatter.
- `CLAUDE.md` §6: revealing a secret sets `pg_known: true` on the public pages; the `secrets/` page keeps `pg_known: false` and becomes `partly-revealed` or `revealed`.

## [0.5.0] - 2026-10-18

### Added

- **`TEMPLATES.md`**: the rules for every page type, followed by both the DM and the agent: slugs, links, base frontmatter, one card per type (folder, fields, `status` values, sections, back-links), how to add a type.
- **Templates for every type**: 19 `_template-*.md` files. New types `adventure`, `deity`, `creature`, `lore` and `faction-history`, plus templates for `world`, `region`, `region-history`, `location`, `dungeon`, `pc`, `item` and `secret`.
- **Link fields** that connect pages: `region`, `owner`, `deity`, `giver`, `adventure`, `about`, `player`, and quest dependencies with `requires`.
- New folders `adventures/`, `locations/places/`, `deities/`, `creatures/`, `lore/`.
- Lint checks link fields, back-links, quest dependencies and public links to hidden pages.

### Changed

- **Links use the file name**: `[[slug|Shown name]]` in the text, `"[[slug]]"` in the frontmatter. Title and path links are reported by lint as old style.
- Factions keep their chronology in a separate `faction-history` page, like cities and regions.
- Each type has its own `status` values; quests gain `pending`, dungeons `cleared`, secrets use `hidden`, `partly-revealed`, `revealed`.
- History pages use `subject` (`city_page` is still read) and gain `canon_status` and `pg_known`.
- The quest template's secrets heading is now exactly `## Secrets (DM only)`.
- The Silverbrook example follows the new rules and gains an adventure page.

The vault format stays **1**: every change is additive, and existing campaigns open as before.

## [0.4.0] - 2026-10-18

### Removed

- **The Companion Dashboard moved to its own repository**, [DM-Second-Brain-app](https://github.com/tzoorlee/DM-Second-Brain-app), with its history. `dashboard/` is no longer part of the template. Clone the app next to your campaign and set `VAULT_PATH` to open it.

### Added

- `dm-second-brain.json` at the vault root, in the template and in the example. It declares the vault format (`vaultFormat: 1`) that apps such as the companion app rely on. The README documents what format 1 covers.

### Changed

- README, `CLAUDE.md` and `examples/README.md` now point to the companion app's repository.

## [0.3.2] - 2026-10-18

### Security

- **Player Safe is now enforced on the server.** `/api/entity` is player-safe by default; the full page requires `mode=dm`. In player mode it answers 404 for `secrets/` pages and entities with `pg_known: false`, the same as for missing pages. It also removes the DM-only section before the text leaves the server. Before this, anyone who knew a page path could read its secrets.
- `/api/entity` only serves pages the search index lists. `index.md`, `log.md`, `_template-*.md` and anything outside `wiki/` can no longer be read through it. It no longer returns the page's absolute file path.

### Changed

- DM-only sections are also recognized by any level-2 heading that names the DM or GM, such as `## Segreti (solo DM)`. This is a safety net for wikis written in other languages. `## Secrets (DM only)` stays the convention.
- Switching between DM View and Player Safe reloads the open page in the new mode, and closes it if players may not see it.

## [0.3.1] - 2026-10-18

### Changed

- Dashboard UI pass. Wiki pages render proper markdown: paragraphs, numbered lists, bold, italic and inline code. The repeated title is gone.
- The layout adapts to smaller screens. Columns narrow first, and below 1024px a tab bar shows one panel at a time.
- Player Safe is now a two-way DM View / Player Safe switch. A visible banner shows while secrets are hidden.
- The entity sheet shows location, faction, last update and tags. In DM View it also shows an "Unknown to PCs" badge for `pg_known: false`.
- Following a wikilink adds a Back button. A link to a missing or hidden page appears muted instead of opening an alert.
- The page list shows a result count and a "Clear filters" link, and it works from the keyboard. Statuses and all entity types have distinct colors.
- Session notes:
  - The `[[` autocomplete opens at the cursor.
  - Unsaved notes are kept as a draft in the browser.
  - Ctrl/Cmd+S saves.
  - A word count is shown.
- `CLAUDE.md`: frontmatter keys and values, and the `## Secrets (DM only)` heading, now always stay in English, whatever the output language. Before this, the agent could translate the heading, and Player Safe then showed those secrets to the players.
- Fonts are self-hosted through `next/font` (no requests to Google from the browser). An unused stylesheet was removed.

### Fixed

- Dropdown menus in the wiki column were hard to read: their options now use a dark, opaque background.

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
