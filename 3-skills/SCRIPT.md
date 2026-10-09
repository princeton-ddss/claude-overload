# Skills — Presenter Script

**Budget:** 35 min (of a 60-min block: skills 35 · hooks 12 · MCP 13) · **Handout:** [README.md](README.md)

---

## Pre-flight

1. **Two tabs, both in a clone of `data-analyst`:**
   - **Tab A (clean):** fresh clone, `data/` empty. You demo `setup` and `ingest` here.
   - **Tab B (loaded):** election **and** ACS already ingested. This is the escape hatch, and where you write the `describe` skill. The hooks and MCP demos reuse Tab B, so it needs a populated `logs/skills.log`.
2. **Pre-run the ACS ingest in Tab B** (parallel fetches and an API key: too slow and fragile to do live).
3. `CENSUS_API_KEY` exported in both tabs, or `.env` copied from `.env.example`.
4. **Run `/skills` and `/skill-doctor` once** so you know which `skill-creator` prefix your session lists and what the output looks like.
5. `draft-describe.md` open in an editor (the `SKILL.md` from README §4), ready to paste.
6. Font up. Theme that reads on a projector.

> **5 minutes to prep?** Use Tab B for everything and narrate the README transcripts. The talk survives without a live fetch; it doesn't survive without data.

---

## Timing

| Min | Beat | README |
|---|---|---|
| 0–3 | The context table | intro |
| 3–7 | List and read a skill | §1–2 |
| 7–10 | Invoke: ask vs. name (`setup`) | §3 |
| 10–19 | **`ingest`: a real pipeline** | §3 Fetching Data |
| 19–28 | **Write a skill: `describe`** | §4 |
| 28–33 | Tips: descriptions are triggers · everything else in one breath | Tips, skill-creator |
| 33–35 | Where skills live; close | skill-doctor, locations |

**Checkpoints:** `ingest` started by 0:10, `describe` pasted by 0:19. If either slips, cut from the Tips block, never from these two.

---

## 0:00 — The context table [SLIDE]

**Say:** "Context is the scarce resource. Every feature in this workshop is a strategy for spending it."

Put up the load-strategy table. Say one line per row:
- `CLAUDE.md` → loaded every request. Short, or it crowds everything else.
- Memory and **skills** → just-in-time. An index entry always, the body only when needed.
- Subagents → just-in-time, in a fresh window; you get a summary back.

**Ask (answer it yourself):** "So where do the 200 lines on how your lab cleans its data go? Not in `CLAUDE.md`."

**Three levels, fast:** description always loaded (~100 words) · body when triggered · resources when needed. **"Every skill you install costs a small amount forever; the body is what's free."**

**Land it:** *"A skill is a methods section that executes."*

---

## 0:03 — List and read a skill [TERMINAL]

**Do:** `/skills`

**Say:** "Six `project` skills; the rest are built-ins, plugins, and mine. Notice the token counts: 50–340 each, always loaded." Read the six as a research workflow: setup → ingest → match → geo → analyze → notebook.

**One-line plug:** "`/skill-doctor` shows which of these you never use and what they cost. We'll come back to it."

**Do:** `Show me @.claude/skills/setup/SKILL.md` (the `@` avoids a tool call).

**Say:** Two parts. **Frontmatter**: `name` and `description`; "the description is the only thing Claude sees before deciding to use the skill." **Body**: plain Markdown. "If you can write a README, you can write a skill."

---

## 0:07 — Invoke: ask vs. name [TERMINAL]

**Do (Tab A):** `Set up the project environment.`

**While the install runs, say:** "I never said 'setup'. Claude matched my sentence against the descriptions. Find the `Skill(setup)` line in the transcript. If you asked for a protocol and that line isn't there, Claude is improvising."

> If the install is still going at 1:30, switch to Tab B. Don't wait on `uv`.

**Then the second way:** `/notebook` → `Esc`. "Slash form skips the matching. Use it when you know the protocol or when Claude guessed wrong."

---

## 0:10 — `ingest`: a real pipeline [TERMINAL] ⭐

**Do:** `!ls data/` (empty), then say: "Fresh clone, no data. This repo ships **protocols, not outputs**. A repo that ships a parquet file lets me read your results; one that ships the pipeline lets me reproduce them."

**Do:**
```
Ingest the MIT Election Lab county presidential returns, 2000 through 2024.
```

**Narrate the wait (Dataverse is slow). Point at four things as they appear:**
1. **Raw file kept.** `data/raw/countypres_2000-2024.csv`. Quote the skill: *"Never modify raw files after saving — they are the source of truth."*
2. **Cleaning became a script.** `scripts/fetch_countypres.py`, `scripts/clean_countypres.py`; show the docstring with the DOI and cleaning decisions. "That's an analytical decision, now in a reviewable file instead of a chat log nobody will scroll back through."
3. **Throwaway work went to `.tmp/`** (gitignored). The skill encodes what's reproducible and what isn't.
4. **Source registered.** `!cat data/sources.yaml` → "a machine-readable citation. Other skills read this before they do anything."

**Expected result:** two tables in `data/data.duckdb`: `countypres_county` (21,783 rows) and `countypres` (76,653 rows); 2024 national share ≈ 48.3% D / 49.8% R.

**Hard stop at 0:19.** If it hasn't finished by ~0:16, switch to Tab B: "here's one I ran earlier," show `sources.yaml` with both sources, and move on.

| Symptom | Recovery |
|---|---|
| Dataverse slow / 503 | Tab B. "Here's one I ran earlier." |
| Permission prompt mid-run | Approve; "compound Bash commands need every part allowed" (module 4). |
| Claude builds a *different* pipeline | **Good.** "A skill constrains the protocol, not the keystrokes. The shape is identical: raw kept, cleaning in a script, source registered." |
| ACS 403 / no key | Expected; quote the skill's fallback; don't debug live. |

---

## 0:19 — Write a skill: `describe` [TERMINAL] ⭐

**This is the part that transfers. Protect it.**

**Do (Tab B):**
```bash
mkdir -p .claude/skills/describe
```
Paste the `SKILL.md` from `draft-describe.md`. Read the description aloud, then the three workflow steps.

**Do:** start a fresh `claude` session in Tab B, then:
```
Summarize the election data.
```

**If it fires:** "I never said 'describe'." Open the transcript to the `Skill(describe)` line.
**If it doesn't:** *good*: "That's the most common failure in skill authoring, and it's almost always the description." Fix it live by adding the sentence you just typed to the examples; retry.

**Then, for 90 seconds, the three checks:**
1. Trigger condition and *your* phrasings in the description?
2. A falsifiable body? "Handle missing data appropriately" isn't an instruction; "flag variables above 5% missingness" is.
3. A *decision*, not just a sequence? Point at `match`'s four strategies, each with a "use when."

**Ask the room (take 2–3 answers):** "What's a task you've explained to a grad student more than twice?"

**Close with the framing:** "Skills are to coding agents as functions are to code, except they don't guarantee exact execution. They're not a replacement for code."

---

## 0:28 — Tips [SLIDE + TERMINAL]

### Tip 1: descriptions are triggers, not labels (3 min)

**Do:** show `ingest`'s description. "Three parts: **trigger condition** ('Use when…'), **one-line mechanism**, **example phrasings in the user's own words**. A researcher types 'add census data,' not 'execute the ingestion pipeline.'"

**The asymmetry:** "Claude under-triggers. Anthropic's own advice is to make descriptions slightly **pushy**: 'use this whenever the user mentions X, even if they don't say Y.' Over-triggering is cheap and obvious. A timid description means your protocol **silently never ran**, an error you find when a number in your paper is wrong."

**Write-this-down line:** *"If a skill isn't firing, rewrite the description, not the body."*

### Tips 2–3 and skill-creator: one breath each (2 min)

- **Progressive disclosure.** `!ls .claude/skills/notebook/references/` → "eight reference files, 700 lines, read only when needed. `SKILL.md` holds what every task needs; links hold the rest."
- **Bundle code.** "For anything load-bearing (your deflator, your crosswalk), ship the script and have the skill call it. Reviewed once, reused, not regenerated each run. `ingest` did exactly this with the cleaning script."
- **`/skill-creator`.** "Generates a first draft in 30 seconds, which is why I showed it *last*. It can also evaluate a skill with and without the skill and benchmark it. A skill is an instrument; an uncalibrated instrument is a liability." *Don't run an eval live.*

---

## 0:33 — Where skills live; close [SLIDE]

**Do:** `/skill-doctor`: "Unused skills aren't inert, they're distractors. Claude picks by matching descriptions, so more near-misses means more wrong picks."

**Three locations, one line:** project `.claude/skills/` · user `~/.claude/skills/` · plugins.

**Final line:** *"A project skill is part of the research artifact: cloned, reviewed, cited. A home-directory skill silently stops existing when a collaborator clones your work, and when their numbers differ from yours, nothing in the repo explains why."*

---

## Questions to expect

- **"R / Stata?"** Skills are just instructions; permissions already allow `Rscript`.
- **"Versus a saved prompt?"** Claude chooses it itself, it's versioned with the code, and your collaborators get it.
- **"Does Claude always follow it?"** No, it's a strong prior, not a guarantee. That's the next segment (hooks).
- **"Stop it doing X?"** Weakly. Prohibitions belong in deny rules or hooks.

## If you're behind

1. Tips 2–3 and skill-creator → drop to one sentence, or cut.
2. `setup` → skip; start in Tab B.
3. `/skill-doctor` → mention only.
4. **Never cut `ingest` or `describe`.** The pipeline is the proof; writing one is the transfer.
