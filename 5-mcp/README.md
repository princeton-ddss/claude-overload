# Tools and MCP

## 1. Inventory what Claude already has

Everything Claude does to the outside world, it does through a *tool*. We've been using them all workshop without naming them: `Bash` to run commands, `Read` and `Write` for files, `Glob` and `Grep` to search, `WebFetch` and `WebSearch` to pull from the internet, `Skill` to invoke the protocols from module 3. Module 4 was entirely about controlling them.

Take stock before adding anything. Ask Claude:

```
What tools do you have available?
```

The built-in set is small and general, and one member of it is doing most of the work. `Bash` is the universal adapter: anything with a command-line interface is already available to Claude without configuration, integration, or a protocol. Your HPC scheduler, `git`, `duckdb`, `gdal`, `ffmpeg`, Stata in batch mode, R via `Rscript`, every script in your own `scripts/` directory — all of it is reachable today.

This is worth saying plainly at the start of a module about extending Claude, because the honest answer for most research computing is that **you don't need to extend it**. `data-analyst` does real empirical work — fetches from Harvard Dataverse and the Census API, cleans with polars, queries DuckDB, renders maps with folium — and it connects to exactly zero external tool servers. It's all `Bash` and scripts.

So the question this module answers is narrow: what do you do when the thing you need to reach *doesn't* have a usable command line?

## 2. Understand what MCP is

The **Model Context Protocol** (MCP) is an open standard for exposing tools, and data, to a model. You run (or connect to) an MCP **server**; the server advertises a set of tools; Claude can then call them like any built-in.

The value of a *protocol* here, rather than a pile of bespoke integrations, is that the server is written once and works with any client that speaks MCP. The author of a Zotero server doesn't need to know you're using Claude Code.

Servers reach you over one of a few transports, and the distinction has practical consequences:

| Transport | Where it runs | Typical case |
|---|---|---|
| **stdio** | A local process on your machine | Local databases, filesystem access, your own lab's tooling |
| **HTTP** / **SSE** | A remote service | Hosted SaaS APIs — Slack, GitHub, issue trackers |

A local stdio server is a program on your laptop, which means it has your filesystem and your network. A remote HTTP server is somebody else's service, which means authentication, usually OAuth, and data leaving your machine. Both deserve scrutiny; they deserve *different* scrutiny.

> [!NOTE]
> MCP servers can expose more than tools — the protocol also covers *resources* (data Claude can read) and *prompts* (templates the server supplies). Tools are what you'll notice in practice, and what the rest of this module concerns.

## 3. Install a server

We'll use Slack, because it's an authenticated remote API with no usable CLI — exactly the case MCP is for — and because the results are immediately visible to a room.

The easiest route is the plugin marketplace, which bundles the server configuration for you:

```
/plugin
```

Search for "Slack", select it, and install. You'll be walked through an OAuth flow in your browser to authorize the workspace. Module 7 covers plugins properly; for now treat this as a convenient installer.

> [!NOTE]
> Installing a plugin and connecting a server are not the same act, which is why the OAuth step is separate. The plugin ships a *configuration*; authorizing it grants that configuration access to your actual Slack workspace, under your identity. Claude will be able to read and post as you.

Plugins aren't the only way in. You can register a server directly from the command line:

```bash
claude mcp add --help
claude mcp list
```

…or declare one in configuration, which we'll get to in section 9.

## 4. Inspect the connection with /mcp

Once installed, check it:

```
/mcp
```

You should see the Slack server listed and connected. Select it to see the tools it provides — searching messages, reading channel history, posting, managing drafts, looking up users, and so on.

Spend a moment here. The tool list *is* the integration: it defines precisely what Claude can and cannot do with your Slack workspace. Read it the way you'd read the methods available on an API client, and note in particular which of them **write**.

Here is the configuration the plugin installed:

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

Four lines of substance: a transport, an endpoint, and an OAuth client. That's the whole surface of an MCP integration, which is the point of standardizing it.

> [!TIP]
> `/mcp` is also where you go when a server stops working — it reports connection and authentication state, and offers to re-authenticate. A server can ask for additional OAuth scope later, during a tool call, and you'll get a prompt to approve the expansion. Read those prompts; a mid-session scope request is a server asking for more access than you originally granted.

## 5. Read the tool namespace

MCP tools are named mechanically, and learning to read the name tells you where a tool came from:

```
mcp__<server>__<tool>
mcp__plugin_<plugin>_<server>__<tool>
```

So a server you registered yourself as `slack` gives you `mcp__slack__search_messages`, while the same server arriving via the Slack *plugin* gives you `mcp__plugin_slack_slack__search_messages`. Ugly, but unambiguous — and the prefix is how you tell a first-party tool from something a plugin brought along.

If this looks familiar, it's because you've seen it already. When we ran `/permissions` in module 4, the list was full of entries like:

```
mcp__computer-use__left_click
mcp__computer-use__screenshot
```

That's the same naming scheme, which brings us to the part that matters.

## 6. Use the server

Start with a read:

```
What are the recent messages in #claude-overload?
```

Claude calls the Slack server's history or search tool and summarizes the channel. Nothing has changed in the world; we've only pulled data in.

Now a write:

```
Post a message to #claude-overload asking how the presentation is going.
```

This is a different kind of act, and you should feel the difference. The previous command read data into a context window. This one takes an irreversible, externally visible action under your identity — a message your colleagues will see, which you cannot un-send by pressing `Esc`.

> [!TIP]
> This read/write asymmetry is the single most useful lens for thinking about MCP. Reading is cheap and reversible; writing is neither. When you evaluate any server — Slack, GitHub, your institutional database — sort its tools into those two buckets first, and set permissions accordingly.

## 7. Permission MCP tools

Everything from module 4 applies, with one change: the rules are written against tool names rather than shell patterns.

```
/permissions
```

Add an allow rule for the read-only tools you use constantly, and leave the writes to prompt. A reasonable shape for Slack:

```json
{
  "permissions": {
    "allow": [
      "mcp__plugin_slack_slack__search_messages"
    ],
    "deny": [
      "mcp__plugin_slack_slack__send_message"
    ]
  }
}
```

You can also allow or deny a server wholesale by naming it without a tool.

> [!NOTE]
> This is where auto mode deserves a second thought. Module 4 described it as allowing "any action the classifier approves except `Ask` and `Deny` rules" — and a tool that posts to a channel of five hundred colleagues may well read as innocuous to a classifier, because nothing about it is destructive on your machine. The consequences are social rather than technical. If you run in auto mode with write-capable servers connected, put the writes behind explicit `Deny` or `Ask` rules yourself.

## 8. Count the cost

Return to the context ladder from module 3. A connected server's tools have to be *discoverable* for Claude to use them — but discoverable turns out to be much cheaper than loaded.

Claude Code uses a mechanism called **tool search**. A connected server's tool *names* are announced to the model; the full definitions — parameter schemas, detailed descriptions — are **deferred**, and fetched on demand only when Claude decides it needs that tool. So connecting a forty-tool server does not drop forty schemas into every request. You pay for the names and for whatever Claude actually reaches for.

This is a significant improvement over the earlier behavior, and it means the warning in this section is much milder than it would have been a year ago. Connect a server you rarely use and the standing cost is small.

Three caveats keep it from being free:

1. **Names still accumulate.** The cost scales with how many tools exist across your connected servers, even if it no longer scales with how verbose each one's schema is.
2. **Tool search isn't universal.** It can be off — on some model and platform combinations, or behind a proxy or gateway that doesn't support it. With tool search off, you *do* get the full definitions up front, which is the expensive case.
3. **Crowding, again.** This is the same point as `/skill-doctor` in module 3. A tool Claude must choose among is a tool that can be chosen wrongly, and a hundred half-relevant tool names is a worse menu than ten good ones.

Measure rather than assume:

```
/context
```

You can also make the deferral explicit. Setting `"alwaysLoad": false` on a server's configuration defers *all* of that server's tools behind tool search, which is the right setting for something you want available but rarely use.

Practical guidance, now appropriately scaled:

- **Connect servers per project, not globally.** Still the main rule — mostly for crowding and for blast radius, not for tokens.
- **Disconnect what you'll never use.** Not because it's expensive, but because an unused write tool is standing risk (section 11).
- **Prefer a narrow server to a broad one** when you have the choice.

> [!TIP]
> If Claude seems to have forgotten your instructions in a long session, check `/context` before blaming the model. `CLAUDE.md`, skill descriptions, and tool names are all competing for the same budget — and after this module, you have a command for auditing each of them.

## 9. Ship servers with a project

Like permissions and skills, MCP configuration can live in the repository. A `.mcp.json` file at the project root declares servers for anyone who works in that workspace:

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

This is the same bargain as `data-analyst`'s `settings.json` from module 4: the repo is saying "this project expects these connections." And it carries the same warning, more sharply. A permission rule grants access to commands on your own machine. A `.mcp.json` entry can point at a server that runs code on your machine or sends your data somewhere else.

> [!NOTE]
> Note the `${LAB_DATABASE_URL}` indirection. The repo declares *which* variable holds the credential, never the credential itself — the same discipline module 4 recommended for API keys and that `ingest` enforces in module 3. If you find yourself about to commit a connection string, that's the pattern you want.

> [!TIP]
> Institutional context matters here. Claude Code supports managed settings that restrict which servers are permitted, including an `allowedMcpServers` allowlist and a switch to disable connectors outright. If you're at a university with a security office and you want MCP in a lab workflow, those controls are the vocabulary to have that conversation in — and it's a much easier conversation to start before someone connects a server to clinical or restricted-use data.

## 10. Decide whether you need a server at all

Now that it works, the useful question: when is this the right tool?

You need MCP when the system you want is **authenticated, remote, and has no usable command line**. Slack qualifies. So do most SaaS APIs, issue trackers, and hosted research platforms whose only interface is a web app and an OAuth-gated API.

You almost certainly don't need it when:

- **A CLI already exists.** `gh` for GitHub, `aws`/`gcloud`, `duckdb`, `rclone`. Allow the command in module 4's permissions and you're done.
- **You can write fifty lines of Python.** A script in `scripts/` is versioned, reviewable, citable, testable, costs nothing in context, and runs identically for your collaborators. An MCP server is none of those things by default.
- **It's a one-off.** `.tmp/fetch_whatever.py`, as `ingest` does it.

The comparison is unflattering to MCP for research work, and it should be. Weigh it honestly:

| | Script in `scripts/` | MCP server |
|---|---|---|
| Context cost | None | Tool names announced; schemas on demand |
| Versioned with the project | Yes | Only the config |
| Reviewable in a pull request | Yes | Not the server's code |
| Reproducible by a collaborator | Yes | Needs their own auth |
| Works for OAuth-gated APIs | Painfully | Yes — this is the point |

> [!TIP]
> A good rule: reach for MCP at the authentication boundary. If the hard part is *getting in*, a server is probably worth it. If the hard part is what you do once you're in, write a script.

## 11. Trust what you connect

Two risks deserve naming before you connect anything to real research data.

**The server itself.** A local stdio server is an executable running on your machine with your privileges. A remote one receives whatever Claude sends it. You are extending your trust boundary to its author, with nothing like the review a Python dependency gets. Prefer first-party servers — Slack's own, GitHub's own — and read the configuration before installing a community one.

**The data that comes back.** This one is less obvious and more interesting. When Claude reads your Slack channel, that content enters its context as *data* — but a sufficiently well-crafted message is indistinguishable from an instruction. Someone who can post in a channel you ask Claude to summarize can attempt to influence what Claude does next. Claude is trained to treat retrieved content as data rather than instruction, and generally does, but the defense is structural rather than perfect:

- Keep write tools behind prompts, so an injected instruction can't quietly act.
- Be most careful where retrieved content is *untrusted* — public issue trackers, shared channels, scraped pages.
- Watch the transcript. An unexpected tool call right after reading external content is the signal.

> [!NOTE]
> For an academic audience there's a sharper version of this. Data governance agreements — IRB protocols, restricted-use licences, DUAs — typically constrain where data may be transmitted. A remote MCP server is a transmission to a third party. Before connecting one to anything covered by an agreement, read the agreement. "The AI tool did it" is not a defence, and the obligation was yours.
