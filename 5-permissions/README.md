# Permissions

## Basics

1. Show `data-analyst/.claude/settings.json` permissions
2. Compare to `data-analyst/.claude/settings.local.json` (may not exist yet)

## Allow

1. Demonstrate `/permissions` use to allow `WebFetch(domain:api.census.gov)`
2. Show the update to `settings.local.json`
3. Use the `/ingest` skill to fetch ACS data
4. Demonstrate that "Accept Always" updates `settings.local.json` (ask to checkout `main`)

## Permissions Mode

1. Demonstrate `acceptEdits` with `Shift + Tab`
2. Ask Claude to create a new file, `test.txt`

## Deny

1. Flip it - add a deny rule:
2. Ask Claude to delete `test.txt`: it should refuse.

```json
{
"permissions": {
    "deny": [
    "Bash(rm *)"
    ]
}
}
```

## Bonus

1. How to deal with secrets and `.env`?
2. The best practice, if possible, is to use environment variables, e.g., `export CENSUS_API_KEY`
