# Tools and MCP — Presenter Script

**Budget:** 13 min (of a 60-min block: skills 35 · hooks 12 · MCP 13) · **Handout:** [README.md](README.md)

---

## Pre-flight

1. **Slack plugin installed and authorized** before the session. The OAuth flow opens a browser, may need a workspace admin, and is the likeliest thing to derail this segment.
2. **`#claude-overload` exists**, you're a member, and it has 5–10 recent messages.
3. **Ask 2–3 colleagues to be in the channel** and reply when Claude posts. The live reply is the moment the segment earns its slot.
4. **A browser tab on `#claude-overload`**, ready to switch to so the room sees the message land.
5. `/mcp` → connected, not "needs authentication."
6. **Copy the exact tool names from `/mcp`** if you show a permission rule. The README's example names (`search_messages`, `send_message`) are not the real ones; they are `slack_search_public` and `slack_send_message`, e.g. `mcp__plugin_slack_slack__slack_send_message`.
7. **Check whether your institution restricts MCP.** If servers are blocked on your machine, walk the README and spend the time on "Do you need a server?"

---

## Timing

| Min | Beat | README |
|---|---|---|
| 0–2 | `Bash` is the adapter; what MCP is | intro |
| 2–4 | Install (already done) and inspect | §1–2 |
| 4–8 | **Read, then write** | §3 |
| 8–10 | Permission it | §4 |
| 10–12 | **Do you need a server?** | §1, "Do you need a server?" |
| 12–13 | Trust | §4, "Trust what you connect" |

**The Slack demo is the hook; "Do you need a server?" is the takeaway.** If time is short, cut from the permission beat, not that one.

**Dropped from the longer version:** the tool-namespace walkthrough, the context-bill discussion, and `.mcp.json`. They are in the README's Bonus. Answer them if asked.

---

## 0:00 — `Bash` is the adapter [TERMINAL + SLIDE]

**Do:** `What tools do you have available?`

**Say:** "We've used all of these already. `Bash` is the universal adapter: anything with a command line is *already* available. `git`, `duckdb`, `gdal`, `Rscript`, Stata in batch mode, every script in `scripts/`. `data-analyst` does real empirical work against the Census API and Dataverse with **zero** external servers."

**Pose the narrow question:** "So what do you do when the thing you need has no usable command line?"

**MCP in three sentences:** "An open standard. You connect a *server*; it advertises tools; Claude calls them like built-ins. Two kinds: **stdio** runs on your machine with your privileges; **HTTP** is somebody else's service, so OAuth, and your data leaves."

---

## 0:02 — Install and inspect [TERMINAL]

**Do:** `/plugin` → show Slack. "Already installed. It opens a browser for OAuth."

**Make the distinction:** "Installing the plugin ships a *config*. The OAuth step grants it access to my real workspace, under my identity. Claude can now read and post **as me**."

**Do:** `/mcp` → Slack → show the tool list. "This list *is* the integration. Read it like the methods on an API client and notice which ones write."

**Do:** `!cat ~/.claude/plugins/cache/claude-plugins-official/slack/*/.mcp.json`. "Transport, endpoint, OAuth client: the whole surface of an integration."

---

## 0:04 — Read, then write [TERMINAL] ⭐

**Do (read):** `What are the recent messages in #claude-overload?`
"Nothing changed in the world. We pulled data into a context window."

**Switch the projector to the Slack tab, then (write):**
```
Post a message to #claude-overload asking how the presentation is going.
```

**Let the room watch it land.** Wait for a colleague's reply.

**Say:** "Feel the difference. That was an externally visible action under my identity, and I can't un-send it with `Esc`."

**The lens:** *"Read is cheap and reversible; write is neither. For any server, sort its tools into those two buckets first."*

| Symptom | Recovery |
|---|---|
| "Needs authentication" | Don't fix live. Narrate the tool list; skip to permissions. |
| Channel not found | Claude must be in it. Have a fallback channel. |
| Nobody replies | Text a colleague, or: "they're all in this room, which is its own answer." |
| Posts the wrong thing | **Excellent.** "That's why writes stay behind a prompt." Go straight to permissions. |

---

## 0:08 — Permission it [TERMINAL]

**Do:** `/permissions` → show adding a rule by tool name.

**Say:** "Same system as the permissions session, rules written against tool names. **Allow** the reads you use constantly; **ask** for the writes." Show the allow-search / ask-send shape from README §4.

**One line:** "Name just the server to cover all its tools, e.g. `mcp__plugin_slack_slack` or `mcp__plugin_slack_slack__*`."

**The auto-mode warning, slowly:** "A tool that posts to five hundred colleagues may look innocuous to the auto-mode classifier, because nothing about it is destructive *on your machine*. The consequences are social. With write-capable servers connected, put the writes behind explicit **Ask** or **Deny** rules yourself."

---

## 0:10 — Do you need a server? [SLIDE] ⭐

**This is the takeaway. Don't let it get squeezed.**

**Say:** "The honest answer for most research computing is no."

- **A CLI exists** → `gh`, `gcloud`, `duckdb`, `rclone`. Allow it in permissions; done.
- **A simple API key** → a script. `ingest` calls the Census API with `CENSUS_API_KEY` from the environment, no server involved. Versioned, reviewable, free in context, identical for your collaborators, and the key stays out of the conversation.
- **A one-off** → `.tmp/fetch_whatever.py`.

**When a server earns its place:** delegated **OAuth**, where each user authorizes their own account and tokens refresh. Hand-rolling that is the painful part. Or a large API someone else already maintains a good server for.

**The rule:** *"Reach for MCP when the hard part is getting in. If a key in an environment variable is enough, write a script."*

*(The README's "Do you need a server?" still reads "authenticated, remote, and no usable CLI." That's looser than this rule; a simple key-authenticated API with no CLI doesn't need MCP.)*

---

## 0:12 — Trust [SLIDE]

**Two risks, one line each:**
1. **The server.** A stdio server is an executable with your privileges; a remote one receives whatever Claude sends it. Prefer first-party.
2. **The data coming back.** "A well-crafted Slack message is hard to tell from an instruction. Keep writes behind prompts."

**Close on governance. For this audience it's the most important sentence in the segment:**

"IRB protocols, restricted-use licenses, DUAs constrain where data may be transmitted. A remote MCP server **is** a transmission to a third party. Read the agreement before you connect one. 'The AI tool did it' is not a defense."

---

## Questions to expect

- **"Is there a server for Zotero / arXiv / PubMed / our cluster?"** Community servers exist; quality varies and nothing is vetted. Check the repo, and ask whether a script wouldn't be simpler.
- **"Does my data go to Anthropic?"** Conversation content passes through Anthropic's API as it does everywhere in this workshop. An MCP server adds a *second* recipient: the operator.
- **"Doesn't a big server eat my context?"** Mostly no. Tool search announces names and loads schemas on demand. Names still accumulate (see the README's Bonus), and crowding is the remaining cost.
- **"Can I share servers with my lab?"** A `.mcp.json` at the project root. Same bargain as shipped permissions and hooks: read it before you run it. (README Bonus.)

## If you're behind

1. Skip the permission beat; keep only the auto-mode warning.
2. Skip the config `cat`.
3. **Never cut the write demo or "Do you need a server?"** One is why they'll remember it; the other is why it was worth teaching.
