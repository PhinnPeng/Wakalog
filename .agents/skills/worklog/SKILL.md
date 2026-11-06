---
name: worklog
description: >
  Drive the workreport CLI to record completed work, plan future work, query and
  search entries, generate reports, and delete records. Use when the user says
  things like "记一条", "补录", "记计划", "查计划", "生成日报", "搜一下",
  "删掉那条", "帮我记今天的活", "what did I do". Not for editing the CLI
  itself, not for non-worklog tasks.
---

# Worklog skill

This skill orchestrates the `workreport` CLI. The CLI already describes itself;
this skill only holds orchestration knowledge the CLI cannot express.

## Contract source

Fetch the live contract once per session and again after any failed call:

`workreport describe --format json`

It returns commands, flags, error codes, exit codes, constraints, and limits.
Never cache it locally, never hand-copy its values into this skill (D6).
If an example here conflicts with `describe`, `describe` wins.

## Global disciplines

1. Branch on the `code` field of an error object, never on its `error` text.
2. Exit codes: `1` means fix params and retry once; `2` means re-read `describe`
   and correct the invocation; `3` means stop and hand to a human, no retry,
   no data rewrite.
3. `add` is not idempotent. If the result is unknown, run `list` to verify
   before deciding whether to retry.
4. Before `delete`, run `show` or `list` to get the id, restate the entry to
   the user, and confirm.
5. `purge` is the only irreversible operation; require both confirmations.

## Routing

| Intent | Load |
| :--- | :--- |
| 记录 / 补录 / 记一条 | [references/record.md](references/record.md) |
| 日报 / 汇报 / 总结 | [references/report.md](references/report.md) |
| 查 / 搜 / 有哪些 / 列出 | [references/query.md](references/query.md) |
| 删 / 删除 / 去掉 | [references/delete.md](references/delete.md) |
| Unclear / no args | Show a menu of 2-3 recommended next steps with reasons, then the full list. Do not auto-execute. |

## Relative dates

Resolve relative dates ("today", "yesterday", "last Friday") to absolute
`YYYY-MM-DD` using the user's local calendar, not UTC. The CLI does not guess.

## STAR and report formats

Report layout rules live in PRD 5.2.9 and are the single source of truth.
This skill references them; it does not restate grouping or sort rules.