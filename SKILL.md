---
name: vibepresto
description: Upload static HTML/CSS/JS page bundles or framework-exported static page builds to VibePresto on WordPress using the published CLI. Use when a user wants to log in, inspect pages or posts, build and verify a frontend app, upload a page bundle, assign it to a WordPress page, manage bundle history, or use `data-vp-*` placeholders for values from the current queried `WP_Post` object. Do not use for wp-admin browser automation, direct REST calls when the CLI covers the task, SSR hosting, or route-aware deployment unless the user explicitly asks for it.
---

# VibePresto Upload

Use this skill to upload a local static page or static-exported frontend page into the VibePresto WordPress plugin through the CLI.

Default MVP model: **one WordPress page = one bundle lineage = one assignment**. If the user asks for Home, About, Terms, or similar multi-page site work, split the work into separate page bundles and upload each one to its matching WordPress page. Do not package the whole site into one ZIP or use `deploy` unless the user explicitly requests route-aware/multi-page deployment.

## CLI invocation

Always prefer the published CLI:

```
npx vibepresto
```

If the CLI repo is checked out locally for development, `node ./bin/vibepresto.js` is also acceptable. Do not fall back to browser automation or direct REST calls unless the CLI is clearly blocked or the user explicitly requests lower-level debugging.

## Version compatibility

`skill.json` is the canonical version source for this skill. `agents/openai.yaml` is OpenAI/Codex UI metadata only — do not use it to determine the installed skill version.

## Recommended workflow

1. **Confirm auth and check version compatibility**
   - `npx vibepresto whoami --site <site> --json`
   - If not logged in: `npx vibepresto login --site <site>`
   - From the `whoami` response, compare `data.compatibility.skill.minimum_version` against the version in `skill.json`. If the plugin requires a newer skill, warn the user: `npx skills add vibepresto/skill`

2. **Choose a deployment path**
   - Plain static folder with `index.html` at root → use `upload --site-dir --page-id <id>`
   - Plain static folder or dist folder with a nested entry → add `--entry-html <relative-html>`
   - Framework project or prebuilt dist folder for one page → use `detect`/`build`/`verify`, then `upload --site-dir <dist> --page-id <id>`
   - Route-aware multi-page deployment → advanced only; use `routes inspect` and `deploy` only when the user explicitly asks for it or an existing deployment requires it

3. **Framework or static-export projects**
   - `npx vibepresto detect --project-dir <dir> --json`
   - `npx vibepresto build --project-dir <dir> --json` (or `verify --output-dir <dir>` if already built)
   - `npx vibepresto verify --output-dir <dir> --json`
   - Check `placeholder_count`, `placeholders`, and `warnings` when the HTML uses `data-vp-*` attributes
   - If `.vibepresto/config.json` exists, prefer its saved `site`, `projectDir`, `outputDir`, `defaults.entryHtml`, `uploadTarget`, `deployment.targets[]`, and `singlePostTemplate.lineageId` defaults unless the user overrides them
   - Auth sessions live in workspace-local `.vibepresto/auth.json`; when assets are outside the logged-in workspace, pass `--workspace-dir <workspace>`

4. **Upload one page bundle**
   - `npx vibepresto upload --site <site> --site-dir <dir> --page-id <id> --json`
   - Repeat this for each WordPress page in a multi-page site request

5. **Advanced dry run before a route-aware deployment on an unfamiliar project**
   - `npx vibepresto deploy --site <site> --output-dir <dir> --dry-run --json`

6. **Advanced multi-route or router-based apps**
   - `npx vibepresto deploy --site <site> --output-dir <dir> --create-missing-pages --json`

7. **Bundle and advanced deployment history**
   - `npx vibepresto bundles list --site <site> --json`
   - `npx vibepresto bundles versions --site <site> --bundle-id <id> --json`
   - `npx vibepresto bundles rollback --site <site> --page-id <id> --version <n> --json`
   - `npx vibepresto deployments list --site <site> --json`
   - `npx vibepresto deployments show --site <site> --deployment-id <id> --json`
   - `npx vibepresto deployments promote --site <site> --deployment-id <id> --bundle-version-id <id> --json`
   - `npx vibepresto deployments rollback --site <site> --deployment-id <id> --version <n> --json`

8. **WordPress Posts page (blog index)**
   - `npx vibepresto pages set-posts-page --site <site> --page-id <id> --json`
   - Targets the WordPress `Posts page` from Reading Settings, not individual single-post views

9. **Pause/resume a VibePresto page takeover**
   - `npx vibepresto pages set-vibepresto --site <site> --page-id <id> --active --json`
   - `npx vibepresto pages set-vibepresto --site <site> --page-id <id> --inactive --json`
   - Add `--bundle-version-id <id>` to switch to a specific bundle version
   - Add `--plugin-hooks-mode inherit|enabled|disabled` to control hook compatibility

10. **Single-post permalink takeover**
   - One specific post: `npx vibepresto upload --site <site> --site-dir <dir> --post-id <id> --json`
   - Fallback template across all single posts: `npx vibepresto posts set-default-template --site <site> --lineage-id <id> --json`

11. Use `--json` whenever output will be parsed or used by a subsequent step.

## Upload vs. Advanced Deploy

**Simple upload** (`upload --site-dir`) — plain HTML/CSS/JS folder with `index.html` at root:

```bash
npx vibepresto upload \
  --site <site> \
  --site-dir ./site-folder \
  --name "Landing page" \
  --page-id 2 \
  --json
```

- CLI validates local references and `data-vp-*` placeholders before uploading.
- If `.vibepresto/config.json` defines `uploadTarget`, `--page-id`/`--post-id` can be omitted.
- `upload --page-id` assigns the bundle to that page but does not change the WordPress homepage. Use `pages set-homepage` only when the homepage should intentionally change.

For a non-root entry:

```bash
npx vibepresto upload \
  --site <site> \
  --site-dir ./dist \
  --entry-html nested/app.html \
  --page-id 2 \
  --json
```

**Advanced framework/static-export deploy** (`deploy`) — route-aware compatibility path for React, Next, Nuxt, Vite, Svelte, TanStack, or any app producing static output that must intentionally map multiple routes from one bundle:

```bash
npx vibepresto deploy \
  --site <site> \
  --project-dir ./my-app \
  --create-missing-pages \
  --json
```

Uses `deployment.targets[]` from `.vibepresto/config.json` when present; otherwise resolves existing WordPress pages and optionally creates missing ones.

For ordinary generated sites with Home/About/Terms pages, use separate `upload --page-id` commands instead of `deploy`.

## `data-vp-*` placeholders

`data-vp-source="post"` refers to the current queried `WP_Post` object. In WordPress, both the `post` and `page` post types are represented by `WP_Post`. Example:

```html
<h1 data-vp-source="post" data-vp-field="post_title">Fallback title</h1>
<p data-vp-source="post" data-vp-field="post_excerpt">Fallback excerpt</p>
```

## Output format

All commands return:

- Success: `{ ok: true, data: { ... } }`
- Failure: `{ ok: false, error: { code, message, details } }`

`whoami` additionally returns `data.compatibility.skill.minimum_version` for version checking (see step 1).

## Non-goals

- No SSR hosting. Next.js, Nuxt, SvelteKit, and TanStack server runtimes do not run inside WordPress; all supported framework flows must produce static/exported HTML plus assets.
- No wp-admin automation beyond the human approval step during device login.
