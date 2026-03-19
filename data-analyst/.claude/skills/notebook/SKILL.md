---
name: notebook
description: >
  Open a new or existing Marimo notebook for interactive analysis.
  Examples: "/notebook", "/notebook turnout", "open a notebook".
user_invocable: true
---

# Notebook

Create or open a Marimo notebook in `notebooks/`.

1. If a name is given, check if `notebooks/<name>.py` exists
2. If it exists, open it
3. If not, create it using the `marimo-notebook` skill conventions
4. If no name given, use a timestamp: `notebook_YYYYMMDD_HHMMSS.py`
5. Open with `marimo edit --watch <file>` (use `run_in_background: true`)
