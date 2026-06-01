# Credits

## The core pattern
This project is built on the **LLM Wiki pattern** by **Andrej Karpathy**:
- Idea file: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

The foundational insight — instead of re-deriving knowledge from raw documents on
every query (RAG), have the LLM *incrementally build and maintain a persistent,
interlinked markdown wiki* — is entirely his. Browse it in Obsidian; the LLM writes,
you curate.

## What this project adds
This repo adapts that pattern for the specific demands of a tabletop RPG campaign:
- **Current state vs. history** split for evolving entities
- **`canon_status`** (prepared vs. established) to separate design from played history
- **Hybrid secret management** (inline minor secrets + dedicated major-plot files)
  with a promotion rule when secrets are revealed in play
- **D&D 5e-oriented templates** and a worked example campaign
- A **design-ingest** operation for building a world before session one

## Related / inspiration
- **second-brain** by Nicholas Spisak — another implementation of the LLM Wiki
  pattern (personal knowledge base), which popularized the four-operation framing
  (onboarding, ingest, query, lint): https://github.com/NicholasSpisak/second-brain

## System reference
- **D&D 5e SRD 5.1** by Wizards of the Coast, under CC-BY-4.0. Not redistributed
  in this repo — download it yourself from the official source.

## Optional tooling
- **qmd** by Tobias Lütke — local markdown search engine (BM25 + vector, on-device),
  recommended once the wiki grows large: https://github.com/tobi/qmd
