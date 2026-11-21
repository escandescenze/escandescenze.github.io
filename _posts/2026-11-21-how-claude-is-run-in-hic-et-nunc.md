---
layout: post
title: "How Claude is run in hic et nunc"
date: 2026-11-21
---

A companion to [I tried to build proof that art isn't AI]({% post_url 2026-11-13-proof-of-human-art %}):
how the code of [hic et nunc](https://github.com/andreanerla/hic-et-nunc) was written with Claude Code.

One Claude Code session per task, steered by the repo's control files and gated by Andrea at three points. Sources: `CLAUDE.md`, `PROMPTS.md`, `.claude/settings.json`, D-013.

<pre class="mermaid">
flowchart LR
  A(["Andrea"])

  subgraph CTRL["Control files (repo)"]
    direction TB
    CM["CLAUDE.md<br>rules, loaded automatically"]
    PR["PRODUCT_RULES.md<br>decision order"]
    SP["SPEC.md<br>scope"]
    DE["DECISIONS.md<br>append-only"]
    ST["STATE.md<br>current task, queue, known problems"]
  end

  subgraph CC["Claude Code session (one task)"]
    direction TB
    S1["1 Start<br>read SPEC, DECISIONS, STATE<br>5-line reply"]
    S2["2 Plan<br>files, approach, not doing"]
    S3["3 Build<br>smallest diff"]
    S4["4 Check<br>pytest + real-data table"]
    S5["5 Wrap up<br>rewrite STATE, add decisions,<br>propose SPEC edit + commit"]
    S1 --> S2 --> S3 --> S4 --> S5
  end

  HK["Hook on every .py edit<br>ruff check + format, blocks on error"]
  DATA[("Real recordings<br>outside the repo<br>HIC_ET_NUNC_KRITA_DATA")]
  CHAT["Claude chat<br>reviews against PRODUCT_RULES<br>and 'Done when'"]
  PUB["Public repo<br>hand export of PUBLIC_FILES.txt"]

  A -- "prompt from PROMPTS.md" --> S1
  S1 -. "confirm task" .-> A
  S2 -. "approve plan" .-> A
  S5 -. "approve decisions, SPEC, commit" .-> A
  CTRL --> S1
  S3 --- HK
  S4 --- DATA
  S5 --> ST
  S5 --> DE
  A --> CHAT
  CHAT -. "review" .-> A
  A --> PUB
</pre>

<script type="module">
  import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs";
  mermaid.initialize({ startOnLoad: true, theme: "neutral" });
</script>

### Key

- **Solid arrow**: input or output, what Claude reads or writes.
- **Dotted arrow**: a gate, where Claude stops and waits for Andrea.
- **Hook**: `.claude/settings.json`, PostToolUse on Edit/Write. Python rules
  Ruff can't check are in `.claude/rules/python.md`.
- **Chat review**: D-013. The decision record gives its context and reason as
  "unknown".
- **Not used**: project-defined subagents, skills, CI, scheduled agents, MCP
  servers.
