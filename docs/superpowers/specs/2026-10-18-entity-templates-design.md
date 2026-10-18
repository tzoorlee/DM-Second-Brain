# Entity templates and TEMPLATES.md — design

Date: 2026-10-18 · Target: template 0.5.0 · Vault format: stays 1 · Status: approved in chat, awaiting spec review

## Goal

Every kind of wiki page has a template, the templates agree with each other and with
`CLAUDE.md`, pages link to each other in one checkable way, and one document,
`TEMPLATES.md`, states the rules the DM and the agent both follow when they create a page.

What the DM asked for:

- check that the templates are complete, consistent and linkable;
- add the missing ones;
- write rules for each template, used by the DM and by the AI to create new pages.

Decisions taken while designing:

- the rules live in a dedicated `TEMPLATES.md` at the template root;
- new types `deity`, `creature`, `lore` and `adventure`;
- quests declare their dependencies (`requires`) and their `adventure`;
- the file name (slug) is the link target: `[[slug|Shown name]]` in text, `[[slug]]` in frontmatter;
- scope: the template and the Silverbrook example now; the companion app later, in its own spec.

The spec and plan live in `docs/`, which is not part of a campaign: the app copies only
`CLAUDE.md`, `dm-second-brain.json`, `.gitignore`, `wiki/`, `raw/` and `output/`.

Out of scope: app changes (type labels, showing the new link fields), a shipped lint
script, a vault format change, translated templates.

## Audit (what is wrong today)

- `CLAUDE.md` §3 lists 10 types; only 6 have templates (`npc`, `event`, `faction`,
  `city`, `city-history`, `quest`). `pc`, `region`, `dungeon`, `item`, `secret` have none.
  The example also uses `location` and `world`, which are in no list; the app already
  knows `region-history` and `faction-history`.
- §4 says cities, regions and factions keep state and history in separate files, but
  the faction template has an inline History section and no `faction-history` exists.
- §3 gives one mixed `status` list; each template uses its own; events have none.
- The quest template's `## Secrets / Twists (DM only)` is not the exact marker §3 requires.
- `city-history` has no `canon_status` or `pg_known`.
- Links mix titles (`[[Silverbrook]]`), slugs (`[[silverbrook]]`), aliased slugs
  (`[[tomas|Tomas]]`) and paths (`[[world/calendar]]`). Obsidian resolves only file names,
  so title links break there.
- Only `npc` has link fields (`faction`, `location`). Quests, items and events have none.
- The example writes `first_seen: design`; the templates only allow `session-NN`.
  Events use `session:` instead.

## 1. Common rules

**Slugs and links**

- The slug is the file name: lowercase ASCII letters, digits and dashes (`elder-maren.md`).
  It is set at creation and never changes; renaming an entity changes only its `# Title`.
- Slugs are unique across the whole wiki, not only within a folder.
- In text, links are `[[slug|Shown name]]`. In frontmatter they are `"[[slug]]"`; several
  values are a YAML list: `requires: ["[[a]]", "[[b]]"]`. No title links, no path links.
- Every entity named on a page is linked at its first mention in each section.

**Base frontmatter** for every entity page:

| Field | Values |
|---|---|
| `type` | a type from the catalog (section 2) |
| `tags` | free list, lowercase with dashes |
| `canon_status` | `prepared` or `established` |
| `pg_known` | `true` or `false` |
| `first_seen`, `last_updated` | `session-NN`; `design` for prepared material; `setup` only for onboarding pages |
| `status` | only for types with a lifecycle, values from the type's card |

Link fields specific to a type are listed in its card.

**History pages** (`city-history`, `region-history`, `faction-history`)

- Same folder as their entity, slug with the `-history` suffix (`silverbrook-history.md`).
- Frontmatter: `type`, `tags`, `subject: "[[slug]]"`, `canon_status`, `pg_known`,
  `last_updated`. `subject` replaces `city_page`, which stays accepted as a legacy name.
  The main page has `history_page: "[[slug-history]]"`.
- Append-only. Each entry is `## [in-world-date | session-NN]`.
- Cities, regions and factions all split state and history. The faction template loses
  its History section.

**Secrets**

- Every template that has DM-only content ends with exactly `## Secrets (DM only)`.

**Compatibility**

- All of this stays vault format 1: the new fields are optional, the types are new, no
  marker changes. Existing campaigns keep opening in the app.
- Lint reports old-style content (title links, `city_page`, `Secrets / Twists`) as
  minor, with a proposed fix it doesn't apply without the DM's consent.

## 2. Type catalog

★ = new.

| Type | Folder | `status` | Link fields |
|---|---|---|---|
| `world` | `world/` | — | — |
| `region` | `locations/regions/` | active, destroyed, abandoned | `region` ★ (parent region), `history_page` |
| `region-history` | `locations/regions/` | — | `subject` |
| `city` | `locations/cities/` | active, destroyed, abandoned | `region` ★, `history_page` |
| `city-history` | `locations/cities/` | — | `subject` |
| `location` | `locations/places/` ★ | active, destroyed, abandoned | `location` (city or region), `owner` ★ |
| `dungeon` | `locations/dungeons/` | active, cleared ★, destroyed, abandoned | `location`, `faction` (who controls it) |
| `pc` | `characters/pcs/` | alive, dead, missing, retired | `faction`, `location`, `player` ★ (player's name, plain text) |
| `npc` | `characters/npcs/` | alive, dead, missing, unknown | `faction`, `location` |
| `faction` | `factions/` | active, disbanded, destroyed | `location` (headquarters), `deity` ★, `history_page` ★ |
| `faction-history` ★ | `factions/` | — | `subject` |
| `deity` ★ | `deities/` ★ | active, dead, forgotten | — (`domains`, `symbol` as plain fields) |
| `creature` ★ | `creatures/` ★ | active, extinct | `location` (habitat, may be a list), `srd` (SRD monster it derives from, plain text) |
| `lore` ★ | `lore/` ★ | — | — |
| `adventure` ★ | `adventures/` ★ | pending, active, completed, abandoned | — (quests listed in order in the body) |
| `quest` | `quests/active/`, `quests/completed/` | pending ★, active, completed, failed, abandoned | `adventure` ★, `requires` ★ (list), `giver` ★, `location` ★ |
| `item` | `items/` | active, lost, destroyed | `owner` ★, `location` ★ |
| `event` | `events/` | — | `date_in_world` (required), `session`, `location` ★, `adventure` ★ |
| `secret` | `secrets/` | hidden, partly-revealed, revealed | `about` ★ (list of entities it concerns) |

Rules that follow:

- `pending` quests haven't been taken up. `completed`, `failed` and `abandoned` quests
  move to `quests/completed/`. A quest can't be `active` while a quest in its `requires`
  isn't `completed`; lint reports it.
- Secrets always have `pg_known: false`. A revealed secret becomes `status: revealed` and
  stays as an archive; its information moves to the public pages (§6 of `CLAUDE.md`).
- Places inside a city or region (taverns, temples, shops) are `location` pages in
  `locations/places/`.
- `world` pages (overview, calendar, cosmology) use the base frontmatter without `status`.

Link targets (used by lint and by the summary table in `TEMPLATES.md`):

| Field | Points to | Back-link to add |
|---|---|---|
| `location` | `region`, `city`, `location`, `dungeon` | the target's list for that kind of page, named in the target's card (e.g. a city's "Resident NPCs" for an `npc`, its "Notable Places" for a `location`); `creature` habitats need none |
| `region` | `region` | the region's "Cities and Places" list |
| `faction` | `faction` | the faction's "Key Members" (for `npc`, `pc`) or "Holdings" (for `dungeon`) |
| `owner` | `npc`, `pc`, `faction` | the owner's "Possessions" list |
| `deity` | `deity` | the deity's "Followers" list (factions only) |
| `giver` | `npc`, `faction` | the giver's "Quests Given" list |
| `adventure` | `adventure` | the adventure's "Quests" (for `quest`) or "Key Events" (for `event`) list |
| `requires` | `quest` | — |
| `about` | any entity | — (the secret is DM-only; nothing public links back) |
| `subject` / `history_page` | the paired page | each other |

## 3. TEMPLATES.md and CLAUDE.md

`TEMPLATES.md` at the template root, in English:

1. Purpose: the single source of rules for creating wiki pages, for the DM and the agent.
2. Common rules (section 1 above).
3. Creating a page, step by step: find the card; check the slug is free across the wiki;
   copy `_template-<type>.md` into the type's folder; fill the frontmatter (catalog
   values, `[[slug]]` links, `prepared` or `established` by source); write the sections in
   order and link every named entity; add back-links on the linked pages; update
   `index.md` (the agent also appends to `log.md`).
4. One card per type: purpose, folder, fields (required and optional, allowed values),
   body sections in order, back-links to maintain, what to do on promotion, archiving or
   reveal.
5. Adding a new type: check first that an existing type plus `tags` isn't enough; pick a
   lowercase English name, a folder, fields and `status` values; write the template and
   the card; add the catalog row. It is the DM's decision: the agent may propose a type,
   not create one.
6. Summary table of link fields (the link-targets table above).

Each template file `_template-<type>.md` lives in its type's folder and holds only the
skeleton: frontmatter with every field (inline comments listing allowed values) and the
body sections in card order.

Changes to `CLAUDE.md`:

- §1: folder list adds `adventures/`, `deities/`, `creatures/`, `lore/`,
  `locations/places/`.
- §2: design ingest and ingest create or update pages following `TEMPLATES.md`, back-links
  included. Lint also checks: link fields pointing to the wrong type, missing back-links,
  quest dependencies (cycles, active quests blocked by unfinished ones, adventures that
  don't list a quest pointing to them), and old-style content (minor).
- §3: replaces its own type list with a pointer to `TEMPLATES.md`, keeping a short
  summary (slugs, link form, fixed English markers).
- §4: unchanged in substance; the example code block uses slug links.
- §9: new rule: "Every new page follows its `TEMPLATES.md` card; a new entity type needs
  the DM's approval."
- Template version 0.5.0.

## 4. Example, release and verification

Silverbrook example migrated:

- every link to `[[slug|Name]]` in text and `[[slug]]` in frontmatter;
- `silverbrook-history`: `subject` instead of `city_page`, plus `canon_status`, `pg_known`;
- quest `the-eastern-hills`: `## Secrets (DM only)`, `giver: "[[elder-maren]]"`,
  `location: "[[sunken-crypt]]"`, `adventure` set;
- one new `adventure` page grouping that quest, with no new lore: it only orders what the
  sessions already say;
- `the-mended-kettle` moves to `locations/places/` (slug unchanged);
- the secret: `status: hidden`, `about` set;
- the item gets `owner` or `location` only where the sessions say so;
- `index.md` gains the new sections (Adventures, Places, Deities, Creatures, Lore), empty
  where there's nothing;
- back-links added where the new link fields require them;
- no new world content (rule 2 of `CLAUDE.md`): `deity`, `creature` and `lore` appear only
  as templates.

Empty template:

- new folders with `.gitkeep`;
- one template per type: 19 `_template-*.md` files (13 new, 6 revised);
- `index.md` with the new sections.

Release: template 0.5.0 (`CLAUDE.md` "Template version", README folder tree and a pointer
to `TEMPLATES.md`, CHANGELOG). `dm-second-brain.json` stays `{ "vaultFormat": 1 }`.

Verification (the template has no test suite):

- a throwaway check script, kept out of the repository, run on the example and on the
  templates: every link resolves to an existing file; every `type` and `status` is in the
  catalog; link fields point to allowed types; required back-links exist; no cycle in
  `requires`; every template with DM-only content ends with `## Secrets (DM only)`;
- the example opened in the companion app 0.5.1 (from a copy in a scratch campaigns
  folder): all pages listed, links open, Player Safe still hides secrets;
- a branch and a pull request on the template repository.
