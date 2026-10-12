# Skills — Presenter Script

## Setup

1. Create sessions
   - **Tab A (clean):** fresh clone, `data/` empty.
   - **Tab B (loaded):** election and ACS already ingested.
2. Add `CENSUS_API_KEY` to `.env`.
3. Prepare `describe/SKILL.md` draft.

## 1. List and read a skill

- Show locations:
   - Project: `.claude/skills/`
   - User `~/.claude/skills/`
   - Plugins

- Run `/skills` command
   - Six `project` skills; the rest are built-ins, plugins, and mine.
   - Token counts are 50–340 each, always loaded.

- Prompt 💬 "Show me @.claude/skills/setup/SKILL.md"
   - **Frontmatter**: description is what Claude sees before deciding to use the skill.
   - **Body**: plain Markdown. "If you can write a README, you can write a skill."

## 2. Invoke a skill ⭐

- Method 1: Ask
   - In Tab A, 💬 "Set up the project environment."
   - Highlight `Skill(setup)` in the transcript.

- Method 2: Command
   - Shows `!ls data/` is empty.
   - Run `/ingest mit election lab county presidential returns, 2000 through 2024` 
      1. **Raw file** `data/raw/countypres_2000-2024.csv`. *"Never modify raw files after saving — they are the source of truth."*
      2. **Cleaning script.** `scripts/fetch_countypres.py`, `scripts/clean_countypres.py`.
      3. **Throwaway to `.tmp/`** (gitignored). The skill encodes what's reproducible and what isn't.
      4. **Source registered.** `!cat data/sources.yaml`

## 3. Write a skill: `describe` ⭐

- Switch to Tab B (if needed)

- Run `mkdir -p .claude/skills/describe`, `nvim .claude/skills/describe/SKILL.md`
   - Paste the `SKILL.md` from `draft-describe.md`
   - **Condition**
   - **Summary**
   - **Examples**
   - **Precise and complete (Body)**

- Check that `/describe` is available
   - Ask to "Summarize the Census data."

## 4. Skill tips

- **Progressive disclosure**
   - Show `!ls .claude/skills/notebook/references/`
   - 8 reference files, 700 lines, read only when needed.
   - `SKILL.md` links to references using Markdown syntax. Claude follows *as needed*.

- **Bundle code**
   - DRY — "Don't repeat yourself"
   - Skills are not a replacement for code.
     
- **`/skill-creator`**
   - Generates a first draft in 30 seconds, which is why I showed it *last*.
   - Can also evaluate a skill and and benchmark it.

- **`/skill-doctor`**
   - Find unused skills and token usage
