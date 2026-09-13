# trustfall

[trustfall.xyz](https://www.trustfall.xyz) is a showcase for small interactive experiments in trust, cooperation, AI, and cryptography.

## Projects

| Project   | Live site                                             | Repository                                                      |
| --------- | ----------------------------------------------------- | --------------------------------------------------------------- |
| trustfall | [trustfall.xyz](https://www.trustfall.xyz)            | this repository                                                 |
| latep     | [latep.trustfall.xyz](https://latep.trustfall.xyz)    | [thisyearnofear/latep](https://github.com/thisyearnofear/latep) |
| chime     | [chime.trustfall.xyz](https://chime.trustfall.xyz)    | [thisyearnofear/chime](https://github.com/thisyearnofear/chime) |
| bothy     | [bothyapp.netlify.app](https://bothyapp.netlify.app/) | [thisyearnofear/bothy](https://github.com/thisyearnofear/bothy) |
| elcaro    | [elcaro.trustfall.xyz](https://elcaro.trustfall.xyz/) | [udirobert/elcaro](https://github.com/udirobert/elcaro)         |
| srelok    | [srelok.netlify.app](https://srelok.netlify.app/)     | [sneldao/srelok](https://github.com/sneldao/srelok)             |

## Stack

The hub is a statically prerendered SvelteKit showcase. Its full-screen project field uses Three.js, prefers WebGPU/TSL for liquid-glass materials, and falls back to WebGL when needed. Cards wrap infinitely across a spherical field, respond to inertial mouse/touch dragging, and link directly to each project.

- `src/routes/+page.svelte` — project data and minimal showcase chrome
- `src/lib/ShowcaseScene.svelte` — spherical grid, card artwork, interaction, and materials
- `docs/BRANDS.md` — adding and publishing projects

## Development

```bash
npm install
npm run dev
npm run build
npm run preview
```

`npm run build` writes the deployable static site to `build/`.

## Deploy

```bash
npm run deploy
```

The deploy command builds the site and publishes `build/` to the `trustfall-hub` Cloudflare Pages project.

## Checks

Pre-commit hooks run Secretlint and Prettier on staged files. Keep credentials and private notes out of the repository; `.env`, `.config/`, and `private/` are ignored.
