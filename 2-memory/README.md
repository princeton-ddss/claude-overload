# Memory

## 1. Explore project memory

Claude uses *memory* (`/memory`) to save notes about you and your project and recalls them in later sessions. The most basic form of memory is the `CLAUDE.md` files we encountered in the previous module. Let's access these files from Claude using the `/memory` command:

```
  Memory

  ❯ Auto-memory  true

  ❯ Project instructions   Checked in at ./CLAUDE.md
    User instructions      Saved in ~/.claude/CLAUDE.md
    Open auto-memory folder

  Learn more: https://code.claude.com/docs/en/memory

  Enter to confirm · Esc to cancel
```

This menu allows you to view project- and user-specific memory. Select the project instructions and press `Enter` to view this repository's memory (the file might open in a separate window depending on your machine's default Markdown file handler).

`CLAUDE.md` is a *Markdown* file. Markdown is a lightweight markup language that lets you add formatting elements to plain text documents using simple keyboard symbols, such as "#" to denote a header (`h1`). There are no hard requirements for `CLAUDE.md` files—they don't even *have* to be valid Markdown—but there are some recommendations:

1. **Length**. Be concise. The whole file is loaded into context on *every* request. Long or rambling files cost tokens and dilute context.
2. **Clarity**. Be consistent and precise. Vague or contradictory rules get followed inconsistently.

If you ask Claude about it's role, it should give you a summary of `CLAUDE.md` (try it!)

> [!NOTE]
> Claude now accepts `AGENT.md` in lieu of `CLAUDE.md`. An AGENTS.md file is an open-standard version of CLAUDE.md, meaning that instructions written for a coding agent will be picked up by Claude.

## 2. Make a memory

Earlier versions of Claude required users to manage memory exclusively via `CLAUDE.md` files. Anthropic quickly figured out that user-managed memory is annoying. At best, it means telling Claude to "write this down" every time you stumble on another important bit of instructions. In response, Claude Code added the "auto-memory" feature in early 2026. With auto-memory, Claude decides what to remember and stores it in a fixed format.

Auto-memory is turned on by default. You can view and toggle your auto-memory setting with the `/memory` command. Simply select the auto-memory and press `Enter` to toggle on/off.

> [!NOTE]
> Auto memory is restricted to your local session. It is saved to `~/.claude/projects/<project>/memory` so that it is kept separate from version controlled (i.e., shared) code and data.

With auto-memory enabled, Claude auto-magically records critical feedback for later use. Let's see if we can create a memory for our `data-analyst`. For consistency, I'm going to ask Claude to make a memory:

```bash
❯ For this project, remember to refer to me by my code name "Passive Conquistador".

  Wrote 2 memories (ctrl+o to expand)

⏺ Got it, Passive Conquistador. I've saved that to my memory for this project, so future sessions here will use the code name too.
```

We can see what Claude created:

```bash
❯ ls ~/.claude/projects/-Users-me-Desktop-data-analyst/memory
MEMORY.md		user-code-name.md
```

The contents of these files is instructive:

```bash
❯ cat ~/.claude/projects/-Users-cs7101-Desktop-data-analyst/memory/MEMORY.md
- [User code name](user-code-name.md) — address the user as "Passive Conquistador" in this project

❯ cat ~/.claude/projects/-Users-cs7101-Desktop-data-analyst/memory/user-code-name.md
---
name: user-code-name
description: "In this project, address the user by their code name \"Passive Conquistador\""
metadata:
  node_type: memory
  type: user
  originSessionId: 10bcc4e0-b923-4128-8508-afee218609a1
  modified: 2026-10-01T19:59:35.687Z
---

Refer to the user as "Passive Conquistador" in this project.

**Why:** The user explicitly asked to be called by this code name for this project.
**How to apply:** Use "Passive Conquistador" when addressing the user directly (greetings, sign-offs, direct address). Don't overuse it in every sentence.
```

As you can see, auto-memories are also Markdown files, but they follow a precise schema/format. Importantly, they are registered in `MEMORY.md`, which allows Claude to load memories *as needed* instead of polluting context with memories (the point of memories is to extend beyond context).
