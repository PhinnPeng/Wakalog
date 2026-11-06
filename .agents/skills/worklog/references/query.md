# Query

Use when the user wants to look up, search, or list entries.

## Steps

1. Resolve relative date phrases to absolute dates yourself.
2. Run `list` with filters, or `search` with a keyword. Check the `truncated`
   field in the envelope; paginate with offset if needed.
3. For fuzzy search, start with `search` using the keyword. If nothing
   matches, retry with a shorter substring.
4. Chinese matching is whole-substring, not tokenized; adjust expectations.
5. For overdue plans, use the overdue filter.

## Notes

- Read the returned items; do not assume completeness when truncated is true.