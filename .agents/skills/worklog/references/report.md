# Report

Use when the user wants a daily/weekly report or summary.

## Steps

1. Confirm the target date or range with the user.
2. Fetch entries with `list` for the date/range, and `search` for topics.
3. Compute the plan section's target date yourself; the CLI does not guess
   the next working day.
4. Choose a presentation: STAR (Situation/Task/Action/Result), by category,
   by timeline, or a concise list. Ask the user if unclear.
5. Format the output. The report layout rules are defined in PRD 5.2.9 and
   are the single source of truth; reference them, do not restate them.

## Notes

- Plans and logs are separate sections; keep them distinct.
- Do not invent grouping or sort rules beyond PRD 5.2.9.