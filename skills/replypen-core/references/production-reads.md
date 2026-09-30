# Narrow production reads

For when the trace shows what the agent saw, but you need a fact it didn't fetch.

## Order of preference

1. **Your product repo's own read-only tooling** (console scripts, admin views, log search) as
   the local wrapper describes. You know those semantics best.
2. **The data the run already pulled.** jq the step's `stdout` in the debug JSONL before querying
   again; it is the exact state the agent reasoned on, at that moment.
3. **ReplyPen's guarded consoles**, only if the wrapper grants them to your login:
   `rc dev console capabilities` lists what you may use; `rc dev console database query` and
   `rc dev console bash run` are read-only by default. Check `--help` for flags; keep queries
   narrow (IDs, `limit`), verify schema first.

## Not a read

- `rc ask "<question>"` **creates a production run**: it costs, is journaled, and may feed the
  brain's learning loop. Use it only to reproduce agent behaviour, and prefer `--simulation`
  (nothing placed, no action, no journal) when the wrapper allows `rc ask` at all.
- `rc run retry` re-runs a run. Same caution.

## Never

- Ad-hoc production writes (SQL, consoles, scripts). Writes go through the project's action or
  policy path per the wrapper, with the required human approval.
- Pasting secrets, DSNs, tokens or share URLs into prompts, tickets, logs or commits.
- Committing `.rootcause/` output; it contains customer data.
