# Skills

## 1. Understand the context ladder

Everything Claude knows about your project arrives through its *context*: a finite budget of tokens that holds the conversation, the files it has read, and the instructions it has been given. Context is the scarce resource in agentic coding, and nearly every feature we'll see in this workshop is a strategy for spending it well. It helps to think of a ladder:

| Mechanism | When it loads | What it costs you |
|---|---|---|
| `CLAUDE.md` | Every single request | Permanent context |
| Memory | When Claude judges a memory relevant | A one-line index entry |
| **Skills** | Body loads when the task calls for it | ~100 words, always; the body only when used |
| Subagents | On delegation, in a *fresh* window | Nothing but a summary |

Module 2 covered the first two rungs and the warning that came with them: `CLAUDE.md` is loaded on every request, so it must be short, and long or rambling files "cost tokens and dilute context". That constraint is real, and it creates an obvious problem. The instructions for correctly ingesting a dataset, or for writing a Marimo notebook that passes the linter, are *not* short. They run to hundreds of lines. They cannot live in `CLAUDE.md`.

Skills are the answer. A skill is a folder containing a Markdown file of instructions whose **body** Claude loads only when the task at hand calls for it. Precisely, skills load in three levels:

1. **Metadata** — the skill's name and description, roughly 100 words. This is *always* in context, for every installed skill, because it's what Claude uses to decide whether the skill is relevant.
2. **The body** — the rest of `SKILL.md`. Loaded whenever the skill triggers.
3. **Bundled resources** — reference files and scripts. Loaded, or executed, only as needed.

Level 1 is the part worth internalizing: a skill is not free. Every skill you install occupies a permanent hundred words whether you ever use it or not, which is why a machine with forty installed skills pays a real tax before you type anything. We'll measure that directly in section 11.

But levels 2 and 3 invert the economics of everything else. Because the *body* costs nothing until invoked, it can afford to be exhaustive. You are no longer writing a terse summary and hoping Claude infers the rest — you are writing the full protocol.

> [!NOTE]
> For researchers, this is the feature that matters most. A skill is a methods section that executes. It is the place where "how we clean this dataset in our lab" stops being tribal knowledge passed between graduate students and becomes a versioned, reviewable, shareable artifact that the agent actually follows.

### When should I create a skill?

Skills are to coding agents as functions are to code. Create a skill when you want a task performed consistently without repeating yourself. Unlike functions, skills do not guarantee exact execution. In compensation, skills do not require precisely shaped inputs. *Skills are not a replacement for code*. 

## 2. List the available skills

Let's see what `data-analyst` ships with. Run the `/skills` command:

```
  Skills

  ❯ 1. analyze    Use when the user asks a question that can be answered with data.
    2. geo        Use for spatial operations: spatial joins, geocoding, boundary…
    3. ingest     Use when the user wants to add a new data source or ingest data.
    4. match      Use when linking records across datasets.
    5. notebook   Open a new or existing Marimo notebook for interactive analysis.
    6. setup      Use at the start of a new project or when the virtual environment…
```

Six skills, and they map directly onto the stages of an empirical research project:

```shell
data-analyst
|- setup       # create the environment
|- ingest      # fetch, clean, and register a data source
|- match       # link records across datasets
|- geo         # spatial joins, crosswalks, and maps
|- analyze     # answer a research question
|- notebook    # interactive exploration in Marimo
```

Notice that this list is itself a claim about how research gets done. Someone decided that "ingesting a source" and "matching two sources" are distinct operations with distinct protocols. That design decision is the interesting part of authoring skills; the file format is trivial by comparison.

## 3. Read a skill

Skills are plain Markdown, so let's just read one. Open `.claude/skills/setup/SKILL.md`:

```markdown
---
name: setup
description: >
  Use at the start of a new project or when the virtual environment is missing.
  Creates a uv virtual environment and installs core data analysis packages.
  Examples: "Set up the project", "Initialize the environment", "Install dependencies".
---

# Setup

Create a virtual environment and install packages for data analysis.

## Workflow

1. **Create virtual environment**
   ...
```

Every skill has exactly two required parts.

The **frontmatter** is the YAML block between the `---` fences. It carries a `name` (which must match the directory name) and a `description`. The description is the only part of the skill that Claude sees before deciding whether to use it, which makes it the most important two sentences in the file. We'll come back to this.

The **body** is everything after the frontmatter. It is ordinary Markdown, and it is loaded into context verbatim when the skill fires. There is no special syntax to learn — if you can write a good README, you can write a good skill. Headings, numbered steps, tables, and fenced code blocks all work, and Claude follows them closely.

> [!TIP]
> Write the body as instructions to a competent new lab member, not as documentation. "Never modify raw files after saving — they are the source of truth" is a good line in a skill. "The raw data directory contains the raw data" is not.

## 4. Invoke a skill

There are two ways to make a skill fire, and the difference matters.

**Ask for it.** In the vast majority of cases you simply describe what you want and Claude matches your request against the available descriptions. Let's set up the project:

```
Set up the project environment.

⏺ Skill(setup)
  ⎿  Successfully loaded skill

⏺ Bash(uv venv && uv pip install polars pyarrow duckdb python-dotenv scipy statsmodels scikit-learn altair matplotlib folium
      geopandas pygris marimo 2>&1 | tail -5 &&…)
  ⎿  Using CPython 3.12.7 interpreter at: /opt/homebrew/Caskroom/miniforge/base/bin/python3
     Creating virtual environment at: .venv
     Activate with: source .venv/bin/activate
     … +5 lines (ctrl+o to expand)
  ⎿  (timeout 10m)
  ⎿  Allowed by auto mode classifier

⏺ Write(.tmp/check_env.py)
  ⎿  Wrote 2 lines to .tmp/check_env.py
      1 import polars, duckdb
      2 print("OK")

⏺ Bash(uv run python .tmp/check_env.py)
  ⎿  OK
  ⎿  Found 4 new diagnostic issues in 1 file (ctrl+o to expand)

⏺ The project environment is set up. A test script that imports polars and duckdb runs without errors.

  - Virtual environment: .venv/, using Python 3.12.7.
  - Packages installed: polars, pyarrow, duckdb, python-dotenv, scipy, statsmodels, scikit-learn, altair, matplotlib, folium,
    geopandas, pygris and marimo.
  - Folders: data/raw/, data/clean/, results/, notebooks/, scripts/, .tmp/ and logs/ now exist.

  Your editor may still show "Import could not be resolved" warnings, because it isn't looking at the new .venv. If they bother you,
  set the editor's Python interpreter to .venv/bin/python and they'll go away.
```

Claude should invoke the `setup` skill, then begin working through its workflow — `uv venv`, the `uv pip install` block, the directory scaffold, and the verification step. You never named the skill. Claude matched "Set up the project environment" against `setup`'s description, which helpfully lists "Set up the project" as an example phrasing.

**Name it.** Every skill is also available as a slash command, which skips the matching step entirely:

```
/notebook turnout
```

Use the slash form when you know exactly which protocol you want, when you want to pass arguments, or when Claude has guessed wrong and you want to override it.

> [!NOTE]
> Watch the output when a skill fires. Claude prints the skill invocation as a tool call, the same way it prints a `Bash` or `Read` call. If you asked for something and the skill you expected *didn't* appear in the transcript, Claude is improvising rather than following your protocol — which is exactly the failure mode skills exist to prevent. In module 6 we'll add a hook that logs every skill invocation to `logs/skills.log` so you get a durable record instead of having to watch for it.

## 5. Run a real pipeline

`setup` built us an empty scaffold. Before we can do anything interesting we need data, and this is the point at which the demo repo's design becomes apparent: clone `data-analyst` and `data/`, `scripts/`, `results/`, and `notebooks/` are all **empty**. They are listed in `.gitignore`. The repository ships the *protocols* and none of the outputs.

That is the correct way to distribute this kind of project, and it's worth saying why. A repo that ships a 40 MB parquet file lets a collaborator read your results. A repo that ships the pipeline that *produces* that parquet file lets them reproduce them. The data is downstream of the code; only the code needs to be versioned.

So let's produce some. Ask Claude for the MIT Election Lab's county-level presidential returns:

```
Ingest the MIT Election Lab county presidential returns, 2000 through 2024.
```

Claude matches this to `ingest` and begins working through its five-step workflow. Expect something close to this:

```bash
❯ Ingest the MIT Election Lab county presidential returns, 2000 through 2024.

⏺ Skill(ingest)

⏺ Read(data/sources.yaml)
  ⎿  File does not exist

⏺ Write(.tmp/fetch_mit_election.py)
⏺ Bash(uv run python .tmp/fetch_mit_election.py)
  ⎿  Downloaded 7.4 MB → data/raw/mit_election_countypres_2000_2024.tab

⏺ Write(scripts/clean_mit_election.py)
⏺ Bash(uv run python scripts/clean_mit_election.py)
  ⎿  Rows: 94,099
     Years: [2000, 2004, 2008, 2012, 2016, 2020, 2024]
     States: 51
     Written to data/clean/mit_election.parquet

⏺ Write(.tmp/load_mit_election.py)
⏺ Bash(uv run python .tmp/load_mit_election.py)
  ⎿  Registered table 'mit_election' (94,099 rows, 12 columns)

⏺ Write(data/sources.yaml)

⏺ Ingested the MIT Election Lab returns: 94,099 county-candidate-year rows
  covering 2000–2024 across 51 state-level jurisdictions, loaded as
  `mit_election` in data/data.duckdb.
```

Four things in that transcript are worth pausing on, because together they are the argument for writing skills at all.

**The raw file is kept.** `data/raw/mit_election_countypres_2000_2024.tab` is the file as it arrived from Harvard Dataverse, and the skill instructs Claude never to modify it: "Never modify raw files after saving — they are the source of truth." Every transformation happens downstream. When a number in your paper is questioned two years from now, this is the file you diff against.

**The cleaning logic became a script, not a conversation.** Claude wrote `scripts/clean_mit_election.py` — a tracked file, in a tracked directory, with a docstring naming the DOI and listing its cleaning decisions. Read it:

```python
"""Clean MIT Election Lab county-level presidential returns.

Source: Harvard Dataverse, doi:10.7910/DVN/VOQCHQ
Raw file: data/raw/mit_election_countypres_2000_2024.tab (tab-separated)

Cleaning steps:
- Parse county_fips as zero-padded 5-digit string
- Drop rows with missing FIPS (write-ins, overseas, etc.)
...
"""
```

Note the third cleaning step. Dropping unmatched FIPS codes is a real analytical decision with real consequences for anything you compute downstream, and it is now written down in a reviewable file rather than buried in a chat transcript that nobody will ever scroll back through.

**The throwaway work went to `.tmp/`.** Fetching and loading are one-off operations, so they landed in `.tmp/` — which `.gitignore` excludes. Cleaning is the step that must be re-runnable and auditable, so it went to `scripts/`. The skill encodes that distinction; Claude would not have invented it.

**The source got registered.** `data/sources.yaml` now contains the provenance record:

```yaml
mit_election:
  origin: https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VOQCHQ
  format: tab
  key: [county_fips, year, candidate]
  cleaning: scripts/clean_mit_election.py
```

That is a machine-readable citation: where it came from, how to join it, and what cleaned it. Downstream skills read this file — `analyze` step 2 checks `sources.yaml` to see what's available before doing anything else — so registering the source is what makes the rest of the project composable.

> [!NOTE]
> Claude will not reproduce this transcript exactly. It may name the scripts differently, fetch via a different route, or split the steps differently. That is expected: a skill constrains the *protocol*, not every keystroke. What should be identical every time is the shape — raw data preserved, cleaning in a tracked script, source registered.

> [!TIP]
> If you're presenting this live, run the ingest before the session and keep the output visible. The Dataverse download takes long enough to kill the room's momentum, and it's the one step here that depends on someone else's uptime.

Now ingest the second source, because the interesting demos need two:

```
Add the Census ACS 5-year county data for 2012 through 2015.
```

Same skill, same five steps, and this time watch for two differences. `sources.yaml` already exists, so Claude reads it before deciding anything (step 1). And because this request spans four years, `ingest` tells Claude to fan out — "spawn general-purpose agents in parallel — one per source or API call" — then clean and load sequentially once the fetches land. One source, one cleaning script, four subsets: the skill is explicit that "Census ACS 5-Year is one source, not one source per year."

> [!NOTE]
> The ACS pull wants a `CENSUS_API_KEY`. Copy `.env.example` to `.env` and add yours if you have one. If you don't, `ingest` handles it: "If a key is not set anywhere, proceed without it when the API allows unauthenticated access." The Census API tolerates modest unauthenticated use, which is enough for this demo. Notice also what the skill forbids — Claude must never `cat` the `.env` file or echo a key, and module 4's deny rules enforce that independently.

With both sources loaded, the rest of the workshop has something to work with: a question to answer here, the hook demo in module 6, and the predictive model in module 7. Try one now:

```
How many counties are in the election dataset per year?
```

## 6. Treat the description as the API

Here is the single most common mistake in authoring skills: writing a brilliant body and a lazy description.

Claude's decision to use a skill is made entirely from the `description` field. The body might contain a perfect, battle-tested, 200-line ingestion protocol — but if the description reads "Data stuff", the skill will never fire, and you will conclude that skills don't work. The description is not a label. It is the trigger condition.

Compare `ingest`:

```yaml
description: >
  Use when the user wants to add a new data source or ingest data.
  Builds a pipeline: fetch raw data → save to data/raw/ → clean → load into DuckDB.
  Examples: "Add census data", "Set up the election results", "Ingest ACS 5-year data".
```

Three things are happening in those four lines:

1. **A trigger condition.** It starts with "Use when…", stating the circumstances in which the skill applies.
2. **A one-line summary of the mechanism**, so Claude can tell this skill apart from its neighbors.
3. **Example phrasings**, in the user's own words, not the author's vocabulary.

That third element does most of the work. A researcher will say "add census data", not "execute the ingestion pipeline". Listing the phrasings you actually expect to type is the cheapest reliability improvement available to you.

There is also a known asymmetry worth exploiting. Claude's current tendency is to **undertrigger** — to not reach for a skill in situations where it would have helped. Anthropic's own guidance for skill authors is therefore to make descriptions slightly *pushy*. Rather than:

```yaml
description: Builds a data ingestion pipeline.
```

write something that states the obligation:

```yaml
description: >
  Builds a data ingestion pipeline. Use this skill whenever the user mentions
  adding, fetching, downloading, or loading data, even if they don't say
  "ingest" and even if they only ask for an analysis that would need new data.
```

The failure mode of an overly pushy description is a skill that fires when you didn't need it, which costs you some context and is immediately obvious. The failure mode of a timid one is a protocol that silently never runs — which is the error you won't notice until a number in your paper is wrong.

> [!TIP]
> If a skill isn't firing when you expect it to, resist the urge to rewrite the body. Rewrite the description, add the exact sentence you just typed to its list of examples, and make the trigger condition more insistent.

## 7. Use progressive disclosure for large skills

The context argument from section 1 has a second act. Skills are cheap *until invoked* — but once invoked, the whole body lands in context. A 700-line skill is still a 700-line bill, just one you only pay when it's relevant.

The `notebook` skill shows the solution. Take a look at its directory:

```shell
.claude/skills/notebook
├── SKILL.md
└── references
    ├── ANYWIDGET.md
    ├── DEPLOYMENT.md
    ├── EXPORTS.md
    ├── PYTEST.md
    ├── SQL.md
    ├── STATE.md
    ├── TOP-LEVEL-IMPORTS.md
    └── UI.md
```

`SKILL.md` holds what's needed for *every* notebook task: the Marimo file format, the cell decorator, the reactivity model, the common mistakes. The eight reference files hold what's needed only sometimes — a total of 700 further lines covering SQL cells, UI widgets, testing, and deployment. `SKILL.md` ends by pointing at them:

```markdown
## Additional resources

- For SQL use in marimo see [SQL.md](references/SQL.md)
- For UI elements in marimo [UI.md](references/UI.md)
- For exporting notebooks (PDF, HTML, markdown, etc.) [EXPORTS.md](references/EXPORTS.md)
```

Those are ordinary relative Markdown links, and that is the whole trick. Claude reads a referenced file when — and only when — the task requires it. Ask for a plain notebook and you pay for `SKILL.md`. Ask for a notebook with a SQL cell and a test suite, and Claude pulls in `SQL.md` and `PYTEST.md` as it goes. This pattern is called **progressive disclosure**, and it's how a skill can be genuinely comprehensive without being ruinously expensive.

A skill directory has three conventional subdirectories, and the names matter because Claude treats them differently:

```
skill-name/
├── SKILL.md          # required: frontmatter + instructions
├── references/       # docs, loaded into context as needed
├── scripts/          # code, executed without being loaded
└── assets/           # templates, icons, fonts used in output
```

Note the distinction between `references/` and `scripts/`. A reference file costs context when Claude reads it. A script costs *nothing* — Claude runs it and sees only its output. That makes `scripts/` the cheapest place to put anything deterministic, which is the subject of the next section.

> [!TIP]
> Anthropic's guidance is to keep `SKILL.md` under about 500 lines. Past that, add a layer of hierarchy: move minority-case material into `references/`, keep the decision-making in `SKILL.md`, and leave clear pointers about when to read what. For reference files over ~300 lines, open them with a table of contents.

## 8. Bundle code with a skill

A skill doesn't have to be only prose. Because it's just a directory, you can ship scripts alongside the instructions and have the skill tell Claude to run them.

This is worth doing deliberately, because there are two quite different ways to get Claude to perform a computation, and they fail differently:

- **Describe the computation in the skill** and let Claude write fresh code each time. Flexible, and wrong in a slightly new way on every run.
- **Ship the script and have the skill invoke it.** The computation is fixed, reviewed, and version controlled. Claude's job shrinks from "implement this correctly" to "call this with the right arguments".

For anything load-bearing — your deflator, your standard-error correction, your crosswalk logic — prefer the second. You want that code reviewed once and reused, not regenerated.

`ingest` applies the idea to cleaning scripts:

```markdown
3. **Clean**
   - If a cleaning script already exists for this source (`scripts/clean_{source}.py`), run it
   - If not, build one:
     ...
   - The cleaning script should handle all subsets/years for this source
```

The first run of a new source generates `scripts/clean_mit_election.py`; every subsequent run reuses it. The skill ratchets toward determinism on its own. Note also that the generated script lands in `scripts/`, under version control, where a collaborator or reviewer can read exactly what was done to the raw data.

> [!NOTE]
> Keep in mind that bundled scripts still have to clear the permission system. `data-analyst` allows `Bash(python scripts/*)` precisely so that the skills in this project can run their own code without prompting on every call. A skill that invokes a command the project doesn't permit will stall waiting for approval — which is a good reason to write the skill and its allow rules at the same time.

## 9. Write your own

Now the part that actually transfers to your work. Pick a task from your own research that you have explained to someone else more than once, and write a skill for it.

Create the directory and the file:

```bash
mkdir -p .claude/skills/describe
```

Then write `.claude/skills/describe/SKILL.md`:

```markdown
---
name: describe
description: >
  Use when the user asks for a summary of a dataset or variable, or for
  descriptive statistics before a formal analysis.
  Examples: "Summarize the election data", "What's in census_acs?",
  "Give me descriptives for vote share".
---

# Describe

Produce a descriptive summary of a dataset or variable.

## Workflow

1. **Locate the data**
   - Check `data/sources.yaml` and `data/clean/` for what's available
   - Query through DuckDB against `data/data.duckdb`

2. **Summarize**
   - Report N, missingness, and unit of analysis first
   - Continuous variables: mean, SD, median, min, max
   - Categorical variables: counts and proportions by level

3. **Report**
   - One table, with the sample size in the caption
   - Flag any variable with more than 5% missingness
```

Then start a new session, ask "summarize the election data", and watch whether your skill fires. If it doesn't, you now know where to look — section 6.

Three things to check as you write:

1. **Does the description state a trigger condition and list real phrasings?** This is where skills fail.
2. **Is the body specific enough to be falsifiable?** "Handle missing data appropriately" is not an instruction. "Flag any variable with more than 5% missingness" is.
3. **Have you encoded a *decision*, not just a sequence?** The best skills in `data-analyst` tell Claude how to choose — `match` lays out four linkage strategies with a "use when" for each, and `analyze` carries a table mapping question types to methods. That judgment is the part Claude can't reconstruct from your directory layout, and it's the part worth writing down.

## 10. Create, evaluate, and improve skills with skill-creator

Having written one by hand, you can now use the shortcut without being misled by it. Anthropic publishes a skill for writing skills, distributed as a plugin on the official marketplace. Install it with `/plugin` (we cover plugins properly in module 7), then invoke it:

```
/skill-creator:skill-creator
```

Describe the task you want captured and it will scaffold the directory, draft the frontmatter, and structure the body — including splitting large material into `references/` when warranted.

> [!NOTE]
> Run `/skills` first and use whatever name your session actually lists. The same Anthropic skill can reach a session by more than one route — as the marketplace plugin above, or pre-bundled under a different namespace such as `anthropic-skills:skill-creator` — and the prefix differs accordingly. This is a good habit generally: `/skills` is the ground truth for what is available to you right now, and the namespace tells you where it came from.

> [!TIP]
> We did this in the opposite order on purpose. `skill-creator` produces a plausible skill very quickly, and if you don't yet know what separates a good description from a bad one, you have no basis for editing what it gives you. Treat its output as a first draft: check the description against your own phrasings, delete the steps that don't reflect how you actually work, and replace vague guidance with your real thresholds.

Generating a first draft is the least interesting thing it does, though. The name undersells it: `skill-creator` is a full iteration loop for skills you already have.

**Modify an existing skill.** Point it at a skill and describe what's wrong. If you have a draft already — including the one you just wrote by hand — it skips the interview and goes straight to the improvement loop.

```
Use skill-creator to improve the describe skill — it's not firing when I ask for summary statistics.
```

**Evaluate a skill.** This is the part that should interest an empirical audience. `skill-creator` will write test prompts, run Claude against them **both with and without the skill**, draft assertions about what a correct response looks like, grade the runs, and build an HTML viewer so you can read the transcripts side by side. That with/without comparison is the only honest way to answer "is this skill actually doing anything?" — and the answer is sometimes no.

**Benchmark it.** Repeated runs with variance analysis, because a single run of a non-deterministic system tells you very little. The bundled scripts give you the shape of the thing:

```
scripts/
├── run_eval.py             # execute a test set
├── run_loop.py             # iterate automatically
├── aggregate_benchmark.py  # pool results, report variance
├── improve_description.py  # optimize triggering
├── generate_report.py
└── package_skill.py        # bundle as a .skill file to share
```

**Optimize the description for triggering.** There is a dedicated routine for just this: it generates realistic queries, runs them, measures whether the skill fires when it should and stays quiet when it shouldn't, and iterates on the wording. Given that the description is where skills fail (section 6), this is the highest-leverage thing in the whole plugin.

> [!NOTE]
> Treating a skill as something you *measure* rather than something you *write* is the right instinct, and it's a familiar one for this audience: a skill is an instrument, and an uncalibrated instrument is a liability. If a protocol is load-bearing for your results — your cleaning rules, your inclusion criteria — run the with/without evaluation at least once. You may find that your careful 200-line skill changes nothing, or that it changes everything, and either way you now know.

## 11. Audit your context with /skill-doctor

Section 1 claimed that every installed skill costs you about a hundred words of permanent context. That raises an obvious question: how much am I actually paying, and for what?

```
/skill-doctor
```

`/skill-doctor` reports which of your loaded skills go **unused** and what each one **costs you in context**, so you can prune them. It is the cleanup tool for the problem that skills create as they succeed.

This matters more than it sounds, for two reasons. The first is cumulative cost: install a dozen plugins over a semester, each shipping two or three skills, and you have thirty-odd always-loaded descriptions competing for attention on every request. The second is subtler and worse — **crowding**. Claude chooses a skill by matching your request against descriptions. The more near-miss descriptions in play, the more opportunities to pick the wrong one. A skill you never use isn't merely inert; it's a distractor.

> [!TIP]
> Run `/skill-doctor` at the end of a project, and when a skill you *do* want starts firing unreliably. Pruning an unrelated skill is a legitimate fix for a triggering problem, and it's the one nobody thinks of.

Combined with the `/context` command from section 1, you now have the full picture: `/context` shows you where your window is going, and `/skill-doctor` tells you which skills are earning their place in it.

## 12. Know where skills live

Finally, the same inside-out resolution we saw for memory and permissions applies here. Skills can come from three places:

- **Project** — `.claude/skills/` in the workspace, shared with everyone who clones the repo. This is where `data-analyst`'s six skills live, and where almost all of your skills should live.
- **User** — `~/.claude/skills/`, available in every session on your machine and invisible to your collaborators.
- **Plugins** — installed bundles, namespaced as `plugin:skill` (hence `/skill-creator:skill-creator`). We'll cover these in module 7.

The distinction between the first two is not merely technical. A skill in the project directory is part of the research artifact: it is cloned with the code, reviewed in pull requests, and cited in the repository that accompanies the paper. A skill in your home directory is a personal convenience that silently stops existing when a collaborator clones your work — and when their results differ from yours, nothing in the repository explains why.

> [!TIP]
> Reach for `~/.claude/skills/` for habits that are genuinely about you rather than the science — how you like commit messages written, say. Put anything that touches data or methods in the project.
