# Hooks

## 1. Understand what skills can't do

Module 3 made a case for skills: write your protocol down, and Claude will follow it. That case was slightly optimistic, and this module is the correction.

A skill is a **prior**, not a guarantee. It is text in a context window, competing with your prompt, the conversation so far, and everything else Claude is weighing. Most of the time it wins. Sometimes it doesn't — on a long session, an unusual phrasing, a task that half-matches two skills. `ingest` says "Never modify raw files after saving — they are the source of truth," and Claude will almost always comply. Almost.

For most instructions, "almost always" is fine. For a few, it isn't. If your raw data gets silently overwritten, no amount of downstream care recovers it.

**Hooks are the deterministic layer.** A hook is a command the *harness* runs when a specified event occurs. Claude does not decide whether to run it, cannot skip it, and cannot talk it out of its decision. The difference is categorical:

| | Skill | Hook |
|---|---|---|
| Who executes it | Claude, by choice | The harness, unconditionally |
| Can it be skipped | Yes | No |
| Costs context | Yes | No |
| Can it block an action | No | Yes |
| Good for | Protocols, judgment, method | Invariants, logging, enforcement |

The last row is the design rule. Anything requiring judgment belongs in a skill. Anything that must be true *every single time* belongs in a hook.

> [!NOTE]
> Hooks also solve a problem memory and skills can't touch: "every time X happens, do Y." No amount of instruction reliably produces that, because instructions are advisory. Hooks are the mechanism.

## 2. Read the hook this project ships

`data-analyst` has one. Look at the `hooks` block in `.claude/settings.json`:

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

Read it structurally, outside in:

- **`PostToolUse`** — the event. Fire *after* a tool has been used.
- **`matcher: "Skill"`** — which tools we care about. Here, only the `Skill` tool; everything else is ignored.
- **`type: "command"`** — run a shell command.
- The command itself receives a JSON object describing the tool call **on stdin**, pulls out the skill name and its arguments with `jq`, prepends a timestamp, and appends a line to `logs/skills.log`.

That's the whole anatomy: event, matcher, action.

## 3. Watch it fire

Ask a question that needs a skill:

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

Notice what *didn't* happen. Claude didn't decide to log this. Claude doesn't know the log exists — nothing about the hook is in its context. The record accrues whether or not Claude is cooperating, which is the entire point.

> [!NOTE]
> This is a reproducibility artifact, and it's worth naming as such for a research audience. "Which analysis steps ran, in what order, on what date, in response to what question" is exactly what you want when a result is queried six months later and your memory of the session is gone. It is also the thing you cannot reconstruct from the output files alone.

## 4. Inspect your hooks

```
/hooks
```

This opens a list of your configured hooks grouped by event. Select one to see what kind of hook it is and where it's configured — useful when a hook is firing and you can't remember which of the three settings files it came from.

Hooks follow the same inside-out layering as everything else: `~/.claude/settings.json` for hooks you want everywhere, `.claude/settings.json` for hooks the project ships to collaborators, `.claude/settings.local.json` for hooks that are yours alone.

> [!TIP]
> `/hooks` will also tell you when hooks are disabled altogether, or when your environment permits only administrator-managed hooks. If a hook you just wrote isn't firing, check here before debugging the command.

## 5. Know the events

`PostToolUse` is one of many. The ones you'll reach for most:

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

The list is longer than this — there are events for permission requests, model switches, subagent starts, and more — and it grows. Check the [hooks documentation](https://code.claude.com/docs/en/hooks) for the current set before building anything elaborate.

`PreToolUse` is the one with teeth, because it's the only one that can stop an action before it happens. Which brings us to the interesting demo.

## 6. Give an invariant teeth

Let's take `ingest`'s most important instruction — *raw data is immutable* — and promote it from a request to a guarantee.

Create `.claude/hooks/guard-raw.sh`:

```sh
#!/bin/sh
jq -e -r '.tool_input.file_path // "" | select(test("data/raw/"))' >/dev/null 2>&1 \
  && { echo "data/raw/ is immutable. Clean into data/clean/ instead." >&2; exit 2; }
exit 0
```

Make it executable, and register it in `.claude/settings.json`:

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

Now ask Claude to do the forbidden thing:

```
Fix the county FIPS codes in data/raw/mit_election_countypres_2000_2024.tab.
```

The write never happens. Claude receives the hook's message and adapts — typically by writing a cleaning step into `scripts/` instead, which is what you wanted in the first place.

```bash
❯ Fix the county FIPS codes in data/raw/mit_election_countypres_2000_2024.tab.

⏺ Write(data/raw/mit_election_countypres_2000_2024.tab)
  ⎿  Blocked by hook: data/raw/ is immutable. Clean into data/clean/ instead.

⏺ Right — raw files are the source of truth. The FIPS normalization belongs in
  scripts/clean_mit_election.py, which already zero-pads to 5 digits...
```

This is the pattern worth taking away. Your project's actual invariants — raw data is immutable, results are never hand-edited, the pre-registration file doesn't change after submission — can all be enforced rather than requested.

> [!TIP]
> Notice the hook *explains itself*. Claude sees the message on stderr and uses it to choose a different approach. A hook that fails silently just produces a confused agent retrying the same blocked action, so always say why.

## 7. Understand the contract

Three things to know to write hooks that work.

**Input arrives on stdin as JSON.** The object describes the event: the tool name, its inputs, the session. Explore it by making a hook that just dumps what it gets:

```sh
#!/bin/sh
cat >> /tmp/hook-debug.json
```

That is the fastest way to learn the shape of any event's payload, and much faster than reading documentation about it.

**Exit codes control the outcome.**

| Exit code | Meaning |
|---|---|
| `0` | Success. Continue. |
| `2` | **Block.** On blocking events, the action is prevented and stderr is fed back to Claude. |
| other | Hook error. Reported, but the action proceeds. |

**stdout can be structured.** If a hook writes a JSON object to stdout, it's parsed as a structured response — including permission decisions on `PreToolUse`, and `hookSpecificOutput.additionalContext` on `Stop` and `SubagentStop` to hand Claude feedback and let the turn continue. Note the sharp edge: a stdout `{...}` that isn't *valid* JSON is reported as a hook error, not treated as text. Keep simple hooks silent on stdout.

> [!TIP]
> Write hooks as script files in `.claude/hooks/`, not as inline one-liners in `settings.json`. Inline commands have to be JSON-escaped, which is how you get the thicket of backslashes in the skill-logging hook above. A script file can be tested directly:
>
> ```bash
> echo '{"tool_input":{"file_path":"data/raw/x.csv"}}' | .claude/hooks/guard-raw.sh; echo $?
> ```
>
> Test every hook this way before registering it. A broken hook on `PreToolUse` can wedge a session.

## 8. Go beyond shell commands

`type: "command"` is the common case, but not the only one. Hooks can also be:

- **`http`** — post the event to an endpoint and use its response. For centralized policy across a lab or institution.
- **`prompt`** — evaluate the event with a model rather than a script, for judgments too fuzzy to express as a condition.
- **`agent`** — hand the event to a subagent for genuinely involved checks.

These are powerful and correspondingly easy to misuse. A `prompt` hook reintroduces exactly the non-determinism you adopted hooks to escape — a hook written as "block commands that look dangerous" is a probabilistic filter wearing a deterministic costume. If you can express the condition as code, express it as code.

> [!NOTE]
> There's also scoping machinery worth knowing exists: per-hook `timeout` values, `if:` conditions to restrict a hook to certain paths, and async execution for slow hooks that shouldn't block a turn. Reach for these when a hook is slow or too broad, and note that `if:` glob semantics have their own rules — a single-segment `dir/**` matches only `<cwd>/dir`, so write `**/dir/**` for any-depth matching.

## 9. Read the full record

The skills log is a deliberately minimal view. Claude also keeps a complete transcript of every session, and for reconstructing what happened it's the authoritative source:

```bash
ls ~/.claude/projects/
cat ~/.claude/projects/<project>/<session>.jsonl | jq
```

One JSON object per line: every prompt, every tool call with its full input, every result, every response. It is verbose and immediately useful. To list the shell commands a session ran:

```bash
cat ~/.claude/projects/<project>/<session>.jsonl \
  | jq -r 'select(.message.content[]?.name == "Bash") | .message.content[].input.command' 2>/dev/null
```

Two honest caveats. These transcripts are local, unversioned, and tied to your machine — they are a debugging and audit resource, not something a collaborator can read. And they record what *Claude* did, not what you concluded; the analysis is still yours to write up.

> [!TIP]
> The transcript is also how you recover work. If a session went well and you want it documented, you don't have to remember what happened — ask Claude to read its own transcript and summarize what was done. That is often a better methods note than anything you'd write from memory a week later.

## 10. Treat hooks as code you're installing

A hook is arbitrary code the harness runs automatically, on every matching event, without asking. That is the source of both its usefulness and its risk.

- **Hooks in a cloned repo run on your machine.** Module 4 warned about third-party permissions; this is sharper. `.claude/settings.json` can specify a hook, and if you clone a repo and start working, that command runs. Read the `hooks` block of any project you clone — including `data-analyst`.
- **They run often.** A slow `PostToolUse` hook on a broad matcher taxes every tool call. Use `timeout`, narrow the matcher, or run it async.
- **`PreToolUse` mistakes are loud.** A guard with an inverted condition blocks everything and the session becomes unusable. Test with the stdin trick before registering.
- **Don't put secrets in hook commands.** They're stored in settings files that are frequently committed. Read credentials from the environment inside the script, as module 4 recommended.

> [!TIP]
> One more reason to prefer script files over inline commands: a script in `.claude/hooks/` is reviewable in a pull request, and a reviewer can actually see what it does. A quoted one-liner buried in a JSON settings file is where mistakes go unnoticed.
