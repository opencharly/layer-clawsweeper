# AGENTS.md — layer-clawsweeper

The ClawSweeper review engine as a charly candy. Part I of the umbrella rulebook
(`opencharly/opencharly`'s `AGENTS.md`) governs this repo; this file adds only what is
specific to it.

## What lives here

- `candy/clawsweeper-review/charly.yml` — the candy that builds the review image, its
  pinned versions, the Codex/Ollama configuration, the profile overlay and the entrypoint.
- `.agents/skills/clawsweeper-review/SKILL.md` — the skill body (routing entity is the
  candy's own `skill:` node).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + CHANGELOG on merge.

## Rules specific to this repo

- **Pin everything.** The upstream commit, Node, pnpm and Codex are `var:`s; advancing any of
  them is producer-first (upstream merge → this pin → the image → the consumer). Never
  `latest`.
- **The review is advisory.** It never gates a merge and never writes to GitHub. It gets no
  token, and `--local-range` is the only mode this image offers.
- **The Ollama Cloud contract is measured.** `wire_api = "responses"` is the only value Codex
  accepts, and the endpoint is `https://ollama.com/v1`. If a probe contradicts this, the
  skill's section changes first.
- **No secrets in the image.** `OLLAMA_API_KEY` is supplied at run time by the caller; the
  config names the env var, never a value.
