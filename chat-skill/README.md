# chat-skill

Source for the **claude.ai chat skills** that drive the upstream half of the
workflow. These are uploaded once to claude.ai (Settings → Capabilities →
Skills) and apply across every framework-based project — they are **not**
distributed to downstream repos by `/upgrade`.

- `spec-author/SKILL.md` — a thin wrapper that authors a feature spec and
  commits it to `main`, ready for `/ship`. It deliberately contains only the
  *workflow*; the *contract* for what a spec holds lives in each repo's
  `spec-guide.md`, which the skill fetches via MCP at runtime. Keep it that
  way — duplicating guide content here would create a second source of truth
  that drifts.

When a skill file changes, re-upload it to claude.ai; this folder is the
canonical, versioned copy.
