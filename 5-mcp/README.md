# Tools and Model Context Protocol (MCP)

Claude interacts with the "outside world" through *tools*: `Bash` to run commands, `Read` and `Write` for files, `Glob` and `Grep` to search, `WebFetch` and `WebSearch` to pull from the internet, `Skill` to invoke skills.

Let's take stock of our available tools before adding anything. Ask Claude:

```
What tools do you have available?
```

The built-in set is small and general. Anything with a command-line interface is already available to Claude without configuration, integration, or a protocol. This is a formidable list: `slurm`, `git`, `duckdb`, `gdal`, `ffmpeg`, Stata in batch mode, R via `Rscript`, and every script in the `scripts/` directory is reachable today. For most research computing, the built-in tools suffice. But what do you do when the thing you need to reach *doesn't* have a usable command line?

The **Model Context Protocol** (MCP) is an open standard for exposing tools and data to a model. You run (or connect to) an MCP **server**; the server advertises a set of tools; Claude calls them like any other built-in tool. The value of a *protocol* is that the server is written once and works with any client that speaks MCP. The author of a Zotero server doesn't need to know you're using Claude Code.

Servers reach you over one of a few transports, and the distinction has practical consequences:

| Transport | Where it runs | Typical case |
|---|---|---|
| **stdio** | A local process on your machine | Local databases, filesystem access, your own lab's tooling |
| **HTTP** / **SSE** | A remote service | Hosted SaaS APIs — Slack, GitHub, issue trackers |

A local stdio server is a program on your laptop, which means it has access to your filesystem and your network. A remote HTTP server is somebody else's service, which generally requires authentication (usually OAuth) and data leaving your machine.

> [!NOTE]
> MCP servers can expose more than tools — the protocol also covers *resources* (data Claude can read) and *prompts* (templates the server supplies). Tools are what you'll notice in practice, and what the rest of this module concerns.

## 1. Install a server

We'll use Slack, because it's an authenticated remote API with no usable CLI — exactly the case MCP.

The easiest route is the plugin marketplace, which bundles the server configuration for you:

```
/plugin
```

Search for "Slack", select it, and install. You'll be walked through an OAuth flow in your browser to authorize the workspace. Installing a plugin and connecting a server are not the same act, which is why the OAuth step is separate. The plugin ships a *configuration*; authorizing it grants that configuration access to your actual Slack workspace, under your identity. Claude will be able to read and post as you.

### Do you need a server?

You need a MCP when the system you want is **authenticated, remote, and has no usable command line**. Slack qualifies, as do many SaaS APIs, issue trackers, and hosted research platforms whose only interface is a web app and an OAuth-gated API. You almost certainly don't need it when:

- **A CLI already exists.** `gh` for GitHub, `aws`/`gcloud`, `duckdb`, `rclone`. Simply allow the command in permissions.
- **You can write fifty lines of Python.** A script in `scripts/` is versioned, reviewable, citable, testable, costs nothing in context, and runs identically for your collaborators. An MCP server is none of those things by default.
- **It's a one-off.** `.tmp/fetch_whatever.py`, as `ingest` does it.

## 2. Inspect a server

Once installed, check it:

```
/mcp
```

You should see the Slack server listed and connected. Select it to see the tools it provides — searching messages, reading channel history, posting, managing drafts, looking up users, and so on. The tool list defines what Claude can and cannot do with your Slack workspace. You should read it the way you'd read the methods available on an API client, and note in particular which of them **write**.

Here is the configuration the plugin installed (`~/.claude/plugins/cache/claude-plugins-official/slack/1.3.0/.mcp.json`):

```json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "oauth": {
        "clientId": "1601185624273.8899143856786",
        "callbackPort": 3118
      }
    }
  }
}
```

The configuration defines: a transport method ("http"), an endpoint, and an OAuth client. With this information, Claude knows how to communicate with the MCP server. "Using" the MCP is a matter of the model generating messages that the server understands.

> [!TIP]
> `/mcp` is also where you go when a server stops working. It reports connection and authentication state, and offers to re-authenticate. A server can ask for additional OAuth scope later, during a tool call, and you'll get a prompt to approve the expansion. Read those prompts; a mid-session scope request is a server asking for more access than you originally granted.

## 3. Use a server

Let's start with a read-only operation:

```
What are the recent messages in #claude-overload?
```

Claude calls the Slack server's history or search tool and summarizes the channel.

Now let's write a message to Slack:

```
Post a message to #claude-overload asking how the presentation is going.
```

The previous command read data into a context window. This one takes an externally visible action under your identity, which you cannot un-send by pressing `Esc`.

## 4. Permission a server

Everything from the permissions module applies, with one change: the rules are written against tool names rather than shell patterns.

```
/permissions
```

Add an allow rule for the read-only tools you use constantly, and leave the writes to prompt. A reasonable shape for Slack:

```json
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

You can also allow or deny a server wholesale by naming it without a tool, e.g., `mcp__plugin_slack_slack` or `mcp__plugin_slack_slack__*`.

### Trust what you connect

There are two risks to be aware of when you connect a server to real research data.

**The server itself.** A local stdio server is an executable running on your machine with your privileges. A remote one receives whatever Claude sends it. Prefer first-party servers — Slack's own, GitHub's own — and read the configuration before installing a community one.

**The data that comes back.** When Claude reads your Slack channel, that content enters its context as *data*, but a sufficiently well-crafted message is indistinguishable from an instruction. Someone who can post in a channel you ask Claude to summarize can attempt to influence what Claude does next. Claude is trained to treat retrieved content as data rather than instruction, and generally does, but the defense isn't perfect.
*Be careful where retrieved content is *untrusted*, such as public issue trackers, shared channels, and scraped pages.*

> [!WARNING]
> Data governance agreements — IRB protocols, restricted-use licenses, DUAs — typically constrain where data may be transmitted. A remote MCP server is a transmission to a third party. Before connecting one to anything covered by an agreement, read the agreement.

## Bonus

### MCP Context

A connected server's tools have to be *discoverable* for Claude to use them — but discoverable turns out to be much cheaper than loaded.

Claude Code uses a mechanism called **tool search**. A connected server's tool *names* are announced to the model; the full definitions — parameter schemas, detailed descriptions — are **deferred**, and fetched on demand only when Claude decides it needs that tool. So connecting a forty-tool server does not drop forty schemas into every request. You pay for the names and for whatever Claude actually reaches for. This is a significant improvement over the earlier behavior, and it means the warning in this section is much milder than it would have been a year ago. Still, it's worthwhile following some practical guidelines:

- **Connect servers per project, not globally.** Name crowding and blast radius are still an issue, even if tokens aren't.
- **Disconnect what you'll never use.** An unused write tool is a forgotten standing risk.
- **Prefer a narrow server to a broad one** when you have the choice.

### Shipping MCP

Like permissions and skills, MCP configuration can live in a repository. A `.mcp.json` file at the project root declares servers for anyone who works in that workspace:

```json
{
  "mcpServers": {
    "lab-db": {
      "type": "stdio",
      "command": "uv",
      "args": ["run", "mcp-server-postgres"],
      "env": { "DATABASE_URL": "${LAB_DATABASE_URL}" }
    }
  }
}
```

This is the same bargain as `data-analyst`'s `settings.json` from module 4: the repo is saying "this project expects these connections."

> [!NOTE]
> Note the `${LAB_DATABASE_URL}` indirection. The repo declares *which* variable holds the credential, never the credential itself.
