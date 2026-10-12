# Hooks — Presenter Script

## Pre-flight

1. **Run in Tab B** (with pre-loaded data)
2. Create and test `.claude/hooks/guard-raw.sh`

```shell
#!/bin/sh
jq -e -r '.tool_input.file_path // "" | select(test("data/raw/"))' >/dev/null 2>&1 \
  && { echo "data/raw/ is immutable. Clean into data/clean/ instead." >&2; exit 2; }
exit 0
```
   
```bash
chmod +x .claude/hooks/guard-raw.sh
echo '{"tool_input":{"file_path":"data/raw/x.csv"}}'     | .claude/hooks/guard-raw.sh; echo $?   # expect 2
echo '{"tool_input":{"file_path":"data/clean/x.parquet"}}' | .claude/hooks/guard-raw.sh; echo $?  # expect 0
```

3. Prepare `PreToolUse` block

```json
"PreToolUse": [
  {
    "matcher": "Edit|Write",
    "hooks": [
      { "type": "command", "command": ".claude/hooks/guard-raw.sh" }
    ]
  }
]
```

**Reset:** `git checkout .claude/settings.json` between runs.

## 1. Hooks vs. Skills

- A skill is a **prior**. Claude will *almost always* comply with 'never modify raw files.'
   - If it's wrong once, nothing downstream recovers deleted data."

- A hook is a deterministic piece of code to execute.
   - "Always do this when..."
   - "Never do that when..."
   - **No context**

- **Design rule:** "Anything requiring judgment goes in a skill. Anything that must be true every single time goes in a hook."*

## 2. Read a hook

- Show the `hooks` block in `.claude/settings.json`.
   - 1. **Event** is `PostToolUse`
     2. **Matcher** is `Skill`
     3. **Action** is a shell command.
        - Receives tool call as JSON, extracts skill name, timestamps and appends to log file.

## 3. Trigger a hook ⭐

- Prompt 💬 "How many counties are in the election dataset per year?"

- Run `!cat logs/skills.log`.
   - Claude didn't decide to log that—we did.

## 4. List hooks

- Run `/hooks`.
   - Same layering as everything else: user, project, local.

## 5. Create a hook - guard ⭐⭐

- Raw data should be immutable. Currently a *suggestion*.

- Show `guard-raw.sh`
   - "If the path contains `data/raw/`, print why, exit 2.
   - **Exit 2 means block**, and the message goes to Claude.

- Paste the `PreToolUse` registration into `.claude/settings.json`

- Prompt 💬 "Fix the county FIPS codes in data/raw/countypres_2000-2024.csv."
   - Claude should redirect to `scripts/clean_countypres.py`.

- Note
   1. **The hook explains itself.** A silent block gives you a confused agent retrying the same action. Always say why.
   2. **Test before you register.** And `chmod +x` -  on hook error, the action *proceeds*, *silently*.

- Note
   - Claude might edit via `Bash` instead. **Pair hooks with deny rules.**

## 6. Warnings

- "A hook is arbitrary code the harness runs automatically, on every matching event, without asking."
   - Read the `hooks` block of anything you clone, **including this one.**
   - They might run *often*: narrow the matcher and set a `timeout`.
   - No secrets in hook commands; settings files get committed.
- Prefer script files in `.claude/hooks/`: reviewable in a pull request.

## Questions to expect

- **"Can a hook stop Claude from finishing?"** Yes, `Stop` hooks can block and send feedback (verification gates).
- **"Auto-format / auto-lint?"** Canonical `PostToolUse`: match `Edit|Write` and run your formatter.
- **"Does a hook cost context?"** Only what's fed back to Claude (a blocking stderr). A hook that writes a file costs nothing.
- **"Share with my lab?"** Project `.claude/settings.json`, committed.
