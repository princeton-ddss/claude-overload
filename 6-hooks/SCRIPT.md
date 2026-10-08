# Module 6: Hooks — Presenter Script

**Duration:** 20 min · **Audience handout:** [README.md](README.md)

---

## Pre-flight

1. **`logs/skills.log` should already have 1–2 entries** from earlier modules, so §3 shows accumulation rather than a single line. If you ran module 3 live in the same clone, you're set. Otherwise seed it by asking a skill-triggering question beforehand.
2. **Write and test `guard-raw.sh` in advance.** Create `.claude/hooks/guard-raw.sh` and verify both branches:
   ```bash
   echo '{"tool_input":{"file_path":"data/raw/x.tab"}}' | .claude/hooks/guard-raw.sh; echo $?   # expect 2
   echo '{"tool_input":{"file_path":"data/clean/x.parquet"}}' | .claude/hooks/guard-raw.sh; echo $?  # expect 0
   ```
   **Do not skip this.** A `PreToolUse` hook with an inverted condition blocks every write and wedges the session in front of the room.
3. **Decide: register the guard live or pre-register it.** Live is better theater (§6 lands harder when they watch you add it), but pre-registered is safer. If live, have the JSON block on your clipboard.
4. `!which jq` — the shipped hook pipes through it. If it's missing, the log stays empty and you'll be debugging instead of teaching.
5. Pick a session for §9 and have the `jq` command ready. A session with plenty of `Bash` calls demos better.
6. `/hooks` → confirm the project hook is listed and not disabled.

> **Reset between runs:** if you present this twice, `git checkout .claude/settings.json` and clear `logs/skills.log` so the demo starts clean.

---

## Timing

| Time | Beat | § |
|---|---|---|
| 0:00 | What skills can't do — the determinism gap | 1 |
| 0:03 | Read the shipped hook | 2 |
| 0:05 | **Watch it fire** | 3 |
| 0:08 | `/hooks`, and the event table | 4–5 |
| 0:10 | **Give an invariant teeth** | 6 |
| 0:14 | The contract: stdin, exit codes, stdout | 7 |
| 0:16 | Beyond shell commands | 8 |
| 0:17 | The full transcript | 9 |
| 0:19 | Hooks are code you're installing | 10 |

**The two beats that matter are §3 and §6.** §3 is the reproducibility payoff; §6 is the one they'll remember. Everything else is scaffolding.

---

## 0:00 — What skills can't do [SLIDE]

**Open by walking back module 3.** This is the honest framing and it earns credibility:

**Say:** "I spent forty minutes telling you to write your protocol down and Claude will follow it. That was slightly optimistic. Here's the correction."

**Say:** "A skill is a **prior**, not a guarantee. It's text in a context window competing with your prompt and everything else. Most of the time it wins. On a long session, an odd phrasing, a task that half-matches two skills — sometimes it doesn't."

**Make it concrete:** "`ingest` says 'never modify raw files, they are the source of truth.' Claude will almost always comply. *Almost.* And if your raw data gets silently overwritten, nothing downstream recovers it."

**Put up the comparison table.** Read only the last two rows:
- Can it block an action? Skill **no**, hook **yes**.
- Good for: skills → protocols, judgment, method. Hooks → invariants, logging, enforcement.

**The design rule to leave them with:** *"Anything requiring judgment belongs in a skill. Anything that must be true every single time belongs in a hook."*

---

## 0:03 — Read the shipped hook [TERMINAL]

**Do:** Show the `hooks` block in `.claude/settings.json`.

**Say:** "Read it outside in. Three parts, and that's the whole anatomy."
- `PostToolUse` → **the event**
- `matcher: "Skill"` → **which tools** — here, only the Skill tool
- `type: "command"` → **the action**

**Say:** "The command gets a JSON object about the tool call on **stdin**, pulls the skill name out with `jq`, timestamps it, appends to `logs/skills.log`."

> Don't read the escaped one-liner aloud. Point at the backslash thicket and say "we'll fix this in §7 — write hooks as script files."

---

## 0:05 — Watch it fire [TERMINAL] ⭐

**Do:**
```
How many counties are in the election dataset per year?
```

**While it runs:** "Claude is picking `analyze` for this."

**Then — the reveal:**
```bash
!cat logs/skills.log
```

**Say, slowly:** "Here's what matters. Claude didn't decide to log that. Claude doesn't know the log *exists* — nothing about the hook is in its context. The record accrues whether or not Claude is cooperating."

**Then make the research argument — this is the beat for your audience:**

**Say:** "'Which analysis steps ran, in what order, on what date, in response to what question.' That's a reproducibility artifact. It's what you want when a result gets queried six months from now and your memory of the session is gone — and it's exactly what you *cannot* reconstruct from the output files."

**Optional, if the room is engaged:** "Every PI here is privately wondering whether they can show a reviewer what the AI did. This is the answer."

---

## 0:08 — /hooks and the events [TERMINAL]

**Do:** `/hooks` → shows hooks grouped by event; select one to see its type and where it's configured.

**Say:** "Same inside-out layering as everything else — user, project, local. And this screen tells you if hooks are disabled entirely, or if your environment only allows admin-managed ones. Check here before debugging a hook that isn't firing."

**Do:** Put up the event table. **Don't read all nine.** Name four and move:
- `PreToolUse` — before a tool, **can block**
- `PostToolUse` — after; logging, formatting, linting
- `UserPromptSubmit` — inject context on every prompt
- `Stop` — when Claude finishes a turn; verification gates

**Say:** "The list is longer and it grows. `PreToolUse` is the one with teeth — it's the only one that can stop an action before it happens."

---

## 0:10 — Give an invariant teeth [TERMINAL] ⭐⭐

**Frame it as a callback:**

**Say:** "Let's take `ingest`'s most important instruction — raw data is immutable — and promote it from a *request* to a *guarantee*."

**Do:** Show `guard-raw.sh` (4 lines). Read the logic aloud: "if the file path contains `data/raw/`, print why, exit 2."

**Do:** Register it in `settings.json` with the `Edit|Write` matcher (paste from clipboard).

**Do:** Ask for the forbidden thing:
```
Fix the county FIPS codes in data/raw/mit_election_countypres_2000_2024.tab.
```

**Let the room watch the block.** Then watch Claude adapt — it should redirect to `scripts/clean_mit_election.py`.

**Say:** "The write never happened. And notice Claude didn't just give up — it got the hook's message on stderr and chose a different approach. The one I wanted in the first place."

**The takeaway:** *"Your project's real invariants — raw data immutable, results never hand-edited, the pre-registration file frozen after submission — can all be enforced instead of requested."*

**One craft point, say it:** "The hook **explains itself**. A hook that fails silently just gives you a confused agent retrying the same blocked action. Always say why."

### Failure modes

| Symptom | Recovery |
|---|---|
| Hook blocks everything | Your condition is inverted. `git checkout .claude/settings.json`, move on, fix at the break. **Test beforehand.** |
| Hook doesn't fire | Matcher typo, or `jq` missing. Check `/hooks` first. |
| Claude edits via `Bash` instead | **Gold.** "The guard matches Edit and Write, not a shell redirect. Defense in depth — pair hooks with module 4's deny rules." |
| Claude argues rather than adapting | Your stderr message wasn't actionable. Good illustration of the craft point. |

---

## 0:14 — The contract [TERMINAL]

**Three things, fast.**

**1. Input on stdin as JSON.** The best way to learn any event's payload:
```sh
#!/bin/sh
cat >> /tmp/hook-debug.json
```
**Say:** "Faster than reading the docs for the shape."

**2. Exit codes.** `0` continue · `2` **block** (stderr goes to Claude) · anything else = hook error, reported, *action proceeds anyway*.

**3. stdout can be structured JSON** — permission decisions on `PreToolUse`, `additionalContext` on `Stop`. **Sharp edge:** a stdout `{...}` that isn't valid JSON is a hook error, not text. Keep simple hooks silent on stdout.

**Then the practical advice, and show it:**

**Say:** "Write hooks as script files in `.claude/hooks/`, not inline in settings.json. That's why the shipped hook is a backslash thicket. And a script file you can test:"

```bash
!echo '{"tool_input":{"file_path":"data/raw/x.csv"}}' | .claude/hooks/guard-raw.sh; echo $?
```

**Say:** "Test every hook this way before registering it. A broken `PreToolUse` hook can wedge a session."

---

## 0:16 — Beyond shell commands [SLIDE]

**Say:** "`command` isn't the only type. There's `http` — post the event to an endpoint, for centralized lab or institutional policy. `prompt` — evaluate with a model. `agent` — hand it to a subagent."

**Then the warning, which is the actual content of this beat:**

**Say:** "Be careful with `prompt` hooks. A hook that says 'block commands that look dangerous' reintroduces exactly the non-determinism you adopted hooks to escape — it's a probabilistic filter wearing a deterministic costume. **If you can express the condition as code, express it as code.**"

**Mention in one line:** per-hook `timeout`, `if:` path conditions, async execution for slow hooks.

---

## 0:17 — The full transcript [TERMINAL]

**Say:** "The skills log is a deliberately minimal view. Claude keeps everything."

**Do:**
```bash
!ls ~/.claude/projects/
!cat ~/.claude/projects/<project>/<session>.jsonl | jq 'select(.type=="user") | .message.content' | head -20
```

**Say:** "One JSON object per line. Every prompt, every tool call with full input, every result."

**Two caveats — say both, they matter:** "These are local, unversioned, tied to your machine. A debugging and audit resource, not something a collaborator reads. And they record what *Claude* did, not what you concluded."

**The genuinely useful trick:** "If a session went well and you want it documented, don't try to remember it — **ask Claude to read its own transcript and summarize what was done.** That's usually a better methods note than anything you'd write from memory a week later."

---

## 0:19 — Hooks are code you're installing [SLIDE]

**Say, and don't soften it:** "A hook is arbitrary code the harness runs automatically, on every matching event, without asking."

**Four bullets:**
1. **Hooks in a cloned repo run on your machine.** Module 4 warned about third-party permissions; this is sharper. Read the `hooks` block of anything you clone — **including `data-analyst`.**
2. They run *often* — a slow hook on a broad matcher taxes every tool call.
3. `PreToolUse` mistakes are loud. Test with the stdin trick.
4. No secrets in hook commands; settings files get committed.

**Close:** "One more reason to use script files: a script in `.claude/hooks/` is reviewable in a pull request. A quoted one-liner in a JSON settings file is where mistakes go unnoticed."

---

## Questions to expect

- **"Can a hook stop Claude from finishing?"** — Yes, `Stop` hooks can block and send feedback, which is how verification gates work. Useful for "did the tests actually pass."
- **"Can I use hooks to auto-format / auto-lint?"** — Canonical `PostToolUse` use. Match `Edit|Write` and run your formatter.
- **"Can hooks see my prompts?"** — Yes, `UserPromptSubmit` receives them. Relevant if you're thinking about an `http` hook sending events off-machine.
- **"Does the hook's output cost context?"** — Only when it's fed back to Claude: a blocking `exit 2`'s stderr, or `additionalContext`. A hook that just writes a file costs nothing.
- **"How do I share hooks with my lab?"** — Project `.claude/settings.json`, committed. Which is also why your collaborators should read them.
- **"What if a hook is slow?"** — `timeout`, a narrower matcher, or async.

## If you're behind

Cut in this order:
1. §8 beyond shell commands → keep only the "express it as code" warning
2. §9 transcripts → keep only the "ask Claude to summarize its own transcript" trick
3. §5 event table → name `PreToolUse` and `PostToolUse`, skip the rest
4. §7 contract → keep exit codes, drop stdin/stdout detail
5. **Never cut §3 or §6.** The log is the reproducibility argument; the guard is the one they'll build themselves.
