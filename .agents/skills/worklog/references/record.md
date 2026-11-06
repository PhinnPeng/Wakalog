# Record

Use when the user wants to record completed work or a plan.

## Steps

1. Resolve the date. Default to the user's local today. If the user names a
   date, use it. Convert relative phrases to absolute `YYYY-MM-DD`.
2. Run `categories` first to normalize the category against existing ones.
3. Choose `item_type`: default `log` for completed work, `plan` for future work.
   A plan requires a planned date.
4. Build the command. If content has quotes, newlines, or is near the length
   limit, read it from stdin instead of the command line.
5. Run `add`. If it fails, branch on `code`:
   - fix params and retry once on a validation failure;
   - re-read the contract on a usage failure;
   - stop and hand to a human on an environment failure.
6. If the result is unknown, run `list` to verify before retrying.

## Notes

- Status defaults to done for logs and planned for plans.
- Never silently fix conflicting fields; surface them.