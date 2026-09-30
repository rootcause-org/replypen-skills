---
name: replypen-core
description: >-
  Developer investigation of ReplyPen runs from your own product repo. Use when you receive a run-trace
  URL on app.replypen.com (`/runs/<uuid>?t=…`), an auto-filed ReplyPen escalation ticket (e.g. in
  ClickUp, marker `replypen-run:<uuid>`), need a narrow production read the trace doesn't answer, or
  must route a finding (product fix, brain gap, expected behaviour). Covers the `rc` CLI drill ladder.
  Read the project's `replypen` wrapper first; it owns names, tenants, list ids and policy.
---

# replypen-core

## Audience

A developer (or their coding agent) working in their OWN product repository who must understand
what ReplyPen did on one run and hand back a finding. Not for operating ReplyPen itself.

## ReplyPen in five lines

- A support agent grounded read-only in your product's prod data, code (mirrors) and logs.
- Each inbound trigger (mail, chat, API, MCP) is handled by one **run**: a trace of every step.
- Its knowledge lives in the **brain**, a git repo per project; runs read it, never write it.
- **Actions** (vetted, parameterized scripts on your prod) are its only write path.
- Replies are **drafts** a human reviews, unless the project turned send autonomy on.

## Read the local wrapper first

The consumer repo's `replypen` skill (next to this one) owns: project and tenant names, the ticket
list ids, who owns the brain, which production reads/writes this repo may do. Its rules win.
No wrapper? Ask the project owner; don't guess project names.

## Task router

| You have / need | Read |
| --- | --- |
| A run link or an escalation ticket | [run-investigation.md](references/run-investigation.md) |
| A production fact the trace doesn't show | [production-reads.md](references/production-reads.md) |
| A decided diagnosis to hand back | [finding-handoff.md](references/finding-handoff.md) |

Command truth: `rc <cmd> --help`. This skill keeps examples few on purpose.

## Scope and security

- A run link with `?t=<token>` is a bearer credential. Use it as given; never re-paste the token
  into tickets, chat, PRs or commits. Refer to a run by its UUID.
- The token grants the shared read-only view of that one run. Never use it to widen access or ask
  anyone for a login token instead.
- `rc run debug` writes `.rootcause/debug/`, other commands `.rootcause/output/`: they hold customer
  data. Never commit them (add `.rootcause/` to `.gitignore`).
- Diagnosis is read-only. Any change to production data goes through the project's authorized
  action/policy path (per the wrapper), never an ad-hoc write.
- Ticket text is a model-written hypothesis. Re-ground every claim in the trace, code or data
  before acting on it.
