# Tools and MCP — Presenter Script

## Setup

1. **Slack plugin installed and authorized** before the session.
2. **`#claude-overload` channel exists**. Ask folks to join ahead of time. Slack open to channel.

## 1. Why MCP?

-  Prompt 💬 "What tools do you have available?"
  - `Bash` is the universal adapter (`git`, `duckdb`, `gdal`, `Rscript`, etc.)
  - **What do you do when there is no CLI?**
    - MCP is open standard: (1) connect to server, (2) it lists tools, (3) Claude calls them like built-ins.
    - **stdio** vs **HTTP** 

## 2. Install a server

- Run `/plugin`
  - Show Slack - MCP ships *with* the Slack plugin

- Run `/mcp`
  - Installing MCP registers a *config*. Authentication (OAuth) grants access.
    - `!cat ~/.claude/plugins/cache/claude-plugins-official/slack/*/.mcp.json`.
  - View tools
    - Look at `Slacked`
    - Look at `Read channel messages` 

## 3. Use a server — read, then write ⭐

- Prompt 💬 "What are the recent messages in #claude-overload?"
- Prompt 💬 "Post a message to #claude-overload asking how the presentation is going."

## 4. Permission a server

- Run `/permissions`
  - Create new permissions
    
```
{
  "permissions": {
    "allow": [
      "mcp__plugin_slack_slack__slack_search_public"
    ],
    "ask": [
      "mcp__plugin_slack_slack__slack_send_message"
    ]
  }
}
```

- **Note**: name just the server to cover all its tools, e.g. `mcp__plugin_slack_slack` or `mcp__plugin_slack_slack__*`."
- **Auto-mode warning** With write-capable servers connected, put the writes behind explicit **Ask** or **Deny** rules yourself."

## 5. Do you need a server? ⭐

- For most research computing, no.
  - **Use a  CLI**. `gh`, `gcloud`, `duckdb`, `rclone`. Allow it in permissions.
  - **Use API token**. Run a script with API key from the environment, no server involved.
  - **Use tmp**. For one-offs, `.tmp/fetch_whatever.py` works fine.

- **When a server makes sense:**
  - Delegated **OAuth**, where each user authorizes their own account and tokens refresh.
  - A large API someone else already maintains a good server for.

## 6. Warnings

- **The server.** A stdio servers execute with your privileges; Claude acts as you on a remote one. Prefer first-party.
- **The response.** Instruction spoofing ("A well-crafted Slack message is hard to tell from an instruction.")
- **Governance**. IRB protocols, restricted-use licenses, DUAs constrain where data may be transmitted.

## Questions to expect

- **"Is there a server for Zotero / arXiv / PubMed / our cluster?"** Community servers exist; quality varies and nothing is vetted. Check the repo, and ask whether a script wouldn't be simpler.
- **"Doesn't a big server eat my context?"** Mostly no. Tool search announces names and loads schemas on demand. Names still accumulate (see the README's Bonus), and crowding is the remaining cost.
- **"Can I share servers with my lab?"** A `.mcp.json` at the project root. Same bargain as shipped permissions and hooks: read it before you run it. (README Bonus.)
