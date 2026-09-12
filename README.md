# trustfall

The hub for [trustfall.xyz](https://www.trustfall.xyz) — a family of small
interactive experiments about trust, cooperation, and cryptography.

## Brands

| Site                                               | Project                     | Repo                                                            | Status |
| -------------------------------------------------- | --------------------------- | --------------------------------------------------------------- | ------ |
| [www.trustfall.xyz](https://www.trustfall.xyz)     | `trustfall-hub` (this repo) | —                                                               | live   |
| [latep.trustfall.xyz](https://latep.trustfall.xyz) | `latep`                     | [thisyearnofear/latep](https://github.com/thisyearnofear/latep) | live   |
| claflin.trustfall.xyz                              | hosted on Vercel            | —                                                               | soon   |

See [docs/BRANDS.md](docs/BRANDS.md) for the DNS architecture and how to add
a new brand.

## Structure

```
site/       → the static hub page (deployed to Cloudflare Pages)
docs/       → public documentation (committed)
private/    → private notes (gitignored — never committed)
```

Anything under `private/` or matching `*.private.*` is gitignored — use it for
internal notes, plans, and drafts that shouldn't be published.

## Deploy

```bash
npm install        # one-time: sets up the pre-commit hooks
npm run deploy     # wrangler pages deploy → trustfall-hub.pages.dev + www.trustfall.xyz
```

### Cloudflare account

Wrangler in this repo is pinned to the shared trustfall Cloudflare account
(`papaandthejimjams@gmail.com`) via `XDG_CONFIG_HOME=".config"` in `.env`
(gitignored). Credentials live in `.config/.wrangler/` — run `npx wrangler
login` from this directory to re-authenticate if they expire.

### DNS

Currently at GoDaddy (`www` CNAME → `trustfall-hub.pages.dev`); migrating to
Cloudflare so the apex serves the hub directly — see
[docs/BRANDS.md](docs/BRANDS.md).

## Pre-commit

Husky runs on every commit:

- `secretlint "**/*"` — blocks committing secrets/keys
- `lint-staged` — prettier on staged files
