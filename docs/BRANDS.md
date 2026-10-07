# trustfall project map

Each trustfall project is independently deployed. The hub links to the current public URL for each project.

| Project | Public URL                                              | Repository                                                        | Host             |
| ------- | ------------------------------------------------------- | ----------------------------------------------------------------- | ---------------- |
| latep   | [latep.trustfall.xyz](https://latep.trustfall.xyz)      | [thisyearnofear/latep](https://github.com/thisyearnofear/latep)   | Cloudflare Pages |
| chime   | [chime.trustfall.xyz](https://chime.trustfall.xyz)      | [thisyearnofear/chime](https://github.com/thisyearnofear/chime)   | Cloudflare Pages |
| bothy   | [bothy.trustfall.xyz](https://bothy.trustfall.xyz/)     | [thisyearnofear/bothy](https://github.com/thisyearnofear/bothy)   | Self-hosted      |
| elcaro  | [elcaro.trustfall.xyz](https://elcaro.trustfall.xyz/)   | [udirobert/elcaro](https://github.com/udirobert/elcaro)           | Cloudflare Pages |
| srelok  | [srelok.netlify.app](https://srelok.netlify.app/)       | [sneldao/srelok](https://github.com/sneldao/srelok)               | Netlify          |
| clawdy  | [clawdy.trustfall.xyz](https://clawdy.trustfall.xyz/)   | [thisyearnofear/clawdy](https://github.com/thisyearnofear/clawdy) | Vercel           |
| grunds  | [grunds.trustfall.xyz](https://grunds.trustfall.xyz/)   | [sneldao/grunds](https://github.com/sneldao/grunds)               | Vercel           |
| claflin | [claflin.trustfall.xyz](https://claflin.trustfall.xyz/) | [sneldao/claflin](https://github.com/sneldao/claflin)             | Vercel           |

## Add a project

1. Deploy the project and confirm its public URL.
2. If using `*.trustfall.xyz`, configure the custom domain in its host and add the required DNS record.
3. Add the project’s `name`, short `kicker`, concise `description`, `href`, and three-color `palette` to `projects` in `src/routes/+page.svelte`.
4. Add a card clip to `static/clips/` and point the project’s `video` at it, or omit `video` to use the generated palette artwork.
5. Add the project to the tables in `README.md` and this document.
6. Run `npm run build` and verify the entry opens the intended site.

The infinite field automatically repeats every project card and handles spherical placement, dragging, glass treatment, and linking. Keep card copy short enough to remain legible inside the generated artwork. Use a browser with WebGPU for the full material treatment; WebGL remains the supported fallback.

## Card video

Every shipped clip is committed to `static/clips/` — not Git LFS, not object storage. The build must stay self-contained and the field must not depend on a third party being up. Target the house spec: **1280×720, 24fps, exactly 6.000s, silent, under ~3 MB.**

Generated with the ElevenLabs Image & Video API (`ELEVENLABS_API_KEY`):

1. **Storyboard stills first.** `POST /v1/flows/image` with `gemini-3.1-flash-image`, `aspect_ratio: "16:9"`, `resolution: "2K"`. Two options per project. Stills are cheap; review them before spending anything on video.
2. **Then the loop.** `POST /v1/flows/video` with `veo-3.1-generate-001`, `duration_secs: 6`, `resolution: "720p"`, `generate_audio: false`, a fixed `seed`, and a `negative_prompt` covering text, logos, faces and cuts. Pass the approved still as a reference by `{"type": "generation", "generation_id": …}` — no upload, and it may still be queued.
3. **Poll** `GET /v1/flows/video/{generation_id}` and download immediately: `content_url` is signed and expires in about an hour.

Set `start_frame` and `end_frame` to the **same** still. The clip then returns to its first frame and loops seamlessly on the card; measure it with a first-vs-last-frame PSNR over ~30 dB. Veo’s fixed 4/6/8s durations mean 6s needs no re-encode.

Gotchas worth remembering:

- Veo 3 rejects `enhance_prompt: false` with a `model_error` — omit the field and accept server-side prompt rewriting. The failed generation is not charged.
- `GET /v1/flows/image` list responses wrap results in `generations`, not `items`.
- Python on macOS may lack a CA store; use `httpx` with `certifi` rather than `urllib` for downloads.

## Provenance

`assets/media_assets.json` is the ledger. Every clip records its `generation_id`, model, full prompt, parameters, seed, and the storyboard still it was chained from, so a card can be re-rendered exactly without archaeology.

The 2K storyboard stills are **not** in the repo — they are pinned to [Grove](https://lens.xyz/docs/storage/usage/upload) with an immutable ACL on Lens Chain mainnet (`chain_id: 232`, never a testnet id, whose retention policy deletes content). Pinning is two unauthenticated calls: `POST https://api.grove.storage/link/new`, then a multipart `POST /{storage_key}` carrying the file and `lens-acl.json`. The `lens://` URI, gateway URL, SHA-256 prefix and byte count live in the ledger, verified byte-identical through the gateway after pinning. Immutable means the pin can never be edited or deleted, so nothing unpublished goes in.

Video files stay in git. Grove is a store for the source artifacts the repo has no reason to carry, not a CDN in front of one you already have.
