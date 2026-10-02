# Permissions

The power of Claude Code comes from its ability to use tools. Tools come in various forms; each gives Claude to read, write, and otherwise control some external system. The most important tool in Claude's arsenal is `Bash`, which allows Claude to execute terminal and shell commands. `Bash` is a powerful—and *dangerous*—tool. Claude therefore provides a permission system that allows users to control access to tools.

To view the permissions for a workspace, use the `/permissions` slash command. 

```bash
  Permissions  Recently denied   Allow   Ask   Deny   Auto mode   Workspace

  Claude Code won't ask before using allowed tools.
  ╭───────────────────────────────────────────────╮
  │ ⌕ Search…                                     │
  ╰───────────────────────────────────────────────╯

    1.  Add a new rule…
    2.  mcp__computer-use__computer_batch
    3.  mcp__computer-use__cursor_position
    4.  mcp__computer-use__double_click
    5.  mcp__computer-use__hold_key
    6.  mcp__computer-use__key
    7.  mcp__computer-use__left_click
    8.  mcp__computer-use__left_click_drag
    9.  mcp__computer-use__left_mouse_down
  ↓ 10. mcp__computer-use__left_mouse_up

  ←/→ to switch · ↓ to select · Esc to cancel
```

The permissions menu shows you *all* permissions for a project. Similar to memories, permissions are scoped to the user and project level. Project-specific permissions are added to `.claude/settings.json`. Here are the permissions for `data-analyst`:

```json
  "permissions": {
    "allow": [
      "Bash(ls *)",
      "Bash(mkdir *)",
      "Bash(python *)",
      "Bash(python3 *)",
      "Bash(.venv/bin/python *)",
      "Bash(Rscript *)",
      "Bash(source .venv/bin/activate && *)",
      "Bash(source renv/activate.R && *)",
      "Bash(duckdb *)",
      "Bash(pip install *)",
      "Bash(pip3 install *)",
      "Bash(uv venv)",
      "Bash(uv venv *)",
      "Bash(uv pip install *)",
      "Bash(uv add *)",
      "Bash(uv run *)",
      "Read(data/**)",
      "Read(results/**)",
      "Read(notebooks/**)",
      "Read(scripts/**)",
      "Read(.tmp/**)",
      "Edit(data/**)",
      "Edit(results/**)",
      "Edit(notebooks/**)",
      "Edit(scripts/**)",
      "Edit(.tmp/**)",
      "Bash(marimo *)",
      "Bash(.venv/bin/marimo *)"
    ]
  },
```
Note that these permissions are part of the repo. The author includes them as a way of saying, "These permissions are needed for this agent to work." This brings us back to the prompt we saw in module one when first running Claude in this workspace: use caution when using third-party agents.

## Test a project permission
`data-analyst` ships with a number of allowed permissions. One of these is `"Bash(mkdir *)"`, which is the shell command to create a new directory. Let's check that it works:

```bash

```

## Add a Deny rule

1. Flip it - add a deny rule with `/permissions`: `Bash(rm *.txt)`
2. Ask Claude to delete `test.txt`: it should refuse.
3. Go back to `/permissions` and delete the rule.

```json
{
"permissions": {
    "deny": [
    "Bash(rm *)"
    ]
}
}
```

## Create a local permission

If you don't like the permissions set by a collaborator, you have two options: either don't use the agent, or customize them using *local* permissions (`settings.local.json`). Local permissions add an additional layer to permissions. Similar to other configuration, settings apply inside-out, so local settings override the agent's shipped permissions.

Local settings can be set by: directly modifying `.claude/settings.local.json`, using the `/permissions` command, or selecting the "Yes, and don't ask again" option in manual mode.

Let's allow our agent to create new Git branches.

- Run `/permissions` slash command
- Under "Allow" tab, select "Add a new rule..." and press `Enter`
- Type in the rule "Bash(git checkout *)` and press `Enter`
- Select "Project settings (local)" and press `Enter`

Confirm that the changes worked by asking Claude to create a new branch.

> [!NOTE]
> Compound Bash commands require every part to be allowed. This means that a command like `mkdir demo && rm -rf test` does *not* match a `Bash(mkdir *)` rule and will only run if both `Bash(mkdir *)` and `Bash(rm *)` are allowed. Claude *loves* chaining commands, which can make it difficult in practice to set permissions in a way that eliminates unnecessary prompts.

### TODO
1. Demonstrate `/permissions` use to allow `WebFetch(domain:api.census.gov)`
2. Show the update to `settings.local.json`
3. Show it works by asking to show the API endpoints for ACS data
4. Demonstrate that "Accept Always" updates `settings.local.json`—just show the updates from accepting web fetches earlier


## Switching modes

Permissions are relevant to the default "Manual mode". At some point, you will probably want to change to another mode. The other modes are:

- **Accept Edits**: allow Reads, file edits, and core filesystem commands *inside the workspace*.
- **Plan**: run read-only exploration to generate a plan without editing files.
- **Auto**: allow any action the classifier approves except `Ask` and `Deny` rules.

### Accept edits

1. Demonstrate `acceptEdits` with `Shift + Tab`
2. Ask Claude to create a new file, `test.txt`

### Plan


### Auto


> [!TIP]
> **Keeping secrets**
>
> Sometimes your agent needs access to credentials in order to perform actions. A common case is API keys. Claude needs to include these in API requests, but you do not want these to enter your conversation history because that history passes through Anthropic's servers. When possible, exporting such variables and telling the name of variable to use in scripts.
