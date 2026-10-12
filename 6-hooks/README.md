# Hooks

A skill is a **prior**, not a guarantee. Consider `ingest` as an example. It says "Never modify raw files after saving — they are the source of truth," and Claude will *almost* always comply. A skill descriptions in a context window competes with your prompt, the conversation so far, and everything else Claude is weighing. Most of the time it wins, but sometimes — in a long session, an unusual phrasing, a task that half-matches two skills - it doesn't. For many instructions, "almost always" is fine. In critical cases, it is a catastrophe waiting: no amount of downstream care recovers deleted data.

Is there a way to force Claude to follow our rules?

**Hooks are the deterministic layer inside the chaos monkey.** A hook is a command the *harness* runs when a specified event occurs. The model does not decide whether to run it and cannot skip it (except by cleverly avoiding the triggering event).

| | Skill | Hook |
|---|---|---|
| Executed by| Model choice | The harness, unconditionally |
| Use case | Protocols, judgment, method | Invariants, logging, enforcement |
| Can it be skipped? | Yes | No |
| Costs context | Yes | No |
| Can it block an action? | No | Yes |

> [!NOTE]
> Hooks also solve a problem memory and skills can't touch: "every time X happens, do Y." No amount of instruction reliably produces that, because instructions are advisory.

## 1. Read a hook

`data-analyst` contains a single hook block in `.claude/settings.json`:

```json
"hooks": {
  "PostToolUse": [
    {
      "matcher": "Skill",
      "hooks": [
        {
          "type": "command",
          "command": "mkdir -p logs && jq -r '.tool_input | .skill + \" | \" + (.args // \"no args\")' | { read -r line && echo \"$(date '+%Y-%m-%d %H:%M:%S') | $line\" >> logs/skills.log; }"
        }
      ]
    }
  ]
}
```

Let's take this one part at a time, working from the outside inwards:

- **`PostToolUse`** — the triggering event. Fire *after* a tool has been used.
- **`matcher: "Skill"`** — which tools trigger the hook. Here, only the `Skill` tool; everything else is ignored.
- **`type: "command"`** — the action to perform. In this case, run a shell command.

The command itself receives a JSON object describing the tool call, pulls out the skill name and its arguments with `jq`, prepends a timestamp, and appends a line to `logs/skills.log`.

That's the whole anatomy: event, matcher, action.

## 2. Trigger a hook

Let's see this hook in action. The hook matches all `Skill` tool uses, so let's ask a question that requires a skill:

```
How many counties are in the election dataset per year?
```

Claude invokes `analyze` and answers. Now look at what the harness recorded without being asked:

```bash
❯ cat logs/skills.log
2026-03-20 07:54:14 | ingest | Fetch county-level national election data from MIT Election Lab
2026-10-08 14:22:03 | analyze | How many counties are in the election dataset per year?
```

A timestamped record of which protocol ran, when, and on what request.

*Claude didn't decide to log this*. Claude doesn't know the log exists — nothing about the hook is in its context (unless it has explored this repo, perhaps). The record accrues whether or not Claude is cooperating, which is the point.

## 3. Inspect a hook

```
/hooks
```

This opens a list of your configured hooks grouped by event. Select one to see what kind of hook it is and where it's configured.

Hooks follow the same inside-out layering as everything else: use `~/.claude/settings.json` for hooks you want everywhere, `.claude/settings.json` for hooks the project ships to collaborators, and `.claude/settings.local.json` for hooks that are yours alone.


### Know your events

`PostToolUse` is one of many events. The most useful of these include:

| Event | Fires | Typical use |
|---|---|---|
| `PreToolUse` | Before a tool runs — **can block it** | Enforce invariants, guard paths |
| `PostToolUse` | After a tool runs | Logging, formatting, linting |
| `UserPromptSubmit` | When you submit a prompt | Inject context, validate requests |
| `SessionStart` | At session start | Environment checks, status banners |
| `SessionEnd` | At session end | Cleanup, archive the log |
| `Stop` | When Claude finishes its turn | Verification gates, "not done yet" checks |
| `SubagentStop` | When a subagent finishes | Same, for delegated work |
| `PreCompact` | Before context is compacted | Preserve state that compaction would lose |
| `Notification` | On a notification | Custom alerting |

Check the [hooks documentation](https://code.claude.com/docs/en/hooks) for the full list.


## 4. Create a hook

Let's take `ingest`'s most important instruction — *raw data is immutable* — and promote it from a request to a guarantee.

Create `.claude/hooks/guard-raw.sh`:

```sh
#!/bin/sh
jq -e -r '.tool_input.file_path // "" | select(test("data/raw/"))' >/dev/null 2>&1 \
  && { echo "data/raw/ is immutable. Clean into data/clean/ instead." >&2; exit 2; }
exit 0
```

Make it executable (`chmod +x .claude/hooks/guard-raw.sh`), and register it in `.claude/settings.json`:

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

> [!WARNING]
> Forgetting to make a script executable is a quiet failure. The shell reports "permission denied" (exit code 126), is treated as a hook error, and the action *proceeds*. A guard that isn't executable doesn't block anything, and nothing tells you. Test the script before you rely on it.

Now ask Claude to do perform a forbidden action:

```
❯ Fix the county FIPS codes in data/raw/countypres_2000-2024.csv.

⏺ Write(data/raw/countypres_2000-2024.csv)
  ⎿  Blocked by hook: data/raw/ is immutable. Clean into data/clean/ instead.

⏺ Right — raw files are the source of truth. The FIPS normalization belongs in
  scripts/clean_countypres.py...
```

Notice the hook *explains itself*. Claude sees the message on `stderr` and uses it to choose a different approach. A hook that fails silently produces a confused agent retrying the same blocked action, so always say why.

### Treat hooks as code you're installing

A hook is arbitrary code the harness runs automatically on every matching event without asking. Take precautions:

- **Hooks in a cloned repo run on your machine.** `.claude/settings.json` can specify a hook, and if you clone a repo and start working, that command runs. Read the `hooks` block of any project you clone — including `data-analyst`.
- **They run often.** A slow `PostToolUse` hook on a broad matcher taxes every tool call. Use `timeout`, narrow the matcher, or run it async.
- **`PreToolUse` mistakes are loud.** A guard with an inverted condition blocks everything and the session becomes unusable. Test with the stdin trick before registering.
- **Don't put secrets in hook commands.** They're stored in settings files that are frequently committed. Read credentials from the environment inside the script, as module 4 recommended.

> [!TIP]
> One more reason to prefer script files over inline commands: a script in `.claude/hooks/` is reviewable in a pull request, and a reviewer can actually see what it does. A quoted one-liner buried in a JSON settings file is where mistakes go unnoticed.
