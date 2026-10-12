# Hello, Claude

## 1. Install the prerequisites

This workshop is hands-on, so there are a few things to install before we start. If you are following along live, do this now — the demos in later modules assume all of it.

**Claude Code.** The CLI itself. Follow the instructions at [code.claude.com/docs](https://code.claude.com/docs/en/setup) for your platform, then confirm it's on your path:

```bash
claude --version
```

You will also need an Anthropic account to sign in with. Run `claude` and follow the login prompt, or use `/login` from inside a session.

**Git.** Used to clone the demo repo in the next section, and to create branches throughout. Most macOS and Linux machines already have it:

```bash
git --version
```

**uv.** An extremely fast Python package and project manager, from Astral. The `data-analyst` demo repo uses `uv` for everything: its `setup` skill builds the virtual environment with `uv venv`, its permissions allow commands like `Bash(uv run python scripts/*)`, and its notebooks declare dependencies inline for `uv` to resolve. Without it, the very first demo fails.

```bash
# macOS and Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# or, with Homebrew
brew install uv
```

```powershell
# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Confirm it worked, and note that you do *not* need to install Python separately — `uv` will fetch an appropriate interpreter when it builds the environment:

```bash
uv --version
```

**jq.** A command-line JSON processor. We don't use it directly for analysis, but the hook we examine in module 6 pipes Claude's tool-call data through `jq` to write `logs/skills.log`, and the same module inspects session transcripts with it.

```bash
# macOS
brew install jq

# Debian/Ubuntu
sudo apt install jq
```

> [!TIP]
> If you're short on time or on a locked-down machine, install Claude Code and `git` and follow along for the rest. Every module can be *watched* without the full toolchain; only the Python demos in modules 3, 6, and 7 need `uv`, and only the hook demo needs `jq`.

## 2. Clone `data-analyst

We'll be demonstrating Claude Code features by working with a demo repo,  `data-analyst`, built for this purpose. Go ahead and clone the repo:

```bash
git clone https://github.com/princeton-ddss/data-analyst
```

## 3. Start Claude Code

Many aspects of Claude Code's behavior are determined (in part) by the directory it is run from. Claude calls these "workspaces". Typically, they are a clone of a version controlled repo. Let's move into the `data-analyst` clone and start Claude:

```bash
cd data-analyst
claude
```

The first time you run `claude` in a workspace you are greeted with a prompt that checks to make sure you want to give Claude access to the directory's contents. In addition, if the project ships with permissions, the message will state the permitted actions. Agree to proceed.

## 4. Configure Claude

If this is your first time using Claude, it's worthwhile familiarizing yourself before diving in. The first thing to know about are **slash commands** (or just commands). Normally, typing into the prompt and pressing enter sends a message to Claude. Let's call this "prompting". Prompting is the vast majority of your interaction with Claude, but sometimes you need to interact with the Claude app itself, not prompt the model. 

For example, you might want to switch to a more advanced model for a particular task. In the CLI, this is accomplished by typing commands prepended with a slash ("/") in the prompt, e.g., `/model`. Typing "/" alone will open a drop-down menu of commands that you can navigate with the up-down arrow keys. If you don't remember a command, just type a guess and the menu will auto-filter candidate commands.

A useful command for first-time users is `/config`. Let's type that in the prompt and press `Enter`. You should see a new Settings menu appear. The menu has four tabs: Status, Config, Usage, and Stats. The Config tab will be selected by default. It allows you to view and manage key Claude settings, such as auto-compaction and thinking mode. Instructions are printed at the bottom of the menu.

### Example: Change the theme
Search "theme" in the search box or scroll using arrow keys to select the Theme setting and press `Enter`. Select a new theme using the arrow keys and press `Enter` again.

The Settings menu is also where you will find information about the application's current status (e.g., version, login method, etc.) and token usage, which are accessed by toggling the menu's tabs. You can also access these tabs directly from the prompt using the `/status`, `usage`, and `stats` commands. When you're done exploring the Settings menu, press `Esc` to exit.

> [!TIP]
> **Useful commands**
> - `/advisor` - Let Claude consult a stronger model at key moments.
> - `/background (`/bg`) - Send this session to the background and free the terminal.
> - `/cd` - Move this session to a different working directory.
> - `/compact` - Free up context by summarizing the conversation so far.
> - `/context` - Visualize current context usage.
> - `/goal` - Set a goal Claude checks before stopping.
> - `/login`, `/logout`- Sign in/out of Anthropic account.
> - `/memory` - Edit CLAUDE.md files and toggle auto-memory.
> - `/model`, `/effort` - Set the model and thinking effort.
> - `/permissions` - Manage tool permissions.
> - `/powerup` - Discover Claude Code features.
> - `/radio` - Listen to Claude FM lo-fi radio.
> - `/remote-control` - Control the current session from your phone or claude.ai/code.
> - `/rename` - Rename the current session.
> - `/resume` (`claude --resume`) - Resume a previous session.
> - `/rewind` - Restore the code and/or conversation to a previous point.
> - `/skills` - List available skills.
> - `/tasks` - View and manage background tasks.
> - `/voice` - Enable voice mode.
> - `/verify` - Verify a code change does what it is supposed to.

## 5. Explore .claude

`data-analyst` is an example repo that is designed as a shareable Claude agent. Essentially, it bundles a collection of code and Claude data designed to help a researcher perform data analysis tasks. Let's take a quick look at its contents:

```bash
├── .claude
│   ├── settings.json
│   └── skills
│       ├── analyze
│       │   └── SKILL.md
│       ├── geo
│       │   └── SKILL.md
│       ├── ingest
│       │   └── SKILL.md
│       ├── match
│       │   └── SKILL.md
│       ├── notebook
│       │   ├── references
│       │   └── SKILL.md
│       └── setup
│           └── SKILL.md
├── .env.example
├── .gitignore
├── CLAUDE.md
└── README.md
```

The Claude data component of the repo has two parts: `CLAUDE.md`, and `.claude`. The `CLAUDE.md` file contains a description of the agent's purpose and generally useful information for the agent, such as directory structure, preferences, and research conventions (e.g., "report up to 5 decimal places"). The `.claude` directory contains project-specific settings, including default permissions, as well as *skills*, which tell the agent how to perform specific tasks.

> [!NOTE]
> Agents are no big deal. Think of them as a formal, standardized way of providing instructions to team members about how to perform a research task. Explain the general goal and motivation; outline global practices to follow; describe specific tasks and create checklists to avoid errors; call out actions that are not permitted; centralize shared code. An agent is simply a repository that contains all of this information is a specific format.

We'll further explore permissions and skills later. Right now, let's compare this repo with your Claude home directory, `~/.cladue`:

```bash
├── .credentials.json
├── api-key
├── CLAUDE.md
├── commands
├── history.jsonl
├── jobs
├── plans
├── plugins
├── projects
├── scripts
├── sessions
├── settings.json
├── skills
└── weekly-standup-repos.conf
```

(I've excluded a number of files for clarity—your data may look slightly different). The main to notice is that your home directory also includes `CLAUDE.md` and `skills`. These files define behavior and skills that apply across all Claude sessions. This is where you define conventions and permissions you would like Claude to follow by default, and skills that apply globally.

> [!TIP]
> Global configuration can add bloat to Claude's context and is easy to forget about. Use this sparingly.

Claude applies configuration "inside-out", meaning that project-specific settings and skills take precedence over their global counterparts wherever conflict occurs. In the case of `CLAUDE.md`, project- and user-level prompts concatenate and therefore might deliver conflicting instructions.

## 6. Prompt Claude

Now that we have a basic understanding of the layout, it's time to actually interact with Claude. Let's start by asking Claude to create a new Git branch for us:

```
Create a new experimental branch.
```

Claude has built-in mechanisms for keeping track of changes to files, but these are not to be trusted, and Claude does not automatically create a new branch (but sometimes will). It's a good idea to explicitly create a new branch when experimenting or starting a new feature: things can get messy *fast*.

The most important thing here is that you note the output. If you are in the default "Manual mode", Claude should prompt to run `git checkout -b experimental`. Pay attention to what Claude is doing and take time to learn how to read its output even if you don't fully understand the exact command it is running. Notice that the output includes a summary of the command for "human consumption".

To proceed, we must pick one of four options:

1. Yes
2. Yes, and don't ask again for git checkout *
3. Yes, and switch to auto-mode for this session
4. No

Option 1 is a one-time approval; option 2 approves any command of the form `git checkout *` for this workspace (settings.json.local); option 3 approves this command and anything else the Anthropic determines is "safe" for the remainder of the current session, but doesn't change project settings; and option 4 is just "No".

Let's select "Yes" for now. Claude should now actually run the command (note the gray dot is now green), report the shell output, and then respond with a summary of what it did. The flow is: `prompt -> tool use [optional, repeatable] -> response`.

> [!TIP]
> **! and @**
> 
> You can run your own commands by entering "!" in an empty prompt and typing the command. For example, check that we switched branches by typing "!" followed by `git status`. You can also inject the contents of a file in your project using "@". For example, "Review @.gitignore". This avoids Claude guessing which file you are referring to as well as an unnecessary tool call to load its contents.
