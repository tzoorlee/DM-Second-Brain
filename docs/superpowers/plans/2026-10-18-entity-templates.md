# Entity Templates and TEMPLATES.md Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete, consistent and linkable entity templates, the rules in `TEMPLATES.md`, `CLAUDE.md` aligned to them, and the Silverbrook example migrated — template release 0.5.0, vault format 1.

**Architecture:** Content-only change in the template repository. A throwaway checker script (kept in the plan's git-ignored workspace) encodes the spec's catalog and rules; each content task starts by running it and seeing the failures, and ends when it passes. Order: checker → templates → `TEMPLATES.md` → `CLAUDE.md`/README/CHANGELOG → example → app check and PR.

**Tech Stack:** Markdown + YAML frontmatter; checker in Node (ESM) using `gray-matter` from the sibling app's `node_modules` (`/home/fabioschenone/sviluppo/DM-Second-Brain-app`).

**Spec:** `docs/superpowers/specs/2026-10-18-entity-templates-design.md`

## Global Constraints

- `dm-second-brain.json` stays `{ "vaultFormat": 1 }`. Frontmatter keys/values and `## Secrets (DM only)` stay in English.
- Slug = file name, `^[a-z0-9]+(-[a-z0-9]+)*$`, unique across the vault. Links: `[[slug|Shown name]]` in text, `"[[slug]]"` (or a YAML list of them) in frontmatter. No title links, no path links.
- Base frontmatter on every entity page: `type`, `tags`, `canon_status` (`prepared`|`established`), `pg_known`, `first_seen`/`last_updated` (`session-NN`|`design`|`setup`); `status` only where the catalog gives values.
- History pages: `subject`, `canon_status`, `pg_known`, `last_updated`; entries `## [in-world-date | session-NN]`; `city_page` is a legacy alias of `subject`.
- Type catalog, folders, `status` values and link fields: exactly spec section 2.
- No new world content in the example (rule 2 of `CLAUDE.md`); `deity`, `creature`, `lore` appear only as templates.
- Template version 0.5.0. The checker never enters the repository.

## Body sections (decided here; the spec leaves them to the cards)

In order; `★S` = ends with `## Secrets (DM only)`. Back-link sections are the ones the spec's link-target table names.

| Type | Sections |
|---|---|
| `world` | Summary, Details, Related, ★S |
| `region` | Current State, Cities and Places, Factions Present, Links, ★S |
| `city` | Current State, Notable Places, Resident NPCs, Factions Present, Links, ★S |
| `location` | Current State, Notable Features, People, History / Chronology, ★S |
| `dungeon` | Current State, Notable Features, Inhabitants, History / Chronology, ★S |
| `pc` | Current State, Profile, Possessions, History / Chronology, Relationships, ★S |
| `npc` | Current State, Profile, Possessions, Quests Given, History / Chronology, Relationships, ★S |
| `faction` | Current State, Key Members, Holdings, Possessions, Quests Given, Relations with Other Factions, Links, ★S |
| `deity` | Current State, Profile, Followers, Holy Places, ★S |
| `creature` | Description, Habitat, Behavior, Statblock, Encounters, ★S |
| `lore` | Summary, Details, Related, ★S |
| `adventure` | Current State, Premise, Quests, Key Events, ★S |
| `quest` | Current State, Objective, Involves, Chronology, ★S |
| `item` | Current State, Description, History / Chronology, ★S |
| `event` | What Happened, Who Was Involved, Consequences, ★S |
| `secret` | The Secret, How It Should Unfold, Revealed So Far, Promotion Notes (whole page is DM-only; no ★S) |
| `*-history` | one `## [in-world-date \| session-NN]` per entry (no ★S: DM-only history goes in the entity's Secrets or in `secrets/`) |

Back-links required (field on page P of type → section on the target):

| Field | P's type | Target section |
|---|---|---|
| `location` | `npc` | city/location/region: "Resident NPCs" (city), "People" (location), "Inhabitants" (dungeon); not required on a region |
| `location` | `location`, `dungeon` | city: "Notable Places"; region: "Cities and Places" |
| `region` | `city`, `region` | region: "Cities and Places" |
| `faction` | `npc`, `pc` | faction: "Key Members" |
| `faction` | `dungeon` | faction: "Holdings" |
| `owner` | `item`, `location` | npc/pc/faction: "Possessions" |
| `deity` | `faction` | deity: "Followers" |
| `giver` | `quest` | npc/faction: "Quests Given" |
| `adventure` | `quest` | adventure: "Quests"; `event` → adventure: "Key Events" |
| `subject` ↔ `history_page` | history page ↔ entity | each other's frontmatter |

## Review Focus

- A vault created by the app (it copies only `CLAUDE.md`, `dm-second-brain.json`, `.gitignore`, `wiki/`, `raw/`, `output/`) must be complete: `TEMPLATES.md` is at the template root, so `CLAUDE.md` must say where to find it when it's missing from a campaign (link to the GitHub file). Test in Task 4.
- Template files are skipped by the app, but their placeholder links (`[[npc-slug]]`) must not count as broken in the checker's template mode. Test in Task 1.
- The empty template's `index.md` today links `[[world/overview]]` and `[[world/timeline]]`, which don't exist: after migration it must have no broken links. Test in Task 2.
- Existing campaigns (format 1, old style) must still open in the app with no change: the example opened in app 0.5.1 before and after migration. Check in Task 6.
- `pg_known: false` pages linked from public pages (e.g. the faction from an NPC's Relationships) — Player Safe must still hide them; no new public back-link may sit outside a `## Secrets (DM only)` section when the target is `pg_known: false`. Test in Task 1 (checker rule) and Task 5.

---

### Task 1: Checker script (throwaway)

**Files:**
- Create: `<workspace>/check-vault.mjs` (workspace = output of `sdd-workspace` for this plan; git-ignored)

**Interfaces:**
- Produces: `node <workspace>/check-vault.mjs <dir> [--templates]` → prints one line per problem `<file>: <rule>: <detail>`, exits 1 if any, 0 and `OK (<n> pages)` otherwise.

- [ ] **Step 1: Write the checker.** Catalog (types, folders, status values, link fields with allowed target types) copied from spec section 2 as a JS object; body sections and back-links from the tables above. Parses frontmatter with `gray-matter` loaded via `createRequire('/home/fabioschenone/sviluppo/DM-Second-Brain-app/package.json')`. Skips `index.md`, `log.md`. Rules:
  - `type-unknown`, `wrong-folder`, `status-not-allowed`, `missing-base-field`, `bad-date` (`first_seen`/`last_updated`);
  - `slug-format`, `slug-duplicate`;
  - `link-style` (title or path link, in body or frontmatter), `link-broken` (target slug has no file), `link-field-type` (target's type not allowed for the field);
  - `sections-order` (headings vs table; extra headings allowed after the listed ones only for `world`), `secrets-not-last`, `secrets-heading-variant` (a `##` with "DM only" that isn't exactly the marker);
  - `backlink-missing` (per the back-link table), `requires-cycle`, `quest-blocked` (`status: active` with a `requires` target not `completed`), `adventure-not-listing`;
  - `public-link-to-hidden` (a link to a `pg_known: false` or `secret` page outside `## Secrets (DM only)` on a `pg_known: true` page);
  - `--templates` mode: only `_template-*.md` files; placeholder values (`session-NN`, `""`, `[]`, `[[...-slug]]`, `in-world-date`) are accepted; link and back-link rules are skipped; also checks there is exactly one template per catalog type, that each status comment lists exactly the catalog values, and `index.md` has no broken links.

- [ ] **Step 2: Run on the current template and example.**
  Run: `node <ws>/check-vault.mjs wiki --templates; node <ws>/check-vault.mjs examples/silverbrook-campaign/wiki`
  Expected: FAIL. Templates: missing types (13), `secrets-heading-variant` on quest, `sections-order` on faction (History), index broken links. Example: `link-style` on title links, `wrong-folder` on `the-mended-kettle`, `status-not-allowed` on the secret, `secrets-heading-variant` on the quest, `missing-base-field` on `silverbrook-history`. Save the output as `<ws>/baseline.txt`.

- [ ] **Step 3: Self-test the checker** on a scratch folder with three hand-made pages (a broken link, a `requires` cycle, a public link to a hidden page) — each rule fires once. No commit (nothing in the repo changed).

### Task 2: Templates, folders and the empty index

**Files:**
- Modify: the 6 existing `_template-*.md`
- Create: 13 templates — `wiki/world/_template-world.md`, `wiki/locations/regions/_template-region.md`, `wiki/locations/regions/_template-region-history.md`, `wiki/locations/places/_template-location.md`, `wiki/locations/dungeons/_template-dungeon.md`, `wiki/characters/pcs/_template-pc.md`, `wiki/factions/_template-faction-history.md`, `wiki/deities/_template-deity.md`, `wiki/creatures/_template-creature.md`, `wiki/lore/_template-lore.md`, `wiki/adventures/_template-adventure.md`, `wiki/items/_template-item.md`, `wiki/secrets/_template-secret.md`
- Create: `.gitkeep` in `wiki/locations/places/`, `wiki/deities/`, `wiki/creatures/`, `wiki/lore/`, `wiki/adventures/`; drop `.gitkeep` where a template now lives
- Modify: `wiki/index.md`

- [ ] **Step 1: Run** `node <ws>/check-vault.mjs wiki --templates` — expected: the Task 1 template failures.
- [ ] **Step 2: Write the templates.** Frontmatter: base fields + the type's link fields from spec section 2, each with an inline comment of allowed values or target types (`location: "" # [[slug]] of a region, city, location or dungeon`); placeholders `session-NN`, `""`, `[]`. Body: `# <Type> Name`, then the sections of the table above, each with one italic hint line saying what goes there (e.g. `- [[npc-slug|NPC Name]] — role`). History templates start with `> Append-only. Each entry dated in the in-world calendar.` Faction loses its History section and gains `history_page`. Quest's heading becomes `## Secrets (DM only)`. `city-history` uses `subject`.
- [ ] **Step 3: Rewrite `wiki/index.md`** with one section per type in catalog order (World, Adventures, Player Characters, NPCs, Regions, Cities, Places, Dungeons, Factions, Deities, Creatures, Active Quests, Completed Quests, Items, Events, Lore, Secrets (DM only)), each `_(none yet)_`, except World: `- [[calendar|In-world calendar]] — the campaign calendar`; version line `Built with DM-Second-Brain v0.5.0`.
- [ ] **Step 4: Run** the checker in template mode — expected: `OK`.
- [ ] **Step 5: Commit** — `feat: complete and consistent entity templates`

### Task 3: TEMPLATES.md

**Files:**
- Create: `TEMPLATES.md`

- [ ] **Step 1: Write `TEMPLATES.md`** with the six parts of spec section 3, in English, matching the README's voice. The common rules copy spec section 1; each card has: Purpose (one line), Folder, Template file, Fields (table: field, required?, values/targets), Body sections (the table's order), Back-links (from the table above), Lifecycle (promotion, archiving, reveal — only where it applies). The adding-a-type part ends with the checklist. Part 6 is the spec's link-target table.
- [ ] **Step 2: Verify coverage:** `for t in <19 types>; do grep -q "^### \`$t\`" TEMPLATES.md || echo missing $t; done` — expected: no output. And each card's folder equals the catalog's (`grep -A3`).
- [ ] **Step 3: Commit** — `docs: TEMPLATES.md with the rules for every page type`

### Task 4: CLAUDE.md, README, CHANGELOG

**Files:**
- Modify: `CLAUDE.md`, `README.md`, `CHANGELOG.md`

- [ ] **Step 1: Edit `CLAUDE.md`** per spec section 3 (§1 folders, §2 ingest/design ingest/lint, §3 pointer + summary, §4 example block with slug links, §9 new rule 17, version 0.5.0). §3 says: "`TEMPLATES.md` is at the template's root; in a campaign created by the companion app it may be missing — read it at https://github.com/tzoorlee/DM-Second-Brain/blob/v0.5.0/TEMPLATES.md".
- [ ] **Step 2: README:** folder tree with the new folders; a short "Page templates" section pointing to `TEMPLATES.md`. **CHANGELOG:** `## [0.5.0] - <date>` with Added (types, `TEMPLATES.md`, link fields, quest dependencies), Changed (link convention, faction history split, status vocabularies, quest secrets heading, `subject`), and a note that vault format stays 1.
- [ ] **Step 3: Verify:** `grep -n "0.4.0" CLAUDE.md README.md wiki/index.md` → only historical mentions (CHANGELOG-style), none as "current version"; `grep -c "TEMPLATES.md" CLAUDE.md` ≥ 3; new folders present in both `CLAUDE.md` §1 and the README tree.
- [ ] **Step 4: Commit** — `docs: CLAUDE.md and README follow TEMPLATES.md, version 0.5.0`

### Task 5: Migrate the Silverbrook example

**Files:**
- Modify: every page under `examples/silverbrook-campaign/wiki/`
- Move: `locations/cities/the-mended-kettle.md` → `locations/places/the-mended-kettle.md` (`git mv`)
- Create: `examples/silverbrook-campaign/wiki/adventures/eastern-hills-mystery.md`

- [ ] **Step 1: Run** `node <ws>/check-vault.mjs examples/silverbrook-campaign/wiki` — expected: the Task 1 example failures.
- [ ] **Step 2: Migrate** per spec section 4: links to slug form; `silverbrook-history` → `subject` + `canon_status: established` + `pg_known: true`; quest gets `giver: "[[elder-maren]]"`, `location: "[[sunken-crypt]]"`, `adventure: "[[eastern-hills-mystery]]"`, `## Secrets (DM only)`; secret → `status: hidden`, `about: ["[[elder-maren]]", "[[sunken-crypt]]", "[[tarnished-silver-locket]]"]`, sections renamed to the table; item: no `owner` (the sessions say "the party", and there are no PC pages), `location` empty; Mended Kettle moved, gains `## People` with Hilda; every page's sections reordered/renamed to the table without changing content (empty new sections get `- _(none yet)_`); a public link to a hidden page (`public-link-to-hidden`) moves, with its sentence, into that page's `## Secrets (DM only)`; `index.md` gets the new sections and slug links.
- [ ] **Step 3: Write the adventure page** `eastern-hills-mystery`: title "The Eastern Hills Mystery", `status: active`, `canon_status: established`, `first_seen: session-01`, `last_updated: session-02`; Premise and Current State use only sentences already in the quest page; Quests lists `[[the-eastern-hills|The Eastern Hills]]`; Key Events lists `[[clearing-the-sunken-crypt|...]]` (and that event gets `adventure`). Log line appended to `log.md`: `## [setup] migration — pages aligned to TEMPLATES.md (template 0.5.0)`.
- [ ] **Step 4: Run** the checker on the example — expected: `OK`. Run `diff <(git show HEAD:examples/.../X) X` on two pages to confirm wording is unchanged apart from links and headings.
- [ ] **Step 5: Commit** — `docs: migrate the Silverbrook example to the template rules`

### Task 6: App check and pull request

- [ ] **Step 1: Open the example in the app.** Copy `examples/silverbrook-campaign` into a scratch campaigns folder, back up the app's `.env.local`, point `CAMPAIGNS_DIR` at the scratch folder, open it in headless Chromium (as in the app's 0.5.x checks): DM view lists 14 pages (the 13 of today + the adventure), `[[slug|Name]]` links open their pages, Player Safe hides the secret, the faction and the DM sections. Restore `.env.local`.
- [ ] **Step 2: Run both checker modes once more** — expected: `OK`, `OK`.
- [ ] **Step 3: Push** `feat/entity-templates` and open a PR to `main` with summary, test plan and the note "vault format unchanged (1); companion app needs no change; app follow-up: labels for adventure/deity/creature/lore and the new link fields".
