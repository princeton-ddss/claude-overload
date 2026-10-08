# Module 5: Tools and MCP — Presenter Script

**Duration:** 22 min · **Audience handout:** [README.md](README.md)

---

## Pre-flight

1. **Slack plugin installed and authorized** in whichever workspace you're demoing. Do the OAuth dance *before* the session — it opens a browser, needs a workspace admin on some plans, and is the single most likely thing to derail this module.
2. **`#claude-overload` channel exists**, you're a member, and it has 5–10 recent messages. Seed them yourself the night before if it's quiet; an empty channel makes §6 fall flat.
3. **Ask 2–3 colleagues to be in the channel** and reply when Claude posts. The live reply is the moment the module earns its slot.
4. **Second workspace tab** in a browser, showing `#claude-overload`, ready to switch to. The room needs to *see* the message land.
5. `/mcp` → confirm connected, not "needs authentication."
6. `/context` → note the current percentage. You'll compare in §8.
7. Decide whether you're demoing `.mcp.json` live or just showing the file. **Recommend: just show it.** Adding a Postgres server live is a tar pit.

> **Known hazard:** if your institution manages Claude Code settings, MCP servers may be restricted or blocked outright. Check this *before* presenting — if servers are disabled on your machine, run the module as a walkthrough of the README and spend the recovered time on §10, which is the part that matters anyway.

---

## Timing

| Time | Beat | § |
|---|---|---|
| 0:00 | What Claude already has — `Bash` is the adapter | 1 |
| 0:03 | What MCP is, and transports | 2 |
| 0:06 | Install + inspect with `/mcp` | 3–4 |
| 0:09 | The tool namespace → callback to module 4 | 5 |
| 0:11 | **Read, then write** | 6 |
| 0:14 | Permissioning MCP tools | 7 |
| 0:16 | The context bill — tool search | 8 |
| 0:18 | `.mcp.json` and institutional reality | 9 |
| 0:20 | **When you don't need a server** | 10 |
| 0:21 | Trust | 11 |

**This module's center of gravity is §10, not the demo.** The Slack demo is the hook; the scripts-vs-servers table is the thing they should leave with. Budget accordingly.

---

## 0:00 — What Claude already has [TERMINAL]

**Do:**
```
What tools do you have available?
```

**Say, while it answers:** "We've used every one of these already. Module 4 was about controlling them."

**Then make the whole module's framing claim:**

**Say:** "`Bash` is the universal adapter. Anything with a command line is *already* available — your scheduler, `git`, `duckdb`, `gdal`, Stata in batch mode, `Rscript`, every script in your own `scripts/`. No configuration, no protocol."

**The honest setup — don't skip it:** "I'm about to spend twenty minutes on how to extend Claude, so let me say up front: for most research computing you don't need to. `data-analyst` does real empirical work — Dataverse, Census API, polars, DuckDB, folium — and connects to *zero* external servers. It's all Bash and scripts."

**Pose the narrow question:** "So: what do you do when the thing you need doesn't have a usable command line?"

---

## 0:03 — What MCP is [SLIDE]

**Say:** "Open standard. You connect a *server*; the server advertises tools; Claude calls them like built-ins. Written once, works with any client that speaks the protocol."

**Transports table** — two rows, and make the consequence explicit:

- **stdio** → a process on *your machine*, with your filesystem and your network
- **HTTP/SSE** → *somebody else's service*, so OAuth, and your data leaves

**Say:** "Both deserve scrutiny. They deserve *different* scrutiny."

---

## 0:06 — Install and inspect [TERMINAL]

**Say:** "Slack: authenticated, remote, no usable CLI. Exactly the case MCP is for."

**Do:** `/plugin` → show Slack in the list. **Don't install live** if you already have it — say "I did this beforehand, it opens a browser for OAuth."

**Make the distinction:** "Installing the plugin and connecting the server are two different acts. The plugin ships a *config*. The OAuth step grants that config access to my real workspace, under my identity. Claude can now read and post **as me**."

**Do:** `/mcp` → select Slack → show the tool list.

**Say:** "This list *is* the integration. Read it like the methods on an API client — and notice which ones **write**."

**Do:** Show the config (4 lines of substance):
```bash
!cat ~/.claude/plugins/cache/claude-plugins-official/slack/*/.mcp.json
```

**Say:** "Transport, endpoint, OAuth client. That's the entire surface of an MCP integration. That's what standardizing buys you."

---

## 0:09 — The tool namespace [TERMINAL]

**Do:** Write on screen or say aloud:
```
mcp__<server>__<tool>
mcp__plugin_<plugin>_<server>__<tool>
```

**Say:** "Ugly, unambiguous. The prefix tells you where a tool came from — and whether a plugin brought it along."

**The callback — this is the beat that ties the module to module 4:**

**Do:** `/permissions` → scroll to the `mcp__computer-use__*` entries.

**Say:** "You've seen this already. Remember module 4, when the permissions list was full of `mcp__computer-use__left_click`? Same scheme. Which means everything you learned about permissions applies here."

---

## 0:11 — Read, then write [TERMINAL] ⭐

**Do (read):**
```
What are the recent messages in #claude-overload?
```

**Say:** "Nothing changed in the world. We pulled data into a context window."

**Do (write) — switch the projector to the Slack tab first:**
```
Post a message to #claude-overload asking how the presentation is going.
```

**Let the room watch it land.** Wait for a colleague's reply. This is the laugh, and it's also the lesson.

**Say, while replies come in:** "Feel the difference. The first command read data. This one took an irreversible, externally visible action under my identity — a message my colleagues can see, which I cannot un-send with `Esc`."

**The lens to give them:** *"Read is cheap and reversible. Write is neither. When you evaluate any server — Slack, GitHub, your institutional database — sort its tools into those two buckets first."*

### Failure modes

| Symptom | Recovery |
|---|---|
| "Needs authentication" | Don't fix live. Narrate the tool list from the README; skip to §7. |
| Channel not found | Claude needs to be in the channel. Have a fallback channel name ready. |
| Nobody replies | Have a colleague on standby by text. Or just say "they're all in this room, which is its own answer." |
| Posts the wrong thing | **Excellent.** "And that is why writes stay behind a prompt." Segue straight to §7. |

---

## 0:14 — Permissioning MCP tools [TERMINAL]

**Do:** `/permissions` → show adding a rule by tool name.

**Say:** "Same system as module 4, rules written against tool names instead of shell patterns. Allow the reads you use constantly; leave the writes to prompt."

**The auto-mode warning — say it slowly, it's the real risk:**

**Say:** "Module 4 said auto mode allows anything the classifier approves. A tool that posts to a channel of five hundred colleagues may read as perfectly innocuous — nothing about it is destructive *on your machine*. The consequences are social, not technical. If you run auto mode with write-capable servers connected, put the writes behind explicit Deny or Ask rules yourself."

---

## 0:16 — The context bill [TERMINAL]

**Do:** `/context`

**Lead with the good news — don't overstate this beat:**

**Say:** "Claude Code uses **tool search**. A connected server's tool *names* are announced; the full schemas are deferred and fetched only when Claude actually reaches for one. So connecting a forty-tool server does not put forty schemas in every request."

**Then the three caveats, fast:**
1. Names still accumulate — cost scales with tool *count*, not schema size.
2. Tool search isn't universal; it can be off on some platforms or behind a proxy. That's the expensive case.
3. **Crowding** — same point as `/skill-doctor` in module 3. A tool Claude must choose among can be chosen wrongly. A hundred half-relevant names is a worse menu than ten good ones.

**Say:** "`\"alwaysLoad\": false` on a server config defers all of that server's tools behind tool search — the right setting for something you want available but rarely use."

**Revised guidance — note the reason changed:**
- Connect per project, not globally. Mostly for crowding and blast radius now, **not** for tokens.
- Disconnect what you'll never use — because an unused *write* tool is standing risk, not because it's expensive.
- Prefer narrow servers to broad ones.

**Useful aside:** "If Claude seems to forget your instructions in long sessions, check `/context` before blaming the model."

> **Don't claim servers eat your context window.** They mostly don't, anymore. If someone in the room has read older material saying otherwise, this is a good place to say the behavior changed.

---

## 0:18 — .mcp.json and institutional reality [TERMINAL]

**Do:** Show the `.mcp.json` example from README §9 (don't build one live).

**Say:** "Same bargain as `data-analyst`'s `settings.json`: the repo says 'this project expects these connections.' Same warning, sharper — a permission rule grants access to commands on your machine; a `.mcp.json` entry can point at a server that *runs code* on your machine or ships your data elsewhere."

**Point at `${LAB_DATABASE_URL}`:** "The repo declares which variable holds the credential, never the credential. Same discipline as module 4's API keys."

**Then the one slide your audience's institution cares about:**

**Say:** "Claude Code supports managed settings that restrict which servers are permitted — there's an `allowedMcpServers` allowlist and a switch to disable connectors entirely. If you want MCP in a lab workflow at a university with a security office, that's the vocabulary for the conversation. And it's a much easier conversation *before* someone connects a server to restricted-use data."

---

## 0:20 — When you don't need a server [SLIDE] ⭐

**This is the takeaway. Do not let it get squeezed.**

**Say:** "MCP earns its place when the system is **authenticated, remote, and has no usable CLI**. Slack qualifies."

**Then the list of when it doesn't:**
- A CLI exists → `gh`, `gcloud`, `duckdb`, `rclone`. Allow it in permissions, done.
- You could write fifty lines of Python → versioned, reviewable, citable, testable, free in context.
- It's a one-off → `.tmp/`, like `ingest` does.

**Put the comparison table up and read the bottom row:** script vs. server across context cost, versioning, reviewability, reproducibility — and then OAuth-gated APIs, where the server wins.

**The rule to leave them with:** *"Reach for MCP at the authentication boundary. If the hard part is getting in, use a server. If the hard part is what you do once you're in, write a script."*

---

## 0:21 — Trust [SLIDE]

**Two risks, one line each:**

1. **The server.** Local stdio = an executable with your privileges. Remote = receives what Claude sends. You're extending your trust boundary to its author with none of the review a Python dependency gets. Prefer first-party.
2. **The data coming back.** "When Claude reads your Slack channel, that content lands in context as *data* — but a well-crafted message is hard to distinguish from an instruction. Anyone who can post in a channel you summarize can try to influence what Claude does next. Claude is trained to treat retrieved content as data and generally does — but the real defense is keeping writes behind prompts."

**Close the module on the governance point. For this audience it's the most important sentence in it:**

**Say:** "IRB protocols, restricted-use licences, DUAs — these constrain where data may be transmitted. A remote MCP server *is* a transmission to a third party. Before you connect one to anything covered by an agreement, read the agreement. 'The AI tool did it' is not a defence. The obligation was yours."

---

## Questions to expect

- **"Is there an MCP server for Zotero / arXiv / PubMed / our cluster?"** — Community servers exist for a lot of this; quality varies and nothing is vetted by anyone. Check the repo before installing. And ask whether a CLI or API script wouldn't be simpler — usually it is.
- **"Can I write my own?"** — Yes, there are SDKs in several languages. But see §10: for lab-internal work a CLI plus a permission rule gets you there faster and is easier for a collaborator to reproduce.
- **"Does my data go to Anthropic?"** — Conversation content passes through Anthropic's API, as it does for everything in this workshop. An MCP server adds a *second* recipient: the server operator. That's the new thing to reason about.
- **"Can I use this to put our database behind Claude?"** — Technically yes. Start read-only with a restricted account, and talk to whoever owns the data first.
- **"Why not just connect everything?"** — §8. Context cost, plus every connected write tool is standing risk.

## If you're behind

Cut in this order:
1. §9 `.mcp.json` → keep only the institutional-controls sentence
2. §5 namespace → fold the module 4 callback into §7
3. §3 install → you pre-installed anyway; start at `/mcp`
4. **Never cut §6 or §10.** The write demo is why they'll remember the module; §10 is why it was worth teaching.
