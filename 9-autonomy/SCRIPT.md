# Module 9: Working Autonomously — Presenter Script

**Duration:** 25 min · **Audience handout:** [README.md](README.md)

> **This is the capstone.** It draws on modules 4 (permissions), 6 (hooks), and 8 (subagents), and it's the module people will actually act on Monday morning. If the workshop is running long, cut from modules 7 and 8, not from this one.

---

## Pre-flight

1. **Run `/sandbox` on your presentation machine and know what it says.** Defaults and availability vary by platform; you need to know whether filesystem isolation is on before you demo it. If it's unavailable, present §4 as "here's the mechanism and the setting names" rather than live.
2. **Have `/fewer-permission-prompts` already run once** in a scratch clone, so you can show real proposed rules from real transcripts rather than an empty result.
3. **A `/goal` run ready to start**, with a *procedural* completion condition. Start it at §6 and let it run through §7–§8 in the background so there's something to verify at §8.
4. **A diff to read at §8.** Either from the goal run or pre-made. The point is to show reading a diff rather than a transcript — have `git diff --stat` ready.
5. **Know the §6 numbers:** max-of-5 inflation is +0.019 at n=620, +0.009 at n=3,100; ~3,155 counties per election year.
6. **`logs/skills.log` with entries**, so §5's "turn on logging first" has something behind it.
7. Check `/usage` before you start, so §10 can quote a real cost.

---

## Timing

| Time | Beat | § |
|---|---|---|
| 0:00 | **The two failure modes + the principle** | 1 |
| 0:03 | Start in plan mode | 2 |
| 0:05 | Stop answering the same prompt twice | 3 |
| 0:08 | **Use the sandbox** | 4 |
| 0:11 | Set the floor that must hold | 5 |
| 0:14 | **Give it a goal — specify a procedure** | 6 |
| 0:19 | Let it run, check in deliberately | 7 |
| 0:21 | Verify before you believe it | 8 |
| 0:23 | Keep an undo | 9 |
| 0:24 | **The recipe** | 10 |

---

## 0:00 — The two failure modes [SLIDE] ⭐

**Open by naming what they're all actually doing.** Ask the room: "Who has approved a permission prompt without reading it?" Hands go up. That's your opening.

**Failure mode one — prompt fatigue:** "Manual mode, forty prompts an hour, and you start approving without reading. The permission system is still technically on, but it's stopped working. You've become a rubber stamp — and the thing it was protecting you from gets approved along with everything else."

**Failure mode two — blind trust:** "So you flip to auto mode, stop watching, and three days later a cleaning script has changed and you can't tell which results came from which version. Nothing dramatic. You just lost the thread."

**Say:** "Most people oscillate between these. The way out isn't a point in between."

**Put the principle up alone on a slide:**

> **Autonomy is bought with guardrails, not with trust.**

**Say:** "Every prompt you stop answering should be replaced by something structural — an allow rule, a deny rule, a sandbox boundary, a hook, a frozen data split, a throwaway branch. You want a session where you're rarely prompted *because* the things that would need a prompt are now impossible. Not because you turned the prompts off."

---

## 0:03 — Start in plan mode [TERMINAL]

**Do:** `Shift+Tab` to cycle to Plan. Then:
```
Plan how to add a county-level fixed effects specification to the turnout analysis.
```

**Say while it explores:** "Read-only. Nothing changes while it looks."

**The argument — this is why it's first:** "Most bad autonomous runs aren't caused by a bad tool call. They're caused by Claude confidently solving a slightly different problem than the one you had. Plan mode surfaces that misunderstanding while it's still free to fix."

**Say:** "Cheapest habit in this module, highest return. It's also the right default for exploring any unfamiliar repo."

**If asked about auto mode:** "They compose. In plan mode, Bash commands that can't be proven read-only get judged by the auto-mode classifier instead of prompting you — so exploration stays fluid without becoming write-capable."

---

## 0:05 — Stop answering the same prompt twice [TERMINAL]

**Say:** "If a prompt has appeared three times and you approved it three times, that's not a safety check. It's a tax."

**Do:** `/fewer-permission-prompts`

**Say:** "It scans your transcripts for the read-only Bash and MCP calls you keep approving and proposes an allowlist for the project settings. Grounded in what you actually did — not what you imagine you'll need."

**Show the proposed rules.** Then two principles:
- **Allow generously on reads, stingily on writes.** `Read(data/**)` costs nothing. `Edit(data/raw/**)` is a decision.
- **Compound commands** — module 4's warning: `mkdir demo && rm -rf test` needs *both* parts allowed. "Claude chains constantly, which is why a hand-written allowlist always has holes that a transcript-based one catches."

---

## 0:08 — Use the sandbox [TERMINAL] ⭐

**Say:** "This is the mechanism that makes real autonomy defensible, and it's the one most people miss."

**Do:** `/sandbox`

**Say:** "A sandbox constrains what a command *can do* instead of asking whether it should. Two layers."
- **Filesystem isolation** — restricted view, `denyRead` regions, `allowRead` exceptions
- **Network egress** — a domain allowlist, `deniedDomains` to carve out of a wildcard, strict mode that refuses non-allowlisted hosts without prompting

**Mention:** "There's also a `sandbox.credentials` setting that blocks sandboxed commands from reading credential files and secret env vars outright — the enforced version of module 4's advice about API keys."

**Then connect it to the thesis — this is the beat's point:**

**Say:** "Here's why this is the key mechanism. Because a sandboxed command is constrained, **it can run unprompted.** Sandbox auto-allow means interpreter calls proceed without a dialog, because the sandbox is doing the work the dialog was doing. Fewer prompts *and* more enforcement. That's the trade you want."

**The institutional hook — say this one slowly:** "Network egress control is the feature to raise with your research computing office. Module 5 said a remote MCP server is a transmission to a third party. A domain allowlist is the technical answer to 'can you guarantee this data doesn't leave?' That's a much better conversation to have with an allowlist in hand."

> **Honesty note:** if `/sandbox` shows filesystem isolation unavailable on your machine, say so. Defaults vary by platform. Present the mechanism and the setting names rather than pretending.

---

## 0:11 — Set the floor that must hold [TERMINAL]

**Say:** "Allowlists and sandboxes reduce prompting. They don't express invariants. For the few things that must *never* happen, use the two mechanisms that can't be argued with."

**Deny rules** — "refuse outright, regardless of mode. Auto mode respects them; the classifier never overrides them. This is where 'never touch the pre-registration file' lives."

**Hooks** — show the four-line `guard-raw.sh` from module 6 again.

**Say:** "Four lines, and it holds whether you're watching or not. That's what makes unattended work reasonable — not that Claude is trustworthy, but that the damaging actions are *unavailable*."

**Then give them the exercise. This is the most actionable thing in the module:**

**Say:** "Before any long run: **name the three worst things that could happen, and write a deny rule or a hook for each.** If you can't name them, you aren't ready to leave it unattended — and you just learned that cheaply."

**One more:** "Install the logging hook *before* the run, not after. An audit trail only helps if it was recording while you weren't looking."

---

## 0:14 — Give it a goal, specify a procedure [TERMINAL] ⭐⭐

**Do:** Start the goal run now so it works through the next two beats:
```
/goal every specification in scripts/ has been fit and its coefficients written to results/
```

**Say:** "Sets a completion condition and keeps working across turns until it's met. Live elapsed, turns, and tokens in the overlay — cost visible while it runs. Works interactive, in `-p`, and through Remote Control."

**Mention the robustness briefly:** "Retries with backoff through API errors, pauses and says why on a usage limit, survives compaction, restores on `--resume`, clears itself if a turn dies. It's hook-driven internally — so it won't work with hooks disabled, and it tells you instead of hanging."

**Then pivot, and slow down. This is the beat that matters:**

**Say:** "How you phrase the completion condition determines whether the result means anything. Consider a goal I could plausibly have set instead:" → put up **"hold-out R² above 75%"**

**Ask:** "What's wrong with it?"

**Take the overfitting answer, then concede it with the arithmetic:**

| Test set | SD of one draw | Max-of-5 inflation |
|---|---|---|
| 620 counties | 0.017 | +0.019 |
| 3,100 counties | 0.008 | +0.009 |

**Say:** "Three thousand counties per election year; a 20% split is 620. Taking the best of five inflates R² by one to two points, growing slowly. Worth knowing. Not a crisis."

> Conceding here is the move. Researchers will discount a hand-wave and then actually listen to what follows.

**Then the real problem:** "Raising a hold-out R² doesn't require trying more regressions. It can be done by fitting the scaler before splitting, re-drawing the split with a new seed, dropping outlier counties, switching from total share to two-party share, or building a predictor that encodes the outcome."

**Say:** "None of those is a multiplicity problem. Each voids the hold-out guarantee entirely. And each is a locally reasonable-looking step that gets summarized in one approving sentence. There's no bound on the error, because the estimate has stopped estimating anything."

**Then the structural point:** "On top of that, a results threshold is **unverifiable by the harness.** Nothing can check 'R² above 75%' for you — so the completion condition is self-reported by the agent that wants to finish."

**Put up the four-row table.** Read the last row: harness can't verify it / harness *can* — the files either exist or they don't.

**Close the beat by tying it to §5:** "Notice what that last row means. A procedural completion condition is one you could **write a hook for.** A results threshold never is. The guardrail and the goal have to be stated in the same vocabulary."

**Generalize, briefly:** "When an agent iterates unsupervised toward a metric, the risk isn't the multiple-comparisons arithmetic — that's bounded and estimable. It's that the agent is choosing the analysis *and* reporting on it, with the degrees of freedom to move what you're measuring. Garden of forking paths. The agent's contribution isn't a bigger garden, it's a faster walk through it."

> If the room goes quiet here, let it. That's thinking.

---

## 0:19 — Let it run, check in deliberately [TERMINAL]

**Do:** `/background` (or `/bg`), then `/tasks`

**Say:** "`/tasks` lists background work and lets you stop any of it. `/remote-control` follows a session from your phone or claude.ai — genuinely useful for a run you started before leaving the office."

**The habit to name:** "Check in **on a schedule**, not continuously. Watching continuously is how you end up rubber-stamping. Checking in at intervals is how you stay able to evaluate."

**One clarification, since they met ralph-loop in module 7:** "`/loop` is a different tool — it recurs on an interval, or self-paced. `/goal` persists until a condition is met. `/usage` has a per-loop breakdown so a chatty loop is easy to spot."

---

## 0:21 — Verify before you believe it [TERMINAL]

**Say:** "An autonomous run ends with a confident summary. The summary is not evidence."

**Do:** `/verify`

**Say:** "Checks that a change does what it was supposed to. Note Claude won't run this on its own — you invoke it deliberately, which is the right design for a verification step."

**Then the three habits, and demo the first:**

**Do:** `!git diff --stat` on the goal run's branch.

1. **Read the diff, not the conversation.** "The diff tells you what changed. The transcript tells you what Claude *said* about what changed. Different documents — and the first one is shorter."
2. **Check what the hooks recorded.** `!cat logs/skills.log` → which protocols ran. Session transcript in `~/.claude/projects/` for full detail.
3. **Get a second read.** "`/advisor` consults a different model at key moments; `/code-review` gives you a review pass. A fresh model reading the diff has no attachment to the approach that produced it."

**The trick worth giving them:** "Have Claude summarize what it did **from the transcript and the diff**, not from memory, and keep that as a methods note. Grounded in artifacts rather than recollection."

---

## 0:23 — Keep an undo [SLIDE]

**Three bullets:**
- **Branch first.** Not optional for unattended work. A run on its own branch is a run you can delete.
- **`/rewind`** (or `Esc` `Esc`) — restores code, conversation, or both. Covers the case git doesn't: the conversation itself.
- **Commit at checkpoints.** "Turns one unreviewable diff into several readable ones."

---

## 0:24 — The recipe [SLIDE] ⭐

**Put the eleven-step checklist up and read it fast.** Don't elaborate — they have the handout.

**Then the line that closes the module:**

**Say:** "Steps one through seven are guardrails. Step eight — the goal — is the only one that's really letting go, and it's safe in proportion to how well you did the first seven."

**And the close for the whole workshop:**

**Say:** "None of this is specific to Claude Code. It's the discipline you'd apply to a capable new research assistant working unsupervised: agree the plan, bound what they can touch, write the protocol down, keep the raw data read-only, and check the work against artifacts rather than their account of it. The tooling is new. The practice isn't."

---

## Questions to expect

- **"Is auto mode safe?"** — Wrong question. Auto mode is safe in proportion to your deny rules, hooks and sandbox. On its own it's a classifier making judgment calls about things you haven't constrained.
- **"How do I know it didn't do something I didn't notice?"** — The diff and the hook logs. That's exactly why §5 says turn logging on first.
- **"Can I run this on the cluster / overnight?"** — Mechanically yes (`-p`, background, remote control). Do §5 and §7 first, and set a token budget.
- **"What if I don't trust the output?"** — Good. §8. Verify from artifacts, and get a second model on the diff.
- **"How much does an unattended run cost?"** — Quote your real `/usage` delta. Then: calibrate on something cheap before trusting a long one.
- **"Does the sandbox work on my machine?"** — Run `/sandbox`; it varies by platform. `/doctor` warns if managed settings override your config.
- **"Isn't this a lot of setup?"** — Once per project, and most of it is the allowlist, which `/fewer-permission-prompts` writes for you. Compare it to the cost of one unreviewable result.

## If you're behind

1. §9 undo → one line: "branch first, `/rewind` exists"
2. §7 background → `/bg` and `/tasks`, skip remote control
3. §3 allowlist → name the skill, skip the demo
4. §2 plan mode → state the argument without the demo
5. **Never cut §1, §5, §6 or §10.** The principle, the floor, the goal-specification lesson, and the checklist they take home.
