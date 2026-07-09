# playerlog — landing page Worker for playerlog.ai

> ✅ **What this repo actually is:** a tiny Cloudflare Worker that serves a
> single static HTML page (the landing page for the `playerlog.ai` domain).
> That's the whole codebase. The Worker has **no application logic** — it is a
> five-line asset passthrough.

This repo is **not** the Python package it advertises. The landing page describes
`playerlog-agent`, a thin "PLATO domain agent" for logging and querying
game-player activity. That package is published on PyPI and its source lives in a
**separate** repo ([`SuperInstance/playerlog-agent`](https://github.com/SuperInstance/playerlog-agent)).
This repo is just the billboard that points at it.

If you arrived here expecting player-activity/analytics code, you want the
`-agent` package, not this repo.

## What problem does the *advertised* package aim at?

Games produce a stream of small events (match finished, level gained,
achievement unlocked). Each event is useful in the moment, but the real signal
is the accumulation — "did this player's engagement drop after the patch?"
`playerlog-agent` is the smallest honest stab at giving that history durable
memory: write player activity into PLATO as tiles, query it back later. (See
"Honesty markers" below — the package is a deliberate stub.)

## Repository layout

```
playerlog/
├── src/index.ts        # The Worker. Five lines. Delegates every request to env.ASSETS.
├── public/index.html   # The landing page (HTML + inlined CSS). The only served asset.
├── wrangler.jsonc      # Cloudflare Workers config: name="playerlog", assets dir=./public
├── family/             # Shared design-system skeleton (see family/README.md)
└── .gitignore
```

**`src/index.ts`** in full:

```ts
export default {
  async fetch(request: Request, env: { ASSETS: Fetcher }): Promise<Response> {
    return env.ASSETS.fetch(request);
  },
};
```

Every request is handed to the Workers [Static Assets](https://developers.cloudflare.com/workers/static-assets/)
binding, which serves `public/index.html`. There is no routing, no server-side
rendering, no database, no secrets.

**`family/`** is a shared design system (CSS tokens + base components + an
"honesty" provenance panel) copied identically across a sibling family of
`*-log` landing-page repos. It is documented in its own
[`family/README.md`](family/README.md). Per this site's plan, the family CSS is
meant to be **inlined at build time** into each site's `<style>` block — and
indeed `public/index.html` already contains that CSS inlined verbatim.

## Run it

Requirements: Node.js and the Wrangler CLI (`npm i -g wrangler` or use `npx`).

```bash
# Local dev server (serves the landing page at http://localhost:8787)
npx wrangler dev

# Production deploy (deploys the Worker + static asset)
npx wrangler deploy
```

✅ Verified: `npx wrangler deploy --dry-run` succeeds for the sibling
`personallog` repo (identical Worker shape) — it reads `public/index.html` as
the single asset and binds `env.ASSETS`. There is **no test suite** in this repo
and no `package.json`; nothing to install beyond Wrangler itself.

## Honesty markers (verified, not assumed)

The org's discipline is that docs must not oversell code. Everything below was
traced against real artifacts, not copied from existing copy.

About **this repo** (the landing page Worker):

- ✅ **Real today:** builds and serves a static landing page via a Cloudflare
  Worker (same shape as its verified siblings). The HTML/CSS is real and
  self-contained.
- 🔮 **Not in this repo:** there is no PLATO integration, no agent logic, no
  player-activity logging code, and no build pipeline beyond Wrangler. The
  `family/README.md` describes a `build.mjs` inlining step as the intended
  pattern, but **no such build script exists in this repo** — the CSS is already
  inlined in `public/index.html` by hand, so the "build step" is currently
  a manual edit.

About the **advertised package** `playerlog-agent` (separate repo/PyPI — I
verified these against PyPI and the upstream GitHub README, not assumed):

- ✅ **Real today:** genuinely published on PyPI as `playerlog-agent` `0.1.0`
  (summary: *"playerlog domain agent for PLATO fleet"*). HTTP 200 on
  `https://pypi.org/pypi/playerlog-agent/json`.
- ⚠️ **Real but thin:** the upstream README (`SuperInstance/playerlog-agent`) is
  the thinnest in the family — a title, a one-line description, an install line,
  and an **empty `Usage` section** (no `Related` links even). So "install it and
  use it" is real, but there is no documented API example. The landing page's
  own banner calls this a **"THIN SEED"** / "placeholder with almost no
  implemented surface yet" — that framing is accurate.
- 🔮 **Not verified here:** whether PLATO itself (the shared tile-storage /
  agent-memory substrate the package writes to) is running, or what the
  package's import surface actually is. That code is not in this repo.

## The sibling family

This repo is one of several near-identical landing-page Workers that each
advertise a different PLATO domain agent and share the `family/` design system:

| repo | domain | advertises | accent |
|---|---|---|---|
| personallog | personallog.ai | `personallog-agent` (personal notes) | `--depth` |
| **playerlog** | playerlog.ai | `playerlog-agent` (game-player activity) | `--antifoul` |
| reallog | reallog.ai | `reallog-agent` (camera scene logging) | `--lattice` |
| studylog | studylog.ai | `studylog-agent` (study-partner CLI) | `--reef` |

Each Worker is identical (`src/index.ts` is byte-for-byte the same); only the
`wrangler.jsonc` `name`, the landing-page HTML content, and the accent color
differ. See `family/README.md` for the shared design-system rationale.
