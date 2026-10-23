# layer-clawsweeper

The ClawSweeper review engine packaged as a charly candy, so a repository can run a
first-party, policy-driven PR review on the OpenCharly self-hosted fleet with **no GitHub
App, no webhook, no Cloudflare Worker and no queue** — and with the org's own Ollama Cloud
endpoint as the model backend.

## What it provides

`candy/clawsweeper-review` builds an Ubuntu image containing:

| Piece | Pin | Why |
|---|---|---|
| ClawSweeper source | `CLAWSWEEPER_COMMIT` | the review engine, built from a pinned upstream commit |
| Node | `NODE_VERSION` (24.x) | the engine's runtime |
| pnpm | `PNPM_VERSION` (12.4.1) | the engine's package manager, via corepack |
| OpenAI Codex CLI | `CODEX_VERSION` (0.162.1) | the engine's only runner; Codex accepts only `wire_api = "responses"` |
| Ollama Cloud provider | `/etc/clawsweeper/codex-config.toml` | `https://ollama.com/v1`, model `deepseek-v4.1-flash` |
| `opencharly` target profile | config overlay | any `opencharly/<repo>` resolves without a per-repo entry |
| `clawsweeper-review` | `/usr/local/bin` | one-shot `--local-range` review, JSON verdict on stdout |

The engine/endpoint pairing is measured, not assumed: Codex performs a real shell tool call
against Ollama Cloud (`codex exec` ran `uname -s && uname -m` and returned `Linux`/`x86_64`).

## Why a container, and why `--local-range`

The review reads the PR tree — untrusted content — and Codex can run shell commands while
gathering evidence. Running it in a **disposable container with the tree mounted read-only
and no GitHub token** keeps untrusted content away from the runner host and from any
credential. `--local-range` reads `merge-base(base, HEAD)..HEAD`, refuses a dirty tree, and
makes no GitHub reads or writes, so the verdict can leave as JSON for the caller to post.

## Build and validate

```bash
charly box validate
charly box build clawsweeper-review
```

The candy's `plan:` asserts, at build time: Node 24, pnpm 12.4.1, Codex 0.162.1, the built
engine's own `--help`, the `opencharly` profile entry, the Codex config's base URL and wire
API, and the entrypoint's presence.

## Related

- `/charly-check:clawsweeper-review` — the candy's skill body (`.agents/skills/`).
- `opencharly/distro-ubuntu` — the box and the disposable bed that runs a real review.
- `opencharly/.github` — the reusable workflow that calls the image from a PR job.
