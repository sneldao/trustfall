# Brands under trustfall.xyz

Each brand is its own Cloudflare Pages project + repo, mapped onto a subdomain
of trustfall.xyz. DNS for trustfall.xyz is managed at **GoDaddy**
(`domaincontrol.com` nameservers); the zone is not on Cloudflare, so every
subdomain is wired with an external CNAME.

## Current map

| Hostname              | Pages project   | CNAME target              | Repo                 |
| --------------------- | --------------- | ------------------------- | -------------------- |
| www.trustfall.xyz     | `trustfall-hub` | `trustfall-hub.pages.dev` | this repo            |
| latep.trustfall.xyz   | `latep`         | `latep.pages.dev`         | thisyearnofear/latep |
| claflin.trustfall.xyz | `claflin`       | `claflin.pages.dev`       | —                    |

The apex `trustfall.xyz` is on GoDaddy forwarding → `https://www.trustfall.xyz`
(GoDaddy can't CNAME an apex). Moving the zone to Cloudflare later would enable
apex-flattened CNAME and Workers routes like `api.<brand>.trustfall.xyz`.

## Checklist: adding a new brand (`<brand>`)

1. **Repo** — new directory + repo for the product (e.g. `~/Dev/<brand>`).
2. **Pages project** — `npx wrangler pages project create <brand>
--production-branch=main` (from a dir wired to the trustfall account — see
   README).
3. **Deploy** — `wrangler pages deploy <dist> --project-name=<brand>` →
   `<brand>.pages.dev`.
4. **Attach domain** — the `pages domain` CLI isn't available in wrangler here;
   use the API (token is in `.config/.wrangler/config/default.toml`):

   ```bash
   curl -X POST -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"name":"<brand>.trustfall.xyz"}' \
     "https://api.cloudflare.com/client/v4/accounts/$ACCOUNT_ID/pages/projects/<brand>/domains"
   ```

5. **DNS (GoDaddy)** — CNAME `<brand>` → `<brand>.pages.dev`. Cloudflare
   verifies + issues SSL automatically within a few minutes of propagation.
6. **Hub card** — add the brand card to `site/index.html` and flip its status
   from `soon` to `live`.

## APIs

Brand APIs run as Workers on `*.workers.dev` (e.g.
`latep-api.papaandthejimjams.workers.dev`) — custom hostnames like
`api.<brand>.trustfall.xyz` require the trustfall.xyz zone on this Cloudflare
account.
