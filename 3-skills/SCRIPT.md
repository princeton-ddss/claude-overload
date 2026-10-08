# Module 3: Skills — Presenter Script

**Duration:** 40 min · **Audience handout:** [README.md](README.md)

---

## Pre-flight (do this before the room sits down)

1. **Two terminal tabs, both in a clone of `data-analyst`:**
   - **Tab A — "clean"**: fresh clone, `/setup` already run (venv exists, dirs exist, `data/` empty). This is where you demo.
   - **Tab B — "loaded"**: both sources already ingested. Your escape hatch if a live fetch stalls.
2. **In Tab B, pre-run the ACS ingest.** Four parallel fetches plus a key — too slow and too fragile to do live.
   ```
   Add the Census ACS 5-year county data for 2012 through 2015.
   ```
3. `export CENSUS_API_KEY=...` in both tabs, or copy `.env.example` → `.env`.
4. Confirm the election fetch still works end-to-end (Dataverse has been known to be slow):
   ```bash
   curl -sI "https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/VOQCHQ" | head -1
   ```
5. `/skills` in Tab A — note **which** `skill-creator` prefix your session lists (see §10 below).
6. Run `/skill-doctor` once in Tab A and glance at the output. If your machine has few skills installed the result is undramatic — if so, present it from a tab with more plugins loaded, or just describe what it reports.
7. Have `draft-describe.md` open in an editor, pre-written, ready to paste in §9.
8. Font size up. `/config` → theme that reads on a projector.

> **If you only have 5 minutes to prep:** use Tab B for everything and narrate the transcripts in the README. The module survives without a live fetch; it does not survive without data.

---

## Timing

| Time | Beat | § |
|---|---|---|
| 0:00 | The context ladder | 1 |
| 0:04 | List and read a skill | 2–3 |
| 0:08 | Invoke: `setup` | 4 |
| 0:12 | **Run a real pipeline: `ingest`** | 5 |
| 0:22 | The description is the API | 6 |
| 0:28 | Progressive disclosure + bundling code | 7–8 |
| 0:33 | Write your own | 9 |
| 0:36 | `skill-creator` as an eval loop | 10 |
| 0:38 | `/skill-doctor`, where skills live | 11–12 |

**Hard checkpoint: be at §5 by 0:12.** Everything before it is framing; §5 is the module.

---

## 0:00 — The context ladder [SLIDE]

**Say:** "Context is the scarce resource. Every feature in this workshop is a strategy for spending it."

Draw the ladder. Four rungs, one line each:

- `CLAUDE.md` → every request → permanent cost
- Memory → when relevant → one index line
- **Skills → three levels: description always (~100 words) · body on trigger · resources as needed**
- Subagents → fresh window → you pay for a summary

**Do not skip the three-level point.** It's what makes `/skill-doctor` make sense at 0:38, and it's the honest version — skills are not free. Say: *"Every skill you install costs a hundred words forever, whether you use it or not. The body is what's free."*

**Call back to module 2:** "We told you to keep `CLAUDE.md` short. So where do the 200 lines about how your lab cleans its data go?"

**Land this sentence:** *"A skill is a methods section that executes."*

> Don't explain the file format yet. Motivation first — they'll read the format off the screen in four minutes.

---

## 0:04 — List and read a skill [TERMINAL]

**Do:** `/skills`

**Say:** Six skills, and read the list aloud as a research workflow: setup → ingest → match → geo → analyze → notebook.

**Point out:** "This list is an argument about how research gets done. Someone decided ingesting and matching are different protocols. That design call is the hard part; the file format is trivial."

**Do:** Open `.claude/skills/setup/SKILL.md` (use `@` so there's no tool call: `Show me @.claude/skills/setup/SKILL.md`)

**Say:** Two parts only.
- Frontmatter: `name` + `description`. "The description is the only thing Claude sees before deciding to use this. Most important two sentences in the file."
- Body: plain Markdown, loaded verbatim. "No syntax to learn. If you can write a README, you can write a skill."

---

## 0:08 — Invoke: `setup` [TERMINAL]

**Do (Tab A):**
```
Set up the project environment.
```

**While it runs, say:** "I never named the skill. Claude matched my sentence against the descriptions."

**Then show the slash form:**
```
/notebook
```
…and `Esc` out of it. "Two ways in. Ask when you want Claude to choose; slash when you know which protocol you want, or when Claude guessed wrong."

**Flag forward:** "Watch for the `Skill(...)` line in the transcript. If you asked for a protocol and it isn't there, Claude is improvising. Module 6 makes that permanent with a log."

---

## 0:12 — Run a real pipeline: `ingest` [TERMINAL] ⭐

**Open with the gitignore reveal.**

**Do:** `!ls data/` → empty. Then `!cat .gitignore`

**Say:** "Fresh clone. No data, no scripts, no results — all gitignored. This repo ships protocols and zero outputs. A repo that ships a parquet file lets me read your results; a repo that ships the pipeline lets me reproduce them."

**Do:**
```
Ingest the MIT Election Lab county presidential returns, 2000 through 2024.
```

**Narrate while it runs** — this is 2–3 minutes of dead air otherwise. Four things, in this order, as they appear:

1. **Reads `sources.yaml` first** → doesn't exist yet → proceeds anyway. "Step 1 of the skill. It checks before it acts."
2. **Raw file saved and never touched again.** Quote the skill: *"Never modify raw files after saving — they are the source of truth."* → "This is the file you diff against when a reviewer questions a number in two years."
3. **Cleaning became a script, not a conversation.** `!cat scripts/clean_mit_election.py` → point at the docstring: DOI, raw path, cleaning steps. **Dwell on "Drop rows with missing FIPS."** → "That is an analytical decision with downstream consequences. It's now in a reviewable file instead of buried in a chat log nobody will scroll back through."
4. **`.tmp/` vs `scripts/`.** Fetch and load went to `.tmp/` (gitignored); cleaning went to `scripts/` (tracked). "The skill encodes that distinction. Claude would not have invented it."

**Then:** `!cat data/sources.yaml`

**Say:** "Machine-readable citation: where it came from, how to join it, what cleaned it. `analyze` reads this file before it does anything. That's what makes the project composable."

**Expected numbers:** 94,099 rows · 12 columns · 2000–2024 · 51 jurisdictions.

**Second source — switch to Tab B.** "I pre-ran this one; it fans out four parallel fetches." Show `sources.yaml` with both entries.

**Say:** "Same skill, and note what's different: `sources.yaml` exists now, so step 1 actually reads it. And ACS is *one* source with four subsets — the skill is explicit about that. One cleaning script, not four."

**Close the beat (Tab B):**
```
How many counties are in the election dataset per year?
```

### Failure modes

| Symptom | Recovery |
|---|---|
| Dataverse slow or 503 | Switch to Tab B. "Here's one I ran earlier." Keep moving. |
| Permission prompt mid-fetch | Approve it, then flag: "Module 4 — compound Bash commands need every part allowed." |
| Claude writes a *different* pipeline | **Good.** "A skill constrains the protocol, not the keystrokes. What's identical every time is the shape." |
| ACS 403 / missing key | Expected. Quote the skill's fallback. Don't debug live. |

---

## 0:22 — The description is the API [TERMINAL]

**Say:** "Most common failure in authoring skills: brilliant body, lazy description."

**Do:** Show `ingest`'s description. Dissect the four lines into three parts:
1. Trigger condition — "Use when…"
2. One-line mechanism — so Claude can tell it from its neighbors
3. **Example phrasings in the user's words**

**Say:** "Three does most of the work. A researcher types 'add census data', not 'execute the ingestion pipeline.'"

**Add the asymmetry:** "Claude's current bias is to **under**trigger — to not reach for a skill that would have helped. Anthropic's own author guidance is to make descriptions slightly *pushy*: 'use this whenever the user mentions X, even if they don't say Y.'"

**Say:** "An over-pushy description fires when you didn't need it — costs context, instantly obvious. A timid one means your protocol silently never ran. That's the error you find out about when a number in your paper is wrong."

**The takeaway they should write down:** *"If a skill isn't firing, don't rewrite the body. Rewrite the description and paste in the sentence you just typed."*

---

## 0:28 — Progressive disclosure + bundling [TERMINAL]

**Do:** `!ls .claude/skills/notebook/references/` → 8 files, 700 lines.

**Say:** "Skills are free until invoked — but once invoked you pay for the whole body. `SKILL.md` holds what every notebook task needs. The references hold what only *some* need."

**Do:** Show the `## Additional resources` link list. "Ordinary relative Markdown links. That's the entire trick. Ask for a SQL cell and Claude pulls in `SQL.md`; otherwise it doesn't."

**Heuristic to give them:** "Past a couple hundred lines, move the minority-case material into `references/`. Keep the decisions in `SKILL.md`, push detail outward."

**Then bundling — connect it to what they just watched:**

**Say:** "Two ways to get a computation done. Describe it and let Claude rewrite it each time — flexible, and wrong in a slightly new way every run. Or ship the script and have the skill call it. For your deflator, your SE correction, your crosswalk: ship the script."

**Call back:** "That's what `ingest` did. First run generated `clean_mit_election.py`; every later run reuses it. The skill ratchets toward determinism on its own."

---

## 0:33 — Write your own [TERMINAL]

**This is the part that transfers. Protect the time.**

**Do:** Create the skill live — paste from `draft-describe.md`:
```bash
mkdir -p .claude/skills/describe
```
Paste the `SKILL.md` from README §9.

**Do:** New session, then:
```
summarize the election data
```

**If it fires:** "Note I never said 'describe'."
**If it doesn't:** *perfect* — "Section 6. The description, not the body." Fix it live.

**Then the three checks** (one line each, don't belabor):
1. Does the description state a trigger and list *your* phrasings?
2. Is the body falsifiable? "Handle missing data appropriately" is not an instruction. "Flag variables above 5% missingness" is.
3. **Have you encoded a decision, not just a sequence?** → point at `match`'s four strategies with "use when" for each. "That judgment is the part Claude can't reconstruct from your directory layout."

**Ask the room:** "What's a task you've explained to a grad student more than twice?" Take 2–3 answers. That's the bridge to the clinic session.

---

## 0:36 — skill-creator as an eval loop [TERMINAL]

**Do:** `/skills` → point at whichever `skill-creator` your session lists.

**Say:** "There's a skill for writing skills. I showed it *last* on purpose — it produces a plausible skill in thirty seconds, and if you don't know what separates a good description from a bad one you can't edit what it hands you. Treat it as a first draft."

**Then reframe — this is the beat that lands with researchers:**

**Say:** "Generating a draft is the least interesting thing it does. It will also write test prompts, run Claude **with and without** your skill, grade both, and show you the transcripts side by side."

**Do:** `!ls ~/.claude/plugins/cache/claude-plugins-official/skill-creator/*/skills/skill-creator/scripts/`

**Say, pointing at the filenames:** "`run_eval.py`, `aggregate_benchmark.py` — repeated runs with variance analysis. `improve_description.py` — a dedicated loop that optimizes triggering. These are evals for your protocol."

**Land it:** *"A skill is an instrument. An uncalibrated instrument is a liability. If a protocol is load-bearing for your results, run the with/without comparison once — you may find your careful 200-line skill changes nothing."*

> Don't run an eval live. It spawns background runs and takes minutes. Showing the script names and naming what they do is enough.

---

## 0:38 — /skill-doctor and where skills live [TERMINAL]

**Do:** `/skill-doctor`

**Say:** "Callback to the first slide. Every installed skill costs ~100 words forever. This tells you which ones go unused and what each is costing you."

**The non-obvious point — make it:** "An unused skill isn't just inert, it's a *distractor*. Claude picks a skill by matching against descriptions. More near-miss descriptions, more chances to pick wrong. So pruning an unrelated skill is a legitimate fix for a triggering problem — and it's the one nobody thinks of."

**Close on provenance.** Three sources: project `.claude/skills/`, user `~/.claude/skills/`, plugins.

**Final line:** "A skill in the project is part of the research artifact — cloned, reviewed, cited. A skill in your home directory silently stops existing when a collaborator clones your work. And when their numbers differ from yours, nothing in the repo explains why."

---

## Questions to expect

- **"Does this work with R / Stata?"** — Yes, skills are just instructions; the demo repo happens to prefer Python. Permissions already allow `Rscript`.
- **"How is this different from a prompt I save in a text file?"** — Claude chooses it on its own, it's versioned with the code, and it's shared with collaborators. The description is the difference.
- **"Does Claude always follow it?"** — No. It's a strong prior, not a guarantee. That's precisely why module 6 exists: hooks are deterministic, skills are not.
- **"Won't it get stale?"** — Yes, like any documentation. The difference is it's in the repo, so it breaks visibly in review.
- **"Can I use skills to make it stop doing X?"** — Weakly. Prohibitions belong in deny rules (module 4) or hooks (module 6).

## If you're behind

Cut in this order:
1. §7 progressive disclosure → one sentence
2. §11 `/skill-doctor` → one command, no commentary
3. §10 skill-creator → mention the eval loop exists, skip the `ls`
4. §4 `setup` → skip; start from Tab B with data loaded
5. **Never cut §5 or §9.** The pipeline is the proof; writing one is the transfer.
