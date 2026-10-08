# Working Autonomously

## 1. Recognize the two failure modes

By now you can configure a great deal. This module is about the question that actually determines whether any of it gets used: **how do you let Claude work without either babysitting every action or losing track of what it did?**

There are two ways this goes wrong, and most people oscillate between them.

**Prompt fatigue.** You run in manual mode, approve forty prompts an hour, and start approving them without reading. The permission system is still technically in force, but it has stopped functioning — you have become a rubber stamp, and the thing it was protecting you from will get approved along with everything else.

**Blind trust.** You flip to auto mode, stop watching, and discover three days later that a cleaning script changed and you can't tell which results came from which version. Nothing dramatic happened; you just lost the thread.

The way out is not to find a point between them. It's this:

> **Autonomy is bought with guardrails, not with trust.**

Every prompt you stop answering should be replaced by something structural — an allow rule, a deny rule, a sandbox boundary, a hook, a frozen data split, a throwaway branch. The goal is a session where you are prompted rarely *because* the things that would need a prompt are now impossible, not because you turned the prompts off.

The rest of this module is the practical version of that sentence, roughly in the order you should apply it.

## 2. Start in plan mode

The single highest-value habit, and the cheapest.

Press `Shift+Tab` to cycle permission modes, or use `/permissions`, and select **Plan**. Claude explores read-only — reading files, searching, running read-only commands — and produces a plan without changing anything. You read the plan, correct it, and *then* let it work.

```
Plan how to add a county-level fixed effects specification to the turnout analysis.
```

Why this matters more than it sounds: most bad autonomous runs are not caused by a bad tool call. They're caused by Claude confidently solving a slightly different problem than the one you had. Plan mode surfaces that misunderstanding while it's still free to fix.

> [!TIP]
> Plan mode is also the right default for exploring an unfamiliar repository — including `data-analyst`. You get Claude's read of the codebase with a guarantee that nothing changed while it looked.

> [!NOTE]
> Plan mode and auto mode compose. In plan mode, Bash commands that can't be statically proven read-only are judged by the auto-mode classifier rather than prompting you, so exploration stays fluid without becoming write-capable.

## 3. Stop answering the same prompt twice

If a prompt has appeared three times and you've approved it three times, that's not a safety check. It's a tax. Convert it into a rule.

Module 4 covered the mechanics — `/permissions`, or "Yes, and don't ask again" at the prompt. The part worth adding here is that you don't have to do this by hand:

```
/fewer-permission-prompts
```

This scans your transcripts for the read-only Bash and MCP calls you keep approving, then proposes a prioritized allowlist for the project's `.claude/settings.json`. It's the fastest way to go from constant interruption to a working allowlist, and it's grounded in what you actually did rather than what you imagine you'll need.

Two things to hold onto while you do it:

- **Allow generously on reads, stingily on writes.** `Read(data/**)` costs you nothing. `Edit(data/raw/**)` is a decision.
- **Remember compound commands.** Module 4's warning applies: `mkdir demo && rm -rf test` needs *both* parts allowed. Claude chains commands constantly, which is why a hand-written allowlist always has holes a transcript-based one catches.

## 4. Use the sandbox

This is the mechanism that makes real autonomy defensible, and it's the one most people miss.

```
/sandbox
```

Sandboxing constrains what a command *can do* rather than asking whether it should. Two layers matter:

- **Filesystem isolation** — sandboxed commands see a restricted view, with `denyRead` regions and `allowRead` exceptions you control.
- **Network egress control** — an allowlist of domains a sandboxed command may reach, with `deniedDomains` to carve exceptions out of a broad wildcard, and a strict mode that refuses non-allowlisted hosts without prompting.

There's also a `sandbox.credentials` setting that blocks sandboxed commands from reading credential files and secret environment variables outright — the enforced version of module 4's advice about keeping API keys out of context.

Here's the part that connects it to this module's thesis. Because a sandboxed command is constrained, **it can run unprompted**. Sandbox auto-allow means interpreter invocations and the like proceed without a permission dialog, because the sandbox is doing the work the dialog was doing. That's the trade you want: fewer prompts, more enforcement.

> [!NOTE]
> Check `/sandbox` to see what's actually active on your machine before relying on this — availability and defaults vary by platform, and filesystem isolation can be disabled independently of network control. The `/status` and `/doctor` commands will also warn you if managed settings are overriding your sandbox configuration.

> [!TIP]
> Network egress control is the feature to raise with your institution. Module 5 made the point that a remote MCP server is a transmission to a third party; a domain allowlist is the technical answer to "can you guarantee this data doesn't leave?" It's a far better conversation to have with a domain allowlist in hand than without one.

## 5. Set the floor that must hold

Allowlists and sandboxes reduce prompting. They don't express invariants. For the small number of things that must never happen, use the two mechanisms that can't be argued with.

**Deny rules** (module 4) refuse a tool call outright, regardless of mode. Auto mode respects them; the classifier never overrides them. This is where "never touch the pre-registration file" lives.

**Hooks** (module 6) are the general case, and `PreToolUse` is the one with teeth. Recall the guard we built:

```sh
#!/bin/sh
jq -e -r '.tool_input.file_path // "" | select(test("data/raw/"))' >/dev/null 2>&1 \
  && { echo "data/raw/ is immutable. Clean into data/clean/ instead." >&2; exit 2; }
exit 0
```

Four lines, and it holds whether you're watching or not. That's what makes unattended work reasonable: not that Claude is trustworthy, but that the damaging actions are unavailable.

A useful exercise before any long run: **name the three worst things that could happen, and write a deny rule or a hook for each.** If you can't, you aren't ready to leave it unattended — and you've learned something cheaply.

> [!TIP]
> Also install the logging hook from module 6 before a long run, not after. The audit trail is only useful if it was recording while you weren't looking.

## 6. Give it a goal — and specify a procedure

With guardrails in place, you can hand over a multi-turn objective.

```
/goal every specification in scripts/ has been fit and its coefficients written to results/
```

`/goal` sets a completion condition and Claude keeps working across turns until it's met, in interactive sessions, in `-p`, and through Remote Control. It shows live elapsed time, turns, and tokens in an overlay panel, so the cost is visible while it runs. It's hook-driven internally — which means it won't work if hooks are disabled by settings, and it will tell you so rather than hanging.

It's also robust in ways a hand-rolled loop isn't: it retries with backoff through API errors and network drops, pauses and says why if you hit a usage limit, survives compaction, restores when you `--resume`, and clears itself if a turn dies unrecoverably. If background work keeps a goal waiting, it checks in on it rather than blocking forever.

Now the part that matters more than the mechanics. **How you phrase the completion condition determines whether the result means anything.**

Consider a goal we might plausibly set for `data-analyst`:

> "hold-out R² above 75%"

Two things are wrong with it, and only one is obvious.

**The obvious one is smaller than people claim.** Letting an agent try several specifications and keeping the one that clears a bar does bias the reported number upward — but modestly. With roughly 3,155 counties per election year, a 20% test split is about 620 observations, and simulating out-of-sample R² around a true 0.75 across five correlated specifications gives:

| Test set | SD of one draw | Max-of-5 inflation |
|---|---|---|
| 620 counties | 0.017 | +0.019 |
| 3,100 counties | 0.008 | +0.009 |

One to two points of R², growing slowly in the number of attempts. Worth knowing; not a crisis.

**The real problem is that the agent controls the analysis.** Raising a hold-out R² doesn't require trying more regressions. It can be done by fitting the scaler or imputer before splitting, re-drawing the split with a new seed, dropping "outlier" counties, redefining the outcome from total share to two-party share, or building a predictor that encodes the outcome. None of these is a multiplicity problem. Each voids the hold-out guarantee entirely, and each is a locally reasonable-looking step that gets summarized in one approving sentence. There's no bound on the resulting error, because the estimate has stopped estimating anything.

And there's a structural issue on top of it: a results threshold is **unverifiable by the harness**. Nothing can check "R² is above 75%" on your behalf, so the completion condition is self-reported by the agent that wants to finish.

So state the procedure, not the result:

| Specifies a target | Specifies a procedure |
|---|---|
| "hold-out R² above 75%" | "fit the three specifications in `scripts/`; report R², RMSE and N for each against the frozen split in `data/clean/split.parquet`" |
| Agent controls the split and preprocessing | Split and preprocessing fixed before the run |
| Selects on the outcome | Selects on completeness |
| Harness can't verify it | Harness *can* — the files either exist or they don't |

Note the last row, because it closes the loop with section 5: a procedural completion condition is one you can **write a hook for**. A results threshold never is. The guardrail and the goal have to be stated in the same vocabulary.

> [!NOTE]
> The general form: when an agent iterates unsupervised toward a metric, the risk isn't the multiple-comparisons arithmetic — that's bounded and estimable. It's that the agent is choosing the analysis *and* reporting on it, with the degrees of freedom to move the thing you're measuring. The relevant literature is the garden of forking paths; the agent's contribution isn't a bigger garden, it's a much faster walk through it.

## 7. Let it run, and check in deliberately

Hand the session off and get your terminal back:

```
/background      (or /bg)
/tasks
```

`/tasks` lists background work and lets you stop any of it. `/remote-control` lets you follow a session from your phone or from claude.ai, which is genuinely useful for a run you started before leaving the office.

The habit to build is **checking in on a schedule rather than watching continuously**. Watching continuously is how you end up rubber-stamping; checking in at intervals is how you stay able to evaluate.

> [!TIP]
> If you want periodic unattended work rather than one long task, `/loop` runs a prompt or slash command on an interval, or self-paced if you omit the interval. It's a different tool from `/goal`: `/goal` persists until a condition is met, `/loop` recurs on a schedule. `/usage` has a per-loop breakdown — run count, tokens per run — specifically so a chatty loop is easy to spot.

## 8. Verify before you believe it

An autonomous run ends with a confident summary. The summary is not evidence.

```
/verify
```

`/verify` checks that a change does what it was supposed to do. Note that Claude will not run it on its own — you invoke it deliberately, which is the right design for a verification step.

Beyond that, three habits worth more than re-reading the transcript:

- **Read the diff, not the conversation.** `git diff` across the whole run tells you what changed. The transcript tells you what Claude said about what changed. These are different documents, and the first is shorter.
- **Check the artifacts the hooks recorded.** `logs/skills.log` tells you which protocols ran. The session transcript in `~/.claude/projects/` has every tool call if you need to reconstruct something.
- **Ask for a second read.** `/advisor` lets Claude consult a different model at key moments, and `/code-review` gives you a review pass over the diff. A fresh model reading the diff has no attachment to the approach that produced it.

> [!TIP]
> Have Claude summarize what it did *from the transcript and the diff*, not from memory, and keep that as a methods note. It's a better record than anything you'd reconstruct a week later, and it's grounded in artifacts rather than recollection.

## 9. Keep an undo

Autonomy is much easier to grant when mistakes are cheap.

- **Branch first.** Module 1 said create a branch before experimenting. For unattended work it isn't optional. A run on its own branch is a run you can delete.
- **`/rewind`** (or `Esc` `Esc`) restores the code, the conversation, or both to an earlier point. It's the fastest way out of a session that went sideways, and it covers the case git doesn't — the conversation itself.
- **Commit at checkpoints.** A commit between phases of a long run turns one unreviewable diff into several readable ones.

## 10. A recipe

Putting it together, as a checklist for a real unattended run:

1. **Branch.** `git checkout -b experiment-xyz`
2. **Plan first.** Shift+Tab to Plan mode; get the approach right before anything executes.
3. **Name the three worst outcomes** and write a deny rule or `PreToolUse` hook for each.
4. **Turn on logging** — the module 6 skills hook, so the audit trail exists before you look away.
5. **Freeze what must not move.** The data split, the preprocessing, the raw files.
6. **Tune the allowlist.** `/fewer-permission-prompts`, so you aren't interrupted by things you've already decided.
7. **Check the sandbox.** `/sandbox` — filesystem and network boundaries appropriate to the data.
8. **State a procedural goal.** `/goal` with a completion condition the harness could check.
9. **Background it** and check in on a schedule.
10. **Verify from artifacts.** The diff, the logs, `/verify`, a second model on the diff.
11. **Report the cost.** `/usage`, so the next run is easier to budget.

Steps 1 through 7 are the guardrails. Step 8 is the only one that's really "letting go," and it's safe in proportion to how well you did the first seven.

> [!NOTE]
> Nothing here is specific to Claude Code. It's the same discipline you'd apply to a capable new research assistant working unsupervised: agree the plan, bound what they can touch, write down the protocol, keep the raw data read-only, check the work against artifacts rather than their account of it. The tooling is new. The practice isn't.
