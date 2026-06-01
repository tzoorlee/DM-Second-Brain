# 🎲 LLM D&D Campaign Wiki

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

- **Current state vs. history** — an NPC isn't just "dead", they died *in session 14*
  and that has consequences. The wiki tracks both what's true *now* and *how you got there*.
- **Prepared vs. established** (`canon_status`) — design a whole world before you play.
  Entities you've drafted are `prepared` (still malleable); the moment the players
  touch them in-game, they become `established` (canon). Players always overturn your
  plans — this keeps design and history from blurring.
- **Secret management** — the agent knows what the *players* know vs. what only *you*
  know. Minor secrets live inline; campaign-defining twists live in `secrets/`. When
  a secret is revealed in play, the agent promotes it to public history.

---

## Quick Start (5 minutes)

**You need:**
- [Obsidian](https://obsidian.md) — to browse the wiki (backlinks + graph view)
- An AI coding agent — [Claude Code](https://claude.ai/code), Cursor, Gemini CLI,
  Codex, or any agent that can read/write files
- [Git](https://git-scm.com) — strongly recommended (see "Git as a time machine" below)

**Setup:**
1. Click **"Use this template"** above (or clone this repo).
2. Open the folder in your AI agent — it reads `CLAUDE.md` automatically.
   _(Using a different agent? See "Agent config" below.)_
3. Open the same folder in Obsidian: "Open folder as vault".
4. Open `CLAUDE.md` and set the **output language** and **campaign name** at the top.
5. (Optional) Download the [SRD 5.1](https://dnd.wizards.com/resources/systems-reference-document)
   into `raw/srd/` for monster/spell reference.

**Then either:**
- **Build your world first** — drop ideas into `raw/worldbuilding/`, tell the agent
  "design ingest this". You'll have a queryable world before session one.
- **Or just play** — after each session, drop a recap in `raw/sessions/session-NN.md`
  and tell the agent "ingest session NN".

---

## The three things you'll do

| Operation | When | What you say |
|-----------|------|--------------|
| **Ingest** | after a session | "Ingest session 4" → agent updates 10-20 pages |
| **Query** | prepping a session | "What open threads involve the thieves' guild?" |
| **Lint** | every few sessions | "Run a consistency check" → finds contradictions, dangling plots, orphan pages |

---

## See it filled in

The [`examples/silverbrook-campaign/`](examples/silverbrook-campaign/) folder is a
complete worked example: a classic high-fantasy starter (a village, a tavern, a
nearby crypt) after two sessions of play. It shows real `[[wikilinks]]`, the
state/history split, a `prepared` antagonist faction that hasn't entered play yet,
and a `secrets/` file holding the campaign's main twist. **Open it in Obsidian and
explore the graph view** to understand how the pieces connect.

---

## Folder structure

```
your-campaign/
├── raw/                    # YOU write here — the agent never edits it (source of truth)
│   ├── sessions/           # session-01.md, session-02.md, ...
│   ├── worldbuilding/      # design material (becomes "prepared" entities)
│   ├── srd/                # SRD 5.1 system reference (you download it)
│   ├── house-rules.md      # your mechanical variants
│   └── world-bible.md      # your lore pillars and tone
├── wiki/                   # the agent builds and maintains everything here
│   ├── world/  locations/  characters/  factions/
│   ├── quests/  items/  events/  secrets/
│   ├── index.md            # catalog of every page
│   └── log.md              # append-only history of operations
├── output/                 # session prep, player recaps (generated, not canon)
└── CLAUDE.md               # the agent's rulebook — the heart of the system
```

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

## Optional tools (as your wiki grows)

None of these are needed to start — the `index.md` catalog is enough at small scale.
But a long-running campaign can reach hundreds of pages, and that's when they help:

- **[qmd](https://github.com/tobi/qmd)** — a local search engine for markdown files
  (hybrid BM25 + vector search, on-device). Once your wiki passes ~100 pages, reading
  the index alone gets slow; qmd lets the agent search the wiki properly. Has both a
  CLI and an MCP server. Install: `npm i -g @tobilu/qmd`
- **Obsidian Dataview** — query your YAML frontmatter to auto-generate dynamic lists
  (e.g. "all living NPCs in Silverbrook", "all `prepared` entities"). The templates
  already include the frontmatter Dataview needs.

The agent uses qmd automatically if it's installed; otherwise it falls back to the index.

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, run your campaigns with it.
The SRD 5.1 (which you download separately) is under CC-BY-4.0 and is not redistributed here.

## Credits

The core idea — dump raw sources, let an LLM compile a persistent wiki, browse it in
Obsidian — is [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).
This project adapts it for tabletop RPG campaigns. See [CREDITS.md](CREDITS.md).
