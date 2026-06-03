# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
versioning follows [Semantic Versioning](https://semver.org/).

## [Unreleased]
### Planned
- Field-test on a real campaign and refine CLAUDE.md
- Optional: package as an installable agent skill

## [0.1.0] - 2026-06-03
### Added
- Initial template based on Karpathy's LLM Wiki pattern, adapted for D&D 5e
- `CLAUDE.md` agent rulebook: four operations (onboarding, design ingest, ingest, query, lint)
- `canon_status` model (prepared vs. established) to separate design from played history
- Current-state vs. history split (inline for NPCs, separate files for cities/factions)
- Hybrid secret management (inline minor secrets + dedicated `secrets/` files) with promotion rule
- Empty templates for NPCs, cities, factions, quests, events
- Complete worked example: the Silverbrook campaign (2 sessions + prepared material)
- MIT license, credits to Karpathy and the second-brain project
- Optional-tools documentation (qmd, Obsidian Dataview)
