# Brands under trustfall.xyz

Each brand is its own deploy target + repo, mapped onto a subdomain of
trustfall.xyz.

## Target architecture

DNS for trustfall.xyz is **moving from GoDaddy to Cloudflare** (account:
papaandthejimjams). The domain stays registered at GoDaddy; only the
nameservers change. Once migrated:

- `trustfall.xyz` apex serves the hub directly — Cloudflare does CNAME
  flattening, no redirect hop.
- `www` 301-redirects → apex via a Redirect Rule (or serves directly).
- Every brand gets `<brand>.trustfall.xyz` as a CNAME in the CF zone.
- Brand APIs can use Workers custom domains (`api.<brand>.trustfall.xyz`)
  since the zone will be on the same account as the Workers.

| Hostname              | Target                                       | Proxy        | Serves           |
| --------------------- | -------------------------------------------- | ------------ | ---------------- |
| trustfall.xyz         | `trustfall-hub.pages.dev` (CNAME, flattened) | proxied      | this hub         |
| www.trustfall.xyz     | `trustfall-hub.pages.dev`                    | proxied      | this hub         |
| latep.trustfall.xyz   | `latep.pages.dev`                            | proxied      | Latep app        |
| claflin.trustfall.xyz | `*.vercel-dns-017.com`                       | **DNS only** | Claflin (Vercel) |

> Vendor CNAMEs (Vercel, etc.) must be **DNS-only** (grey cloud) — proxying
> through Cloudflare breaks their domain verification and SSL issuance.

## Migration checklist (GoDaddy → Cloudflare)

1. CF dashboard (papaandthejimjams) → Add site → `trustfall.xyz` → Free.
2. Recreate all records (table above) before switching — nothing else exists
   in the zone (no MX/TXT; verified 2026-09-12).
3. GoDaddy → change nameservers to the pair Cloudflare assigns.
4. After activation: attach `trustfall.xyz` to the `trustfall-hub` Pages
   project (same-account zone → instant verification).
5. Add a Redirect Rule: `www.trustfall.xyz` → `https://trustfall.xyz` (301)
   so the apex is canonical.

## Current state (pre-migration)

Until the NS switch, DNS stays at GoDaddy (`domaincontrol.com`) and each
hostname is an external CNAME there. The apex sits on GoDaddy parking —
either leave it or set GoDaddy forwarding → `https://www.trustfall.xyz` as a
stopgap.

## Checklist: adding a new brand (`<brand>`)

Post-migration:

1. **Repo** — new directory + repo (e.g. `~/Dev/<brand>`).
2. **Pages project** — `wrangler pages project create <brand>
--production-branch=main` (run from a repo wired to the trustfall CF
   account — see README).
3. **Deploy** — `wrangler pages deploy <dist> --project-name=<brand>` →
   `<brand>.pages.dev`.
4. **DNS** — CF dashboard or API: CNAME `<brand>` → `<brand>.pages.dev`
   (proxied; DNS-only if pointing at an external vendor like Vercel).
5. **Attach domain** — `POST /pages/projects/<brand>/domains` with
   `{"name":"<brand>.trustfall.xyz"}`; same-account zone verifies instantly.
6. **Hub card** — add/flip the card in `site/index.html` from `soon` to `live`.

## APIs

Brand APIs run as Workers (`<brand>-api.papaandthejimjams.workers.dev`).
After the zone migration, `api.<brand>.trustfall.xyz` routes can be enabled
in each brand's `wrangler.toml`.
