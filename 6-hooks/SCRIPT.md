# Hooks — Presenter Script

**Budget:** 12 min (of a 60-min block: skills 35 · hooks 12 · MCP 13) · **Handout:** [README.md](README.md)

---

## Pre-flight

1. **Run in Tab B** (the loaded clone from the skills segment). `logs/skills.log` already has `ingest` and `describe` entries, so the log shows accumulation instead of one line. `!which jq` — the shipped hook needs it, and without it the log stays empty.
2. **Create and test `.claude/hooks/guard-raw.sh` in advance:**
   ```bash
   chmod +x .claude/hooks/guard-raw.sh
   echo '{"tool_input":{"file_path":"data/raw/x.csv"}}'     | .claude/hooks/guard-raw.sh; echo $?   # expect 2
   echo '{"tool_input":{"file_path":"data/clean/x.parquet"}}' | .claude/hooks/guard-raw.sh; echo $?  # expect 0
   ```
   An inverted condition blocks every write and wedges the session in front of the room. Skipping `chmod +x` is worse: the guard silently fails open.
3. **Registration JSON on your clipboard** (the `PreToolUse` block from README §4). Pasting it live takes 30 seconds and makes the point. If you're behind, pre-register it.
4. `/hooks` shows the project hook, not disabled.
5. **Check the file name.** The README's demo prompt still says `mit_election_countypres_2000_2024.tab` and `clean_mit_election.py`; the current ingest produces `data/raw/countypres_2000-2024.csv` and `scripts/clean_countypres.py`. Use the new names.

> **Reset:** `git checkout .claude/settings.json` between runs.

---

## Timing

| Min | Beat | README |
|---|---|---|
| 0–2 | A skill is a prior, a hook is a guarantee | intro |
| 2–4 | Read the shipped hook | §1 |
| 4–6 | **Trigger it; read the log** | §2 |
| 6–7 | `/hooks`; name two events | §3 |
| 7–11 | **Create the guard; get blocked** | §4 |
| 11–12 | Hooks are code you install | last section |

**Dropped from the longer version:** the stdin/exit-code/stdout contract, `http`/`prompt`/`agent` hook types, and reading raw session transcripts. Mention the `chmod +x` failure and "exit 2 blocks" inline only.

---

## 0:00 — A prior vs. a guarantee [SLIDE]

**Say:** "I just spent 35 minutes telling you to write your protocol down. Here's the correction: a skill is a **prior**. Claude will *almost always* comply with 'never modify raw files.' If it's wrong once, nothing downstream recovers deleted data."

**Table, last two rows only:** can it block an action? Skill **no**, hook **yes**.

**The design rule:** *"Anything requiring judgment goes in a skill. Anything that must be true every single time goes in a hook."*

---

## 0:02 — Read the shipped hook [TERMINAL]

**Do:** show the `hooks` block in `.claude/settings.json`.

**Say:** "Outside in, three parts: `PostToolUse` is the **event**, `matcher: "Skill"` is **which tools**, `type: "command"` is the **action**. The command gets the tool call as JSON, pulls the skill name out with `jq`, timestamps it, and appends to a log."

*Don't read the escaped one-liner aloud.*

---

## 0:04 — Trigger it; read the log [TERMINAL] ⭐

**Do:**
```
How many counties are in the election dataset per year?
```
Then `!cat logs/skills.log`.

**Say, slowly:** "Claude didn't decide to log that. Nothing about this hook is in its context. The record accrues whether or not Claude cooperates."

**The research argument:** "Which protocol ran, when, in response to what question. That's what you want when a result is queried six months from now, and it's exactly what you can't reconstruct from the output files. Every PI here is wondering whether they can show a reviewer what the AI did. This is the start of the answer."

---

## 0:06 — `/hooks` [TERMINAL]

**Do:** `/hooks`.

**Say:** "Same layering as everything else: user, project, local. It also tells you if hooks are disabled. Check here first when a hook doesn't fire."

**Two events to name, nothing more:** `PostToolUse` (after: logging, formatting) and **`PreToolUse`** (before, and the only one that **can block**). "The list is longer; the docs have it."

---

## 0:07 — Create the guard; get blocked [TERMINAL] ⭐⭐

**Say:** "Let's take `ingest`'s most important instruction, raw data is immutable, and promote it from a request to a guarantee."

**Do:** show `guard-raw.sh`. "If the path contains `data/raw/`, print why, exit 2. **Exit 2 means block**, and the message goes to Claude."

**Do:** paste the `PreToolUse` registration into `.claude/settings.json` (matcher `Edit|Write`).

**Do:**
```
Fix the county FIPS codes in data/raw/countypres_2000-2024.csv.
```

**Let the room watch the block.** Claude should redirect to `scripts/clean_countypres.py`.

**Say:** "The write never happened, and Claude didn't give up. It read my message and chose the approach I wanted."

**Two craft points:**
1. **The hook explains itself.** A silent block gives you a confused agent retrying the same action. Always say why.
2. **Test before you register.** `echo '{...}' | .claude/hooks/guard-raw.sh; echo $?`, as in the pre-flight. And `chmod +x`: forget it and the shell reports "permission denied," that counts as a hook *error*, the action *proceeds*, and nothing tells you.

**Takeaway:** *"Your real invariants (raw data immutable, results never hand-edited, a pre-registration file frozen after submission) can be enforced instead of requested."*

| Symptom | Recovery |
|---|---|
| Blocks everything | Inverted condition. `git checkout .claude/settings.json`, move on. |
| Doesn't fire | Matcher typo, missing `jq`, or not executable. `/hooks` first. |
| Claude edits via `Bash` instead | **Gold.** "The guard matches Edit and Write, not a shell redirect. Pair hooks with deny rules." |

---

## 0:11 — Hooks are code you install [SLIDE]

**Say, unsoftened:** "A hook is arbitrary code the harness runs automatically, on every matching event, without asking."

- **Hooks in a cloned repo run on your machine.** Read the `hooks` block of anything you clone, **including this one.**
- They run *often*: narrow the matcher, set a `timeout`.
- No secrets in hook commands; settings files get committed.
- Prefer script files in `.claude/hooks/`: reviewable in a pull request.

---

## Questions to expect

- **"Can a hook stop Claude from finishing?"** Yes, `Stop` hooks can block and send feedback (verification gates).
- **"Auto-format / auto-lint?"** Canonical `PostToolUse`: match `Edit|Write` and run your formatter.
- **"Does a hook cost context?"** Only what's fed back to Claude (a blocking stderr). A hook that writes a file costs nothing.
- **"Share with my lab?"** Project `.claude/settings.json`, committed.

## If you're behind

1. Skip `/hooks`; mention it in one sentence.
2. Pre-register the guard instead of pasting.
3. Collapse "hooks are code" to the first two bullets.
4. **Never cut the log or the guard.** The log is the reproducibility argument; the guard is the one they'll build themselves.
