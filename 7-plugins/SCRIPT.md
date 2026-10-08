# Module 7: Plugins — Presenter Script

**Duration:** 10 min · **Audience handout:** [README.md](README.md)

---

## Pre-flight

1. **Two layouts ready to show side by side** — `data-analyst/` and the conventional plugin layout. A slide with both is better than two `ls` calls; the whole module depends on them being visible at once.
2. **`my-lab-plugin/` pre-built** in a scratch directory, from §4's commands. Showing a finished conversion beats typing `cp -r` live.
3. `!cat ~/GitHub/data-analyst/.claude/settings.json` ready, so you can point at `enabledPlugins` in §5.
4. `/plugin` opens cleanly. You're only scrolling it, not installing.

> **This is a 10-minute module and it has one idea.** Resist expanding it. The autonomy material in module 9 is where the remaining time belongs.

---

## Timing

| Time | Beat | § |
|---|---|---|
| 0:00 | **You've been inside one** | 1 |
| 0:02 | What the manifest adds | 2 |
| 0:04 | **What transfers, what doesn't** | 3 |
| 0:06 | Convert it — what goes in yours | 4 |
| 0:08 | The marketplace, and who it's for | 5 |
| 0:09 | Read what you install | 6 |

---

## 0:00 — You've been inside one [SLIDE] ⭐

**Put both layouts up together. Say nothing for a beat and let them look.**

**Say:** "Left is `data-analyst`, from module 1. Right is the conventional layout of a Claude Code plugin. `data-analyst` has six skills and a hook. A plugin holds skills and hooks."

**Land it:** *"Structurally, `data-analyst` is nearly a plugin already."*

**Then the framing for the whole module:** "That's the content of this module. You haven't been collecting four unrelated features across four modules — you've been building a bundle. A plugin is what that bundle is called once it's installable."

---

## 0:02 — What the manifest adds [SLIDE]

**Say:** "A plugin is a directory with a manifest and conventionally-named subdirectories that Claude discovers automatically."

**Show `plugin.json`.** "Strictly, one required field: `name`. In practice you want version, description, author, repository."

**Two rules:** manifest goes *inside* `.claude-plugin/`; component directories go at the plugin **root**, not nested in it.

**Then the payoff sentence — this is what the manifest actually buys:**

**Say:** *"A project is somewhere you work. A plugin follows you everywhere."*

**Say:** "`data-analyst`'s skills exist when you're in `data-analyst`. The same skills in a plugin are available in every project you open, and installable by your whole group with one command. You stop cloning a workspace and start installing a capability."

---

## 0:04 — What transfers, what doesn't [SLIDE] ⭐

**Put up the table.** Read the two "No" rows: `CLAUDE.md` and `permissions`.

**Ask:** "Why wouldn't those transfer?"

**Then answer:** *"A plugin carries capabilities. A project carries context and trust."*

**Say:** "`CLAUDE.md` describes *this* project — its layout, its conventions, the fact that results go in `results/`. That's the project's self-description; it isn't portable and shouldn't be. And permissions are a trust decision about one workspace. A plugin granting itself blanket filesystem access across all your projects would be exactly the wrong default."

**The precision point, if anyone asks:** "Plugins *can* pre-approve the tools their own commands use, via `allowed-tools` — narrow and scoped. Not a general allowlist."

**The rule to leave them with:** *"Ask whether the thing describes a method or a place. Methods go in the plugin. Places stay in the project."*

---

## 0:06 — Convert it [TERMINAL]

**Do:** Show the pre-built `my-lab-plugin/`. Walk the three steps verbally — copy `skills/` up a level, lift the `hooks` block into `hooks/hooks.json`, write the manifest.

**Say:** "Five minutes of work. Put it in a Git repo and your group installs from there."

**Then spend the rest of the beat on the better question — what should be in theirs:**
- The cleaning and QC protocols your field requires → skills
- A hook that blocks writes to raw data, or to a pre-registration file after submission
- Commands for what everyone runs — rebuild the derived dataset, regenerate the tables
- An `.mcp.json` entry for the institutional database

**Close:** *"A new graduate student installs one plugin and inherits your group's methods — enforced rather than described, versioned rather than remembered. Better than a wiki page nobody reads."*

**Practical first step:** "Don't start from the format. Fork `data-analyst`'s skills, replace the conventions with yours, add the one hook that enforces the rule your group keeps breaking."

---

## 0:08 — The marketplace, and who it's for [TERMINAL]

**Do:** `/plugin` → scroll briefly. Let them see the volume, don't narrate it.

**Say:** "Three hundred plus. Atlassian, AWS, Azure, Airtable, Asana, Box, BigQuery, Canva. I grepped all of them for anything recognizably scientific and got about four: `boltz` for protein structure, `nvidia-skills` for CUDA, a couple of database plugins. A search for 'astronomy' returns a company that makes Airflow pipelines."

**Say:** "**The marketplace is built for commercial software teams.** You're not the target market. That's why we did §4 first."

**Then the genuine exception:**

**Do:** Show `enabledPlugins` in `data-analyst/.claude/settings.json`.

**Say:** "You already have two installed. `pyright-lsp` is worth having — real type information about your Python instead of inferences, useful regardless of field. And it's instructive: it's essentially nothing but config pointing at a language server, where module 5's Slack plugin ships commands, eight skills, *and* an MCP server. Both are plugins. The format spans that whole range."

**One note:** "`enabledPlugins` is in the *project* settings file — so this repo ships these plugins to anyone who clones it. Same bargain as its permissions and its hook."

**Mention in passing:** `/reload-plugins` if new commands don't appear.

---

## 0:09 — Read what you install [SLIDE]

**Four bullets, fast:**
1. A plugin can install **hooks** → they run automatically, no prompt. Read `hooks/` first.
2. A plugin can install **MCP servers** → module 5's trust questions.
3. A plugin's commands can **pre-approve their own tools** → less oversight.
4. **`enabledPlugins` in a cloned repo** comes with the repo.

**Close:** "The good news is plugins are *legible*. Markdown, JSON, and shell scripts in a directory you can read in a few minutes. Prefer first-party — and if it ships a hook, read that first."

---

## Questions to expect

- **"So should I distribute my project as a plugin or as a repo to clone?"** — Both, for different things. The data and the project conventions want a repo. The methods want a plugin, so they follow people into their own projects.
- **"Can a plugin ship a `CLAUDE.md`?"** — No, and §3 explains why. Put the portable guidance in skill bodies instead; that's what loads on demand anyway.
- **"How do I host one privately for the department?"** — Plugins install from Git repos, so a private repo is the straightforward version. Check current docs for the marketplace manifest format if you want a browsable catalog.
- **"How do I know a plugin is safe?"** — Read it. §6. It's a few files.
- **"Is there a plugin for <their field>?"** — Probably not. §5. That's the honest answer and it leads back to §4.

## If you're behind

1. §6 trust → one line: "it can install hooks; read them"
2. §5 marketplace → state the "not your market" finding without scrolling
3. §2 manifest fields → show `{ "name": ... }` and move
4. **Never cut §1 or §3.** The side-by-side is the module; the transfers table is the only part that requires thought.
