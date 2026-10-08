# Plugins

## 1. Notice that you've been inside one

Here's the layout we explored back in module 1:

```
data-analyst/
├── CLAUDE.md
└── .claude/
    ├── settings.json          # permissions, hooks, enabledPlugins
    └── skills/
        ├── analyze/SKILL.md
        ├── geo/SKILL.md
        ├── ingest/SKILL.md
        ├── match/SKILL.md
        ├── notebook/SKILL.md
        └── setup/SKILL.md
```

And here is the conventional layout of a Claude Code **plugin**:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json            # required: the manifest
├── skills/
│   └── skill-name/SKILL.md
├── hooks/
│   └── hooks.json
├── commands/                  # slash commands (.md files)
├── agents/                    # subagent definitions (.md files)
├── .mcp.json                  # MCP server definitions
└── scripts/
```

Look at those side by side. `data-analyst` has six skills and a hook. A plugin holds skills and hooks. The directories sit at slightly different depths and there's a manifest file we don't have, but **structurally, `data-analyst` is nearly a plugin already**.

That's the whole content of this module. You haven't been accumulating four unrelated features across four modules — you've been building a bundle, and a plugin is what that bundle is called once it's installable.

## 2. Understand what a plugin adds

A plugin is a directory with a manifest and a set of conventionally-named subdirectories that Claude discovers automatically. The manifest lives at `.claude-plugin/plugin.json`, and strictly speaking it needs one field:

```json
{ "name": "my-lab-plugin" }
```

In practice you want a few more:

```json
{
  "name": "my-lab-plugin",
  "version": "1.0.0",
  "description": "Our group's data cleaning and QC protocols",
  "author": { "name": "Your Lab", "email": "you@university.edu" },
  "repository": "https://github.com/your-lab/my-lab-plugin",
  "license": "MIT"
}
```

Two rules worth knowing: the manifest must be inside `.claude-plugin/`, and the component directories must be at the plugin *root*, not nested inside `.claude-plugin/`. Only create directories you actually use.

So what does the manifest buy you? One thing, and it's the thing that matters:

**A project is somewhere you work. A plugin follows you everywhere.**

`data-analyst`'s skills exist when you're in `data-analyst`. The same skills in a plugin are available in every project you open, and installable by anyone in your group with one command. You stop cloning a workspace and start installing a capability.

## 3. Know what transfers and what doesn't

Converting is mostly moving directories, with two important exceptions. This table is the part to understand:

| In `data-analyst` | In a plugin | Transfers? |
|---|---|---|
| `.claude/skills/*/SKILL.md` | `skills/*/SKILL.md` | Yes — move up a level |
| `hooks` block in `settings.json` | `hooks/hooks.json` | Yes |
| *(none yet)* | `commands/*.md` | Add if you have them |
| *(none yet)* | `agents/*.md` | Add if you have them (module 8) |
| *(none yet)* | `.mcp.json` | Add if you have them (module 5) |
| `CLAUDE.md` | — | **No** |
| `permissions` in `settings.json` | — | **No** |

The two exceptions aren't arbitrary, and the reason is worth stating:

**A plugin carries capabilities. A project carries context and trust.**

`CLAUDE.md` describes *this* project — its directory layout, its conventions, the fact that results go in `results/`. That isn't portable, and shouldn't be; it's the project's self-description. Permissions are a trust decision about a specific workspace, and a plugin granting itself blanket filesystem access across all your projects would be exactly the wrong default. (Plugins can pre-approve the tools used by *their own* commands, via `allowed-tools` — a narrow, scoped exception we saw in module 5's Slack plugin, not a general allowlist.)

So when you're deciding where something belongs, ask whether it describes a method or a place. Methods go in the plugin. Places stay in the project.

## 4. Convert `data-analyst`

The exercise, which takes about five minutes:

```bash
mkdir -p my-lab-plugin/.claude-plugin
cd my-lab-plugin
cp -r ~/data-analyst/.claude/skills ./skills
```

Write the manifest to `.claude-plugin/plugin.json`, lift the `hooks` block out of `settings.json` into `hooks/hooks.json`, and you have a plugin. Put it in a Git repository and your group installs it from there.

Now the more interesting question: what *should* be in yours?

- The cleaning and QC protocols your field requires, as skills (module 3)
- A hook that blocks writes to raw data, or to a pre-registration file after submission (module 6)
- Commands for the operations everyone runs — rebuild the derived dataset, regenerate the tables
- An `.mcp.json` entry for your institutional database (module 5)

The result is that a new graduate student installs one plugin and inherits your group's methods — enforced rather than described, versioned rather than remembered. That's a better onboarding story than a wiki page nobody reads.

> [!TIP]
> Don't write one from scratch. Fork `data-analyst`'s skills, replace its conventions with your own, and add the one hook that enforces the rule your group keeps breaking. Starting from something that works is much faster than starting from the format.

## 5. Install someone else's

The other direction is the marketplace:

```
/plugin
```

The official marketplace is `anthropics/claude-plugins-official`, and it lists over three hundred plugins. Browse for a minute and the character of it is clear: Atlassian, AWS, Azure, Airtable, Asana, Box, BigQuery, Canva, Buildkite. Search those three hundred for anything recognizably scientific and you get a handful — `boltz` for protein structure prediction, `nvidia-skills` for GPU work, a few database plugins. A keyword search for "astronomy" returns a company that makes Airflow pipelines.

**The marketplace is built for commercial software teams.** You are not its target market, and you shouldn't expect to find your methods there. That's why section 4 comes first in this module.

Infrastructure is the exception, and it's domain-neutral. You already have two plugins installed, recorded in `data-analyst`'s own settings:

```json
"enabledPlugins": {
  "pyright-lsp@claude-plugins-official": true,
  "ralph-loop@claude-plugins-official": true
}
```

`pyright-lsp` is worth having: it gives Claude real type information about your Python instead of inferences, which helps regardless of what you study. It's also instructive as a plugin — essentially nothing but configuration pointing at a language server, where the Slack plugin from module 5 is the opposite extreme, shipping commands, eight skills, *and* an MCP server. Both are plugins. The format spans that whole range.

Note that `enabledPlugins` sits in the *project* settings file, which means `data-analyst` ships these plugins to anyone who clones it — the same bargain as its permissions and its hook. Plugins install at **user** scope (all your projects) or **project** scope (recorded in the repo, for collaborators); the reasoning is the same as for skills in module 3.

> [!TIP]
> If a newly installed plugin's commands don't appear, run `/reload-plugins`.

## 6. Read what you install

Everything from modules 5 and 6 applies here at once, because a plugin can carry all of it:

- **A plugin can install hooks**, which run automatically on your machine without asking. Read the `hooks/` directory before installing.
- **A plugin can install MCP servers**, with module 5's trust and data-transmission questions.
- **A plugin's commands can pre-approve their own tools** via `allowed-tools`, which is convenient and is a reduction in your oversight.
- **`enabledPlugins` in a cloned repo** comes with the repo.

The reassuring part is that plugins are legible. Unlike an opaque binary dependency, a plugin is Markdown, JSON, and shell scripts in a directory you can read in a few minutes:

```bash
ls -R ~/.claude/plugins/cache/<marketplace>/<plugin>/
```

> [!TIP]
> Prefer first-party plugins and vendors you already trust with the system in question. For anything else, read it first — and if it ships a hook, read that first of all.
