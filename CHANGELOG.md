# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
versioning follows [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned

- Field-test on a real campaign and refine CLAUDE.md
- Optional: package as an installable agent skill

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
