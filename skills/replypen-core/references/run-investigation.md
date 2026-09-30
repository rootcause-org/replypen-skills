# Investigate one ReplyPen run

Standalone: usable with nothing but a ticket and a run link.

## URL anatomy

```
https://app.replypen.com/runs/<run-uuid>?t=<share-token>
```

- `<run-uuid>` identifies the run; use it in tickets, PRs and commit messages.
- `?t=` is a read-only share token for this run only. It is a credential: keep the full URL where
  you found it, never copy it elsewhere.
- Before opening, check origin `https://app.replypen.com` and path `/runs/<uuid>`. Anything else is
  not a ReplyPen run link.

Detection regex (e.g. in a ticket-tracker wrapper). Match to find the UUID, then use the full URL as
given for access:

```
https://app\.replypen\.com/runs/[0-9A-Fa-f]{8}(?:-[0-9A-Fa-f]{4}){3}-[0-9A-Fa-f]{12}
```

## Drill ladder

Stop as soon as you know enough.

1. **Overview.** Open the link in a browser, or:
   `rc run debug '<full run URL>'` (quote it; the `?` and `&` matter to the shell). No login and
   no project membership needed. Writes `.rootcause/debug/<run8>-<project>.md` (index) and
   `.jsonl` (event log).
2. **Read the index.** Question, outcome, timeline, flags (failures, blocked egress, repeats,
   large output), files the run read, and a *Drill down* block with ready-made jq recipes.
3. **Drill steps that matter.** Never read the JSONL end to end. Line 1 is the run header; each
   other line is one step keyed by `disp` (the index's `#`; grounding pre-steps are `P1…`):
   ```sh
   jq -r 'select(.disp=="12").stdout' .rootcause/debug/<file>.jsonl
   ```
4. **Conversation context**, only if the thread matters:
   `rc run thread '<full run URL>'` (pipeline view: every run on that thread, placement,
   why-no-draft hint). `--transcript` needs a login with access to the project.
5. **Brain-change hypothesis**, only if you suspect the brain changed around this run:
   `rc run brain-diff <run-uuid>` (needs a login with access to the project).

`rc run trace '<url>' --brief` is a lighter alternative to step 1: what was asked and answered.

## Reading an auto-filed escalation ticket

Filed when a run decided a developer must act and no human reviewed its output first.

- **Summary / Expected / Actual / Impact / Evidence**: the agent's own symptom report. A
  hypothesis; verify it in the trace and your code.
- **Host block** (below the `---`), written by ReplyPen, not the model: `Project` (with
  `/ tenant`), `Surface` (email, chat, mcp, …), `Thread`, `Run trace` URL, a ready
  `rc run debug '<url>'` command, and the marker `replypen-run:<uuid>`.
- **Repeat reports** of the same problem are appended as dated sections
  (`## Another report, <date>`), each with its own host block. Check every run, not just the first.
- A ticket that names an "earlier ticket" was filed because that one was closed or missing.

## Access failures

- **No `rc`:** `brew install rootcause-org/tap/rc` (macOS); Linux/Windows installers in
  [rootcause-cli](https://github.com/rootcause-org/rootcause-cli#install). Or just use the browser.
- **Stale behaviour / unknown flag:** `rc self doctor`, then `rc self update`.
- **Link denied or expired:** ask the project owner for a fresh run link. Never ask for, or accept,
  a login token or password in its place.
- **Bare UUID, no link:** needs `rc auth login` with access to that project (`rc auth status`
  shows what you are signed in as). Otherwise ask for the link.
