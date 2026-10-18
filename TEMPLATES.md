# TEMPLATES.md — How to write a wiki page

This is the single set of rules for creating pages in `wiki/`. The DM and the agent both
follow it: the DM when writing a page by hand, the agent on every ingest and design
ingest (see `CLAUDE.md`). Each page type has a template, `_template-<type>.md`, in its
folder. Copy it, never edit it.

1. [Common rules](#1-common-rules)
2. [Creating a page, step by step](#2-creating-a-page-step-by-step)
3. [Page types](#3-page-types)
4. [Adding a new type](#4-adding-a-new-type)
5. [Link fields at a glance](#5-link-fields-at-a-glance)

---

## 1. Common rules

### Slugs

- A page's **slug** is its file name without `.md`: lowercase ASCII letters, digits and
  dashes, such as `elder-maren` or `the-mended-kettle`.
- Slugs are **unique across the whole wiki**, not only within a folder.
- A slug is set when the page is created and **never changes**. If an entity is renamed,
  change its `# Title` and keep the file name, so no link breaks.

### Links

- Link to pages by slug, never by title or path.
- **In the text**: `[[slug|Shown name]]`, such as `[[elder-maren|Elder Maren]]`.
- **In the frontmatter**: `"[[slug]]"`, quoted. Several values make a YAML list:
  `requires: ["[[first-quest]]", "[[second-quest]]"]`.
- Link every entity a page names, at its first mention in each section.
- This works the same in Obsidian, in the companion app and in lint.

### Base frontmatter

Every entity page starts with:

| Field | Values |
|---|---|
| `type` | one of the [page types](#3-page-types) |
| `tags` | free list, lowercase with dashes: `[silverbrook, tavern]` |
| `status` | only for types that have one; the values are in the type's card |
| `canon_status` | `prepared` (designed, not played yet) or `established` (confirmed at the table) |
| `first_seen` | `session-NN`; `design` for prepared material; `setup` for onboarding pages |
| `last_updated` | same values as `first_seen` |
| `pg_known` | `true` or `false`: do the PCs know this entity exists? |

Then the type's own fields, listed in its card. Fields with no value stay as `""` or `[]`.

`_template-*.md`, `index.md` and `log.md` are not entity pages: they have no base
frontmatter, the templates' `[[...-slug]]` links are placeholders, and nothing needs to
link to them. Lint skips them.

**Fixed markers stay in English**, whatever language the wiki is written in: frontmatter
keys and values, and the heading `## Secrets (DM only)`. The companion app reads them.

### Secrets

- DM-only details go in a last section headed exactly `## Secrets (DM only)`. Player Safe
  mode hides it.
- A public page (`pg_known: true`) never links to a hidden page (`pg_known: false`, or any
  `secret`) outside that section, in the text or in the frontmatter. The link itself
  would tell the players the hidden page exists.
- So a link field of a public page never points to a hidden page. If a public NPC
  secretly belongs to a hidden faction, leave the NPC's `faction` empty and write the
  membership in the NPC's `## Secrets (DM only)`; the hidden faction still lists the NPC
  in its `## Key Members`.
- A history page has the same `pg_known` as its subject: the chronology of a hidden
  faction is hidden too.
- Plots large enough to drive an arc get their own page in `secrets/`.

### History pages

Cities, regions and factions keep their **current state** and their **chronology** in two
pages:

- `silverbrook.md` (`type: city`) has `history_page: "[[silverbrook-history]]"`;
- `silverbrook-history.md` (`type: city-history`) has `subject: "[[silverbrook]]"`.

The history page sits in the same folder, with the `-history` suffix. It is
append-only: each entry is a `## [in-world-date | session-NN]` heading. It has no
Secrets section; DM-only history goes in the entity's `## Secrets (DM only)` or in
`secrets/`. Older campaigns use `city_page` instead of `subject`; both are read.

NPCs, PCs, places, dungeons and items keep their chronology inside their own page, in
`## History / Chronology`.

### Prepared and established

- Material from `raw/worldbuilding/` creates `prepared` pages, with `first_seen: design`.
- Material from `raw/sessions/` creates `established` pages, with `first_seen: session-NN`.
- When the PCs interact with a `prepared` page, it becomes `established`.

---

## 2. Creating a page, step by step

1. Find the type's card below. If no type fits, see [Adding a new type](#4-adding-a-new-type).
2. Choose the slug and check it is free in the whole wiki.
3. Copy `_template-<type>.md` into the type's folder as `<slug>.md`.
4. Fill the frontmatter: the type's `status` values, `prepared` or `established`
   depending on the source, links as `"[[slug]]"`. Delete the inline `#` comments.
5. Write the sections in the template's order. Keep every heading, even when its
   content is `- (none yet)`. Link every entity you name.
6. For a `city`, `region` or `faction`, also create its history page from
   `_template-<type>-history.md` (slug `<slug>-history`, same `pg_known`), and fill
   `history_page` and `subject` so the two pages point at each other.
7. **Add the back-links**: each link field in the card says which list on the other
   page must link back here. For example, an NPC with `location: "[[silverbrook]]"` is
   added to Silverbrook's `## Resident NPCs`.
8. Add the page to `index.md` under its section, with a one-line summary.
9. The agent also appends a line to `log.md` (see `CLAUDE.md` §7).

---

## 3. Page types

| Type | Folder | Status values |
|---|---|---|
| [`world`](#world) | `world/` | — |
| [`adventure`](#adventure) | `adventures/` | pending, active, completed, abandoned |
| [`pc`](#pc) | `characters/pcs/` | alive, dead, missing, retired |
| [`npc`](#npc) | `characters/npcs/` | alive, dead, missing, unknown |
| [`region`](#region) | `locations/regions/` | active, destroyed, abandoned |
| [`region-history`](#region-history) | `locations/regions/` | — |
| [`city`](#city) | `locations/cities/` | active, destroyed, abandoned |
| [`city-history`](#city-history) | `locations/cities/` | — |
| [`location`](#location) | `locations/places/` | active, destroyed, abandoned |
| [`dungeon`](#dungeon) | `locations/dungeons/` | active, cleared, destroyed, abandoned |
| [`faction`](#faction) | `factions/` | active, disbanded, destroyed |
| [`faction-history`](#faction-history) | `factions/` | — |
| [`deity`](#deity) | `deities/` | active, dead, forgotten |
| [`creature`](#creature) | `creatures/` | active, extinct |
| [`quest`](#quest) | `quests/active/`, `quests/completed/` | pending, active, completed, failed, abandoned |
| [`item`](#item) | `items/` | active, lost, destroyed |
| [`event`](#event) | `events/` | — |
| [`lore`](#lore) | `lore/` | — |
| [`secret`](#secret) | `secrets/` | hidden, partly-revealed, revealed |

### `world`

**Purpose:** pages about the world as a whole: overview, global timeline, cosmology,
calendar.
**Folder:** `world/` · **Template:** `_template-world.md`

| Field | Required | Values |
|---|---|---|
| base fields | yes | no `status` |

**Sections:** Summary · Details · Related · Secrets (DM only). Add your own `##`
sections after Details when the topic needs them; `calendar.md` has its own structure.

### `adventure`

**Purpose:** a story arc that groups several quests, such as "The Eastern Hills Mystery".
**Folder:** `adventures/` · **Template:** `_template-adventure.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | pending, active, completed, abandoned |

**Sections:** Current State · Premise · Quests · Key Events · Secrets (DM only).
**Back-links:** `## Quests` lists every quest whose `adventure` points here, in play
order, noting branches; `## Key Events` lists every event whose `adventure` points here.
**Lifecycle:** `pending` until its first quest starts; `completed` or `abandoned` when
its quests end. Adventures are never moved or deleted.

### `pc`

**Purpose:** a player character.
**Folder:** `characters/pcs/` · **Template:** `_template-pc.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | alive, dead, missing, retired |
| `player` | no | the player's name, plain text |
| `faction` | no | `[[slug]]` of a `faction`; empty if the faction is hidden (see [Secrets](#secrets)) |
| `location` | no | `[[slug]]` of a `region`, `city`, `location` or `dungeon` |

**Sections:** Current State · Profile · Possessions · History / Chronology ·
Relationships · Secrets (DM only).
**Back-links:** a `faction` lists the PC in its `## Key Members`. `## Possessions` lists
every item and place whose `owner` points here. PCs move, so their `location` needs no back-link.

### `npc`

**Purpose:** a non-player character.
**Folder:** `characters/npcs/` · **Template:** `_template-npc.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | alive, dead, missing, unknown |
| `faction` | no | `[[slug]]` of a `faction`; empty if the faction is hidden and the NPC public (see [Secrets](#secrets)) |
| `location` | no | `[[slug]]` of a `region`, `city`, `location` or `dungeon` |

**Sections:** Current State · Profile · Possessions · Quests Given · History /
Chronology · Relationships · Secrets (DM only).
**Back-links:** the `faction`'s `## Key Members`; the `location`'s `## Resident NPCs`
(city), `## People` (location) or `## Inhabitants` (dungeon), none for a region.
`## Possessions` lists the items and places whose `owner` points here, `## Quests Given`
the quests whose `giver` points here.
**Lifecycle:** a dead NPC stays, with `status: dead` and its legacy in Current State.

### `region`

**Purpose:** a geographic area: a kingdom, a valley, a forest.
**Folder:** `locations/regions/` · **Template:** `_template-region.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | active, destroyed, abandoned |
| `region` | no | `[[slug]]` of the `region` that contains it |
| `history_page` | yes | `[[slug]]` of its `region-history` |

**Sections:** Current State · Cities and Places · Factions Present · Links · Secrets
(DM only).
**Back-links:** a parent `region` lists it in `## Cities and Places`, which also lists
every city, place and dungeon that points here.

### `region-history`

**Purpose:** the chronology of a region. See [History pages](#history-pages).
**Folder:** `locations/regions/` · **Template:** `_template-region-history.md`

| Field | Required | Values |
|---|---|---|
| `subject` | yes | `[[slug]]` of the `region` |
| `pg_known` | yes | the same as the region's |

### `city`

**Purpose:** a city, town or village.
**Folder:** `locations/cities/` · **Template:** `_template-city.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | active, destroyed, abandoned |
| `region` | no | `[[slug]]` of the `region` it lies in |
| `history_page` | yes | `[[slug]]` of its `city-history` |

**Sections:** Current State · Notable Places · Resident NPCs · Factions Present · Links ·
Secrets (DM only).
**Back-links:** the `region`'s `## Cities and Places`. `## Notable Places` lists the
places and dungeons whose `location` points here; `## Resident NPCs` the NPCs.

### `city-history`

**Purpose:** the chronology of a city. See [History pages](#history-pages).
**Folder:** `locations/cities/` · **Template:** `_template-city-history.md`

| Field | Required | Values |
|---|---|---|
| `subject` | yes | `[[slug]]` of the `city` (older campaigns: `city_page`) |
| `pg_known` | yes | the same as the city's |

### `location`

**Purpose:** a place inside a city or region: a tavern, a temple, a shop, a tower.
**Folder:** `locations/places/` · **Template:** `_template-location.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | active, destroyed, abandoned |
| `location` | yes | `[[slug]]` of the `city` or `region` it is in |
| `owner` | no | `[[slug]]` of an `npc`, `pc` or `faction` |

**Sections:** Current State · Notable Features · People · History / Chronology · Secrets
(DM only).
**Back-links:** the `location`'s `## Notable Places` (city) or `## Cities and Places`
(region); the `owner`'s `## Possessions`. `## People` lists the NPCs whose `location`
points here.

### `dungeon`

**Purpose:** an adventure site: a crypt, a cave system, a ruined keep.
**Folder:** `locations/dungeons/` · **Template:** `_template-dungeon.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | active, cleared, destroyed, abandoned |
| `location` | no | `[[slug]]` of the `region` or `city` it is in or near |
| `faction` | no | `[[slug]]` of the `faction` that controls it |

**Sections:** Current State · Notable Features · Inhabitants · History / Chronology ·
Secrets (DM only).
**Back-links:** the `location`'s `## Notable Places` (city) or `## Cities and Places`
(region); the `faction`'s `## Holdings`. `## Inhabitants` lists the NPCs whose
`location` points here.
**Lifecycle:** `cleared` once the PCs have dealt with it.

### `faction`

**Purpose:** a guild, cult, noble house, church or any organization.
**Folder:** `factions/` · **Template:** `_template-faction.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | active, disbanded, destroyed |
| `location` | no | `[[slug]]` of its headquarters |
| `deity` | no | `[[slug]]` of the `deity` it serves |
| `history_page` | yes | `[[slug]]` of its `faction-history` |

**Sections:** Current State · Key Members · Holdings · Possessions · Quests Given ·
Relations with Other Factions · Links · Secrets (DM only).
**Back-links:** the `deity`'s `## Followers`. `## Key Members` lists the NPCs and PCs
whose `faction` points here; `## Holdings` the dungeons; `## Possessions` the items and
places it owns; `## Quests Given` the quests it gave.

### `faction-history`

**Purpose:** the chronology of a faction. See [History pages](#history-pages).
**Folder:** `factions/` · **Template:** `_template-faction-history.md`

| Field | Required | Values |
|---|---|---|
| `subject` | yes | `[[slug]]` of the `faction` |
| `pg_known` | yes | the same as the faction's: a hidden faction has a hidden chronology |

### `deity`

**Purpose:** a god, demigod or worshipped power.
**Folder:** `deities/` · **Template:** `_template-deity.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | active, dead, forgotten |
| `domains` | no | list of plain words: `[light, life]` |
| `symbol` | no | short plain description |

**Sections:** Current State · Profile · Followers · Holy Places · Secrets (DM only).
**Back-links:** `## Followers` lists every faction whose `deity` points here.

### `creature`

**Purpose:** a monster or people invented for this campaign. SRD monsters need no page;
link to them in `raw/srd/`.
**Folder:** `creatures/` · **Template:** `_template-creature.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | active, extinct |
| `location` | no | list of `[[slug]]` of the places it lives |
| `srd` | no | the SRD monster it derives from, plain text |

**Sections:** Description · Habitat · Behavior · Statblock · Encounters · Secrets (DM only).
**Back-links:** none required.

### `quest`

**Purpose:** a plot thread the PCs can follow.
**Folder:** `quests/active/`, then `quests/completed/` · **Template:** `_template-quest.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | pending, active, completed, failed, abandoned |
| `adventure` | no | `[[slug]]` of the `adventure` it belongs to |
| `requires` | no | list of `[[slug]]` of quests that must be completed first |
| `giver` | no | `[[slug]]` of the `npc` or `faction` that gave it |
| `location` | no | `[[slug]]` of a `region`, `city`, `location` or `dungeon` |

**Sections:** Current State · Objective · Involves · Chronology · Secrets (DM only).
**Back-links:** the `adventure`'s `## Quests`; the `giver`'s `## Quests Given`.
**Lifecycle:**
- `pending` means not taken up yet.
- A quest can't be `active` while a quest in its `requires` isn't `completed`.
- `completed`, `failed` and `abandoned` quests move to `quests/completed/`, slug unchanged.
- No dependency cycles: a quest never requires, directly or indirectly, itself.
- If a required quest ends `failed` or `abandoned`, the DM decides: take it out of
  `requires` (the story found another way) or set the dependent quest to `abandoned`.
  Lint reports the case until then.

### `item`

**Purpose:** an item that matters to the story: an artifact, a key, a cursed blade.
**Folder:** `items/` · **Template:** `_template-item.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | active, lost, destroyed |
| `owner` | no | `[[slug]]` of the `npc`, `pc` or `faction` that holds it |
| `location` | no | `[[slug]]` of where it is, when nobody holds it |

**Sections:** Current State · Description · History / Chronology · Secrets (DM only).
**Back-links:** the `owner`'s `## Possessions`.

### `event`

**Purpose:** something that happened (or is planned to happen) at a date in the world.
**Folder:** `events/` · **Template:** `_template-event.md`

| Field | Required | Values |
|---|---|---|
| `date_in_world` | yes | `YYYY-Month-DD`, see `world/calendar.md` |
| `session` | yes | `session-NN`, or `design` for a planned event; replaces `first_seen` |
| `location` | no | `[[slug]]` of where it happened |
| `adventure` | no | `[[slug]]` of the `adventure` it belongs to |

No `status`. **Sections:** What Happened · Who Was Involved · Consequences · Secrets (DM
only).
**Back-links:** the `adventure`'s `## Key Events`.

### `lore`

**Purpose:** a topic that is neither a place nor a person: a language, an ancient war, a
magical order, a legend.
**Folder:** `lore/` · **Template:** `_template-lore.md`

No `status`, no link fields. **Sections:** Summary · Details · Related · Secrets (DM only).

### `secret`

**Purpose:** a major plot the PCs don't know yet. Small secrets go in the entity's
`## Secrets (DM only)` instead.
**Folder:** `secrets/` · **Template:** `_template-secret.md`

| Field | Required | Values |
|---|---|---|
| `status` | yes | hidden, partly-revealed, revealed |
| `pg_known` | yes | always `false` |
| `about` | no | list of `[[slug]]` of the entities it concerns |

**Sections:** The Secret · How It Should Unfold · Revealed So Far · Promotion Notes. The
whole page is DM-only, so it has no Secrets section.
**Back-links:** none. Public pages never link here (see [Secrets](#secrets)).
**Lifecycle:** when the PCs learn part of it, write it under Revealed So Far, move the
information to the public pages, and set `partly-revealed` or `revealed`. The page stays
as the record of the reveal.

---

## 4. Adding a new type

1. **Check first** that an existing type with `tags` isn't enough. A ship is an `item`
   or a `location`; a cult is a `deity` plus a `faction`; an encounter belongs in a
   `dungeon` or `quest`.
2. Pick a lowercase English name and a folder in `wiki/`.
3. Decide its fields: the base fields, its `status` values (if it has a lifecycle) and
   its link fields, each with the types it may point to and the back-link it needs.
4. Write `_template-<type>.md` in the folder and a card in this file, and add a row to
   the tables in [Page types](#3-page-types) and [Link fields at a glance](#5-link-fields-at-a-glance).
5. Add a section for it in `index.md`.

A new type is the **DM's decision**. The agent may propose one, explaining why the
existing types don't fit, but never creates one on its own.

---

## 5. Link fields at a glance

| Field | On | Points to | Back-link on the target |
|---|---|---|---|
| `location` | `npc` | `city`, `location`, `dungeon`, `region` | `## Resident NPCs` (city), `## People` (location), `## Inhabitants` (dungeon); none for a region |
| `location` | `location`, `dungeon` | `city`, `region` | `## Notable Places` (city), `## Cities and Places` (region) |
| `location` | `pc`, `quest`, `item`, `event`, `creature`, `faction` | `region`, `city`, `location`, `dungeon` | none |
| `region` | `city`, `region` | `region` | `## Cities and Places` |
| `faction` | `npc`, `pc` | `faction` | `## Key Members` |
| `faction` | `dungeon` | `faction` | `## Holdings` |
| `owner` | `item`, `location` | `npc`, `pc`, `faction` | `## Possessions` |
| `deity` | `faction` | `deity` | `## Followers` |
| `giver` | `quest` | `npc`, `faction` | `## Quests Given` |
| `adventure` | `quest` | `adventure` | `## Quests` |
| `adventure` | `event` | `adventure` | `## Key Events` |
| `requires` | `quest` | `quest` | none |
| `about` | `secret` | any page | none: nothing public links to a secret |
| `history_page` / `subject` | entity / its history page | each other | each other's frontmatter |
