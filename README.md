# 🎲 Dungeon Master Second Brain

![version](https://img.shields.io/badge/version-0.6.0-blue)
![license](https://img.shields.io/badge/license-MIT-green)

**An LLM-maintained, self-consistent wiki for your Dungeons & Dragons 5e campaign.**

Drop your session notes into a folder. An AI coding agent reads them, builds a
structured, interlinked wiki — NPCs, cities, factions, quests, secrets — and keeps
it consistent over time. You stay the Dungeon Master; the agent is your tireless
loremaster and continuity editor.

Built on [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f),
adapted for the specific needs of a living, evolving tabletop campaign.

---

## Why this is different from a normal wiki

A normal campaign wiki dies because **maintenance is tedious**: updating
cross-references, keeping NPC states current, noticing when session 14 contradicts
session 3. LLMs don't get bored. They can touch 15 pages in one pass and never
forget to update a backlink.

But D&D needs things a generic knowledge base doesn't, and this template adds them:

- **Current state vs. history** — an NPC isn't just "dead", they died _in session 14_
  and that has consequences. The wiki tracks both what's true _now_ and _how you got there_.
- **Prepared vs. established** (`canon_status`) — design a whole world before you play.
  Entities you've drafted are `prepared` (still malleable); the moment the players
  touch them in-game, they become `established` (canon). Players always overturn your
  plans — this keeps design and history from blurring.
- **Secret management** — the agent knows what the _players_ know vs. what only _you_
  know. Minor secrets live inline; campaign-defining twists live in `secrets/`. When
  a secret is revealed in play, the agent promotes it to public history.

---

## Quick Start (5 minutes)

**You need:**

- An AI coding agent — [Claude Code](https://claude.ai/code), Cursor, Gemini CLI,
  Codex, or any agent that can read/write files
- A way to browse the wiki — the [Companion App](#companion-app-gui)
  (needs [Node.js](https://nodejs.org) and [pnpm](https://pnpm.io)), or optionally
  [Obsidian](https://obsidian.md) for backlinks and graph view. It's all plain markdown,
  so any editor works.
- [Git](https://git-scm.com) — strongly recommended (see "Git as a time machine" below)

**Setup:**

1. Click **"Use this template"** above (or clone this repo).
2. Open the folder in your AI agent — it reads `CLAUDE.md` automatically.
   _(Using a different agent? See "Agent config" below.)_
3. (Optional) Point the [Companion App](#companion-app-gui) at this folder to search
   the wiki and take notes at the table — or open the folder in Obsidian ("Open folder as vault").
4. Open `CLAUDE.md` and set the **output language** and **campaign name** at the top.
5. (Optional) Download the SRD — [5.1](https://dnd.wizards.com/resources/systems-reference-document)
   for the 2014 rules or 5.2.1 for the 2024 rules — into `raw/srd/` for monster/spell
   reference, and note the edition in `raw/world-bible.md` → `## Campaign`.

**Then either:**

- **Build your world first** — drop ideas into `raw/worldbuilding/`, tell the agent
  "design ingest this". You'll have a queryable world before session one.
- **Or just play** — after each session, drop a recap in `raw/sessions/session-NN.md`
  and tell the agent "ingest session NN".

---

## The three things you'll do

| Operation  | When               | What you say                                                                   |
| ---------- | ------------------ | ------------------------------------------------------------------------------ |
| **Ingest** | after a session    | "Ingest session 4" → agent updates 10-20 pages                                 |
| **Query**  | prepping a session | "What open threads involve the thieves' guild?"                                |
| **Lint**   | every few sessions | "Run a consistency check" → finds contradictions, dangling plots, orphan pages |

Between sessions you can also add **design** to `raw/worldbuilding/` (plans, ingested as
`prepared`) or an **interlude** to `raw/interludes/` (what happened off-screen, ingested
as `established` and unknown to the PCs).

---

## See it filled in

The [`examples/silverbrook-campaign/`](examples/silverbrook-campaign/) folder is a
complete worked example: a classic high-fantasy starter (a village, a tavern, a
nearby crypt) after two sessions of play. It shows real `[[wikilinks]]`, the
state/history split, a `prepared` antagonist faction that hasn't entered play yet,
and a `secrets/` file holding the campaign's main twist. **Open it in the companion app**
(`VAULT_PATH` set to `examples/silverbrook-campaign`, see below) **or in Obsidian's graph view**
to understand how the pieces connect.

---

## Folder structure

```text
your-campaign/
├── raw/                    # YOU write here — the agent never edits it (source of truth)
│   ├── sessions/           # session-01.md, session-02.md, ...
│   ├── worldbuilding/      # design material (becomes "prepared" entities)
│   ├── interludes/         # what happened off-screen between sessions
│   ├── setting/            # reference for a published setting or an inherited world
│   ├── srd/                # SRD 5.1 system reference (you download it)
│   ├── house-rules.md      # your mechanical variants
│   └── world-bible.md      # your lore pillars and tone
├── wiki/                   # the agent builds and maintains everything here
│   ├── world/  adventures/  characters/{pcs,npcs}/
│   ├── locations/{regions,cities,places,dungeons}/
│   ├── factions/  deities/  creatures/  lore/
│   ├── quests/{active,completed}/  items/  events/  secrets/
│   ├── index.md            # catalog of every page
│   └── log.md              # append-only history of operations
├── output/                 # session prep, player recaps (generated, not canon)
├── CLAUDE.md               # the agent's rulebook — the heart of the system
├── TEMPLATES.md            # the rules for every page type, for you and the agent
└── dm-second-brain.json    # vault format marker, read by apps (see below)
```

## Page templates

Every page type (NPC, city, quest, adventure, deity...) has a template,
`_template-<type>.md`, in its folder, and a card in [`TEMPLATES.md`](TEMPLATES.md) with
its fields, sections and links. You and the agent follow the same rules, so pages written
by hand and pages written on ingest look the same. In short:

- the file name is the page's **slug** (`elder-maren.md`) and never changes;
- links use it: `[[elder-maren|Elder Maren]]` in the text, `"[[elder-maren]]"` in the
  frontmatter, so they work in Obsidian, in the companion app and in lint;
- link fields such as `location`, `faction`, `owner` or a quest's `requires` connect
  pages, and the page they point to links back.

## Vault format

A campaign folder is a **vault**. Apps such as the [Companion App](#companion-app-gui) read it, and
`dm-second-brain.json` at its root says which format it follows:

```json
{ "vaultFormat": 1 }
```

The number changes only when the parts that apps rely on change. It does not change with
every template release. **Format 1** means:

- the `raw/` and `wiki/` folders, laid out as shown above;
- frontmatter keys and values in English: `type`, `status`, `canon_status`, `pg_known`,
  `first_seen`, `last_updated`, `tags`;
- `[[wikilinks]]` that point to a page's file name or title;
- DM-only text: pages in `wiki/secrets/`, entities with `pg_known: false`, and an inline
  `## Secrets (DM only)` section that runs to the end of its page;
- `_template-*.md`, `index.md` and `log.md` are scaffolding, not campaign pages.

An app declares which formats it supports. Leave the file as it is, unless an upgrade
note for the template tells you to change it.

## Git as a time machine

Commit after every session and each commit becomes a **snapshot of your world** at
that moment. The `git diff` after an ingest is your review step: see exactly which
pages the agent changed before you accept them. If it ever hallucinates, `git revert`
undoes it. **Keep your campaign repo private** — it contains your `secrets/`.

## Agent config

This repo ships with `CLAUDE.md` (for Claude Code). The same rules work in any agent;
just copy `CLAUDE.md` to the file your agent expects:

- Claude Code → `CLAUDE.md` (included)
- Cursor → `.cursor/rules/wiki.mdc`
- Gemini CLI → `GEMINI.md`
- OpenAI Codex → `AGENTS.md`

## Companion App (GUI)

The **[DM-Second-Brain App](https://github.com/tzoorlee/DM-Second-Brain-app)** is a local web app for use _live during sessions_.
It lives in its own repository and opens any campaign made from this template:

- **Search the wiki.** Ranked full-text search tolerates typos and accents
  ("silverbrok", "tomas" → "Tomás") and weighs titles and tags higher. It filters by
  type, `canon_status` and status.
- **View entity sheets** with DM-only secrets. A **Player Safe** mode hides `secrets/`
  pages, entities with `pg_known: false` and inline `## Secrets (DM only)` sections, so
  it's safe to share your screen.
- **Write session notes** with autocompletion for `[[wikilinks]]`. They are saved
  straight to this campaign's `raw/sessions/`, ready for the agent to ingest.

To use it, clone the app next to your campaign and point it at the campaign folder:

```bash
git clone https://github.com/tzoorlee/DM-Second-Brain-app.git
cd DM-Second-Brain-app && pnpm install
cp .env.example .env.local   # then set VAULT_PATH=../your-campaign
pnpm dev                     # open http://localhost:3000
```

The app checks the campaign's `dm-second-brain.json` (see [Vault format](#vault-format)),
so it never opens a campaign it can't read safely. It is meant for `localhost` only.

> The app acts as **you, the DM**, the human who curates the sources. It is not the
> agent. That's why it's allowed to write to `raw/`, even though the agent itself never
> does (see Rule #1 in `CLAUDE.md`).

## Optional tools (as your wiki grows)

None of these are needed to start — the `index.md` catalog is enough at small scale.
But a long-running campaign can reach hundreds of pages, and that's when they help:

- **[qmd](https://github.com/tobi/qmd)** — a local search engine for markdown files,
  for the *agent* (the companion app has its own search). Once your wiki passes ~100 pages,
  reading the index alone gets slow; qmd lets the agent search the wiki properly. Has
  both a CLI and an MCP server — see install instructions there. Mind the footprint:
  keyword search (`qmd search`) needs no models, but semantic search downloads
  local models on first use (`qmd vsearch` ≈ 300 MB, hybrid `qmd query` ≈ 2 GB).
- **[Obsidian](https://obsidian.md)** — an alternative way to browse the wiki, with
  backlinks and graph view. With the **Dataview** plugin you can query the YAML
  frontmatter to generate dynamic lists (e.g. "all living NPCs in Silverbrook", "all
  `prepared` entities"); the templates already include the frontmatter it needs.

The agent uses qmd automatically if it's installed; otherwise it falls back to the index.

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, run your campaigns with it.
The SRD 5.1 (which you download separately) is under CC-BY-4.0 and is not redistributed here.

## Credits

The core idea — dump raw sources, let an LLM compile a persistent wiki, browse it in
Obsidian — is [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).
This project adapts it for tabletop RPG campaigns. See [CREDITS.md](CREDITS.md).
