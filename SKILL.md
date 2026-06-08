---
name: vibepresto
description: Deploy static HTML/CSS/JS sites or framework-exported static builds to VibePresto on WordPress using the published CLI. Use when a user wants to log in, inspect pages or posts, build and verify a frontend app, inspect routes, upload a bundle, create a multi-page deployment, manage bundle/deployment history, or use `data-vp-*` placeholders for values from the current queried `WP_Post` object. Do not use for wp-admin browser automation, direct REST calls when the CLI covers the task, or SSR hosting.
---

# VibePresto Deploy

Use this skill when the task is to deploy a local static site or static-exported frontend build into the VibePresto WordPress plugin through the supported CLI surface.

## What this skill covers

- Device-style CLI login
- Session inspection with `whoami`
- Page listing, search, creation, status changes, Front page assignment, and Posts page assignment
- Page-level VibePresto activation, selected bundle version changes, and plugin hook compatibility mode
- Post listing, search, status changes, direct bundle assignment, and default single-post template assignment
- Framework-aware `detect`, `build`, `verify`, and `routes inspect`
- Auto-bundling a local static site folder
- Uploading an existing ZIP bundle
- Uploading explicit HTML/CSS/JS/assets files
- Multi-route deployments with existing-page resolution and optional page creation
- Bundle lineage listing, version history, and rollback
- Deployment listing, inspection, promotion, and rollback
- Logging out or revoking a saved session
- Validation of WordPress placeholder markup such as `data-vp-source="post"` and `data-vp-field="post_title"`

## Use the CLI, not wp-admin automation

Prefer the published CLI:

`npx vibepresto`

If the CLI repo is checked out locally for development, `node ./bin/vibepresto.js` from that repo is also acceptable.

Do not fall back to browser automation or direct REST calls unless the CLI is clearly blocked or the user explicitly asks for lower-level debugging.

## Version compatibility

Treat `skill.json` as the canonical machine-readable version source for this skill.

The `agents/openai.yaml` file is only OpenAI/Codex UI metadata. It is not the compatibility manifest and should not be used to determine the installed skill version.

Claude-style subagents are a separate integration model and are not represented by `agents/openai.yaml` in this repository.

## Recommended workflow

1. Confirm or establish auth:
   - `npx vibepresto whoami --site <site> --json`
   - if not logged in: `npx vibepresto login --site <site>`
   - after `whoami`, inspect `data.compatibility.skill.minimum_version`
   - read this skill's local version from `skill.json`
   - if the plugin minimum skill version is newer than the installed skill version, warn the user and suggest: `npx skills add vibepresto/skill`
2. Decide whether this is a simple static upload or a framework/static-export deployment:
   - simple static folder: use `upload --site-dir`
   - frontend project or prebuilt dist folder: use `detect`, `build` or `verify`, then `routes inspect`, then `deploy`
3. For framework or export-based projects:
   - `npx vibepresto detect --project-dir <dir> --json`
   - `npx vibepresto build --project-dir <dir> --json`
   - or if already built: `npx vibepresto verify --output-dir <dir> --json`
   - `npx vibepresto routes inspect --output-dir <dir> --json`
   - check `placeholder_count`, `placeholders`, and `warnings` in JSON output when the HTML uses `data-vp-*`
   - if `.vibepresto/config.json` exists, prefer its saved `site`, `projectDir`, `outputDir`, `uploadTarget`, `deployment.targets[]`, and `singlePostTemplate.lineageId` defaults unless the user explicitly overrides them
4. Before first deployment on an unfamiliar project, prefer a dry run:
   - `npx vibepresto deploy --site <site> --output-dir <dir> --dry-run --json`
5. For multi-page or router-based apps, prefer route-manifest deployment:
   - `npx vibepresto deploy --site <site> --output-dir <dir> --create-missing-pages --json`
6. Use bundle/deployment history commands when needed:
   - `npx vibepresto bundles list --site <site> --json`
   - `npx vibepresto bundles versions --site <site> --bundle-id <id> --json`
   - `npx vibepresto bundles rollback --site <site> --page-id <id> --version <n> --json`
   - `npx vibepresto deployments list --site <site> --json`
   - `npx vibepresto deployments show --site <site> --deployment-id <id> --json`
   - `npx vibepresto deployments promote --site <site> --deployment-id <id> --bundle-version-id <id> --json`
   - `npx vibepresto deployments rollback --site <site> --deployment-id <id> --version <n> --json`
7. When the user wants a static template for the WordPress blog index, use:
   - `npx vibepresto pages set-posts-page --site <site> --page-id <id> --json`
   - this targets the WordPress `Posts page` from Reading Settings, not every individual single post view
8. When the user wants to pause or resume a VibePresto takeover while preserving the selected bundle, use:
   - `npx vibepresto pages set-vibepresto --site <site> --page-id <id> --active --json`
   - `npx vibepresto pages set-vibepresto --site <site> --page-id <id> --inactive --json`
   - to switch to an existing bundle version, add `--bundle-version-id <id>`
   - to control plugin hook compatibility for that page, add `--plugin-hooks-mode inherit|enabled|disabled`
9. When the user wants a single post permalink takeover:
   - for one specific post: `npx vibepresto upload --site <site> --site-dir <dir> --post-id <id> --json`
   - for the fallback template used across single posts: `npx vibepresto posts set-default-template --site <site> --lineage-id <id> --json`
10. Prefer `--json` whenever the result needs to be parsed or used by another tool step.

## Upload and deploy modes

### Simple single-page upload

Use when the user already has a plain HTML/CSS/JS site folder with `index.html` at the root.

Example:

```bash
npx vibepresto upload \
  --site <site> \
  --site-dir ./site-folder \
  --name "Landing page" \
  --page-id 2 \
  --json
```

Rules:

- `index.html` must exist at the folder root.
- The CLI validates local references before upload.
- The CLI also validates `data-vp-*` placeholders and reports non-blocking warnings for unsupported source, field, or target usage.
- If `.vibepresto/config.json` defines `uploadTarget`, the skill can omit `--page-id` or `--post-id` and let the CLI resolve them from the active environment.
- Existing single-page uploads remain valid and should still be used when they fit the task.

Example placeholder markup:

```html
<article>
  <h1 data-vp-source="post" data-vp-field="post_title">Fallback title</h1>
  <p data-vp-source="post" data-vp-field="post_excerpt">Fallback excerpt</p>
</article>
```

Terminology note:

- `data-vp-source="post"` refers to the current queried `WP_Post` object.
- In WordPress, both the `post` and `page` post types are represented by `WP_Post`.

### Framework/static-export deployment

Use when the user has a React/Next/Nuxt/Vite/Svelte/TanStack style app that produces static output.

Example:

```bash
npx vibepresto deploy \
  --site <site> \
  --project-dir ./my-app \
  --create-missing-pages \
  --json
```

This flow should:

- detect the project type
- run the static build/export locally
- verify output integrity
- inspect or infer routes
- use `deployment.targets[]` from `.vibepresto/config.json` when present for the active environment
- otherwise resolve existing WordPress pages and optionally create missing pages
- upload the bundle and create/update a deployment

### Prebuilt output directory

Use when the project is already built:

```bash
npx vibepresto deploy \
  --site <site> \
  --output-dir ./dist \
  --dry-run \
  --json
```

## Output handling

- Prefer `--json` for agentic flows.
- Expect:
  - `ok: true` with `data` on success
  - `ok: false` with `error.code`, `error.message`, and `error.details` on failure
- `whoami` also returns plugin compatibility metadata. Use that to warn when the installed skill version in `skill.json` is below `data.compatibility.skill.minimum_version`.
- Use JSON mode when you need page IDs, bundle version IDs, route manifests, deployment IDs, target mappings, or dry-run planning output.

## Non-goals

- Do not use this skill for SSR hosting.
- Do not promise that Next.js, Nuxt, SvelteKit, or TanStack server runtimes will run inside WordPress.
- Supported framework flows must end in static/exported HTML plus assets.
- Do not use wp-admin automation except for the human approval step during device login when needed.
