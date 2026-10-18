# Examples

## silverbrook-campaign/
A complete worked example: a classic high-fantasy starter after two sessions of play.

Open it in the [companion app](https://github.com/tzoorlee/DM-Second-Brain-app). In the
app's `.env.local`, set `VAULT_PATH` to this folder, for example
`VAULT_PATH=../DM-Second-Brain/examples/silverbrook-campaign` with both repos side by side.
Then try the **Player Safe** toggle. Or open it in Obsidian
("Open folder as vault", select `silverbrook-campaign`) and explore the **graph view**
to see how entities link together.

What it demonstrates:
- The **state/history split** ([[silverbrook]] + [[silverbrook-history]])
- **`canon_status`**: most entities are `established` (played), but the
  [[order-of-the-broken-crown]] faction and [[the-broken-crown]] secret are
  `prepared` — designed, not yet touched by play
- **Secret management**: minor secrets inline (e.g. on [[elder-maren]]'s page),
  the major plot in `secrets/`
- A populated `index.md` and `log.md`

Read the `raw/sessions/` files first, then see how the agent would have compiled
them into `wiki/`.
