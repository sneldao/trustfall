# trustfall project map

Each trustfall project is independently deployed. The hub links to the current public URL for each project.

| Project | Public URL                                            | Repository                                                      | Host             |
| ------- | ----------------------------------------------------- | --------------------------------------------------------------- | ---------------- |
| latep   | [latep.trustfall.xyz](https://latep.trustfall.xyz)    | [thisyearnofear/latep](https://github.com/thisyearnofear/latep) | Cloudflare Pages |
| chime   | [chime.trustfall.xyz](https://chime.trustfall.xyz)    | [thisyearnofear/chime](https://github.com/thisyearnofear/chime) | Cloudflare Pages |
| bothy   | [bothyapp.netlify.app](https://bothyapp.netlify.app/) | [thisyearnofear/bothy](https://github.com/thisyearnofear/bothy) | Netlify          |
| elcaro  | [elcaro.trustfall.xyz](https://elcaro.trustfall.xyz/) | [udirobert/elcaro](https://github.com/udirobert/elcaro)         | Cloudflare Pages |
| srelok  | [srelok.netlify.app](https://srelok.netlify.app/)     | [sneldao/srelok](https://github.com/sneldao/srelok)             | Netlify          |

## Add a project

1. Deploy the project and confirm its public URL.
2. If using `*.trustfall.xyz`, configure the custom domain in its host and add the required DNS record.
3. Add a `name`, concise `description`, `tags`, and `href` entry to `experiments` in `src/routes/+page.svelte`.
4. Add the project to the table in `README.md` and this document.
5. Run `npm run build` and verify the entry opens the intended site.

The index automatically renders the entry. Add a corresponding featured-panel mapping in `src/lib/ShowcaseScene.svelte` when the project should receive hover/focus feedback in the interactive field.
