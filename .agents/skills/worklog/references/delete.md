# Delete

Use when the user wants to remove an entry.

## Steps

1. Run `show` or `list` to locate the entry and get its id.
2. Restate the entry content to the user and confirm before deleting.
3. Run `delete`. On failure, branch on `code`.
4. `restore` can bring back a soft-deleted entry while its id is still held.
5. `purge` permanently removes expired tombstones and requires both an age
   threshold and an explicit confirmation. Treat it as irreversible.

## Notes

- Never delete without the user's confirmation.
- Prefer soft delete; reserve purge for explicit cleanup.