---
name: vibepresto
description: Upload trusted single-page static HTML/CSS/JS bundles to the VibePresto WordPress plugin with the published config-backed CLI. Use when a user wants to initialize VibePresto project config, log in, build or verify a static frontend output, upload and assign one bundle to one WordPress page, inspect bundle history, roll back a page bundle, or use WordPress `data-vp-*` placeholders. Do not use for direct REST calls, wp-admin browser automation, SSR hosting, or route-aware deployment unless the user explicitly asks for debugging or advanced compatibility.
---

# VibePresto

Prefer `npx vibepresto`. Use `node ./bin/vibepresto.js` only inside the local CLI repo. Use `--json` for agent workflows.

## Default Model

One WordPress page = one bundle lineage = one active assignment. For Home/About/Terms or similar multi-page work, upload one bundle per WordPress page. Do not upload one whole-site ZIP or use `deploy` unless the user explicitly asks for route-aware deployment.

## Config-First Workflow

Inspect existing defaults:

```bash
npx vibepresto config show --json
```

If there is no config, initialize it once:

```bash
npx vibepresto init --site <url> --project-dir <dir> --output-dir <dir> --page-id <id> --env local --json
```

Log in or confirm auth:

```bash
npx vibepresto login
npx vibepresto whoami --json
```

Use config and auth defaults before asking for a site URL. Use `--workspace-dir <workspace>` for external assets, and `--site <url>` only to override config or when no config exists.

## Build, Verify, Upload

Framework/static-export project:

```bash
npx vibepresto build --json
npx vibepresto verify --json
npx vibepresto upload --site-dir <dist-or-site-dir> --json
```

Already-built folder: `npx vibepresto verify --output-dir <dir> --json`, then `npx vibepresto upload --site-dir <dir> --json`. If config has no `uploadTarget`, add `--page-id <id>` or `--post-id <id>`. If entry HTML is not root `index.html`, add `--entry-html <relative-html>` or save it in config.

Check `placeholder_count`, `placeholders`, and `warnings` for `data-vp-*`. The CLI validates local bundle safety before upload. Uploaded JS is trusted admin content; VibePresto rejects server-executable files but does not sanitize admin-provided JS.

## Bundle History

```bash
npx vibepresto bundles list --json
npx vibepresto bundles versions --bundle-id <id> --json
npx vibepresto bundles rollback --page-id <id> --version <n> --json
```

## Advanced Only

Use only when the user explicitly asks:

- route-aware multi-page deployment: `npx vibepresto deploy --output-dir <dir> --create-missing-pages --json`
- deployment history: `deployments list|show|promote|rollback`
- page management: `pages create|set-status|set-homepage|set-posts-page|set-vibepresto`
- single-post assignment: `upload --post-id <id>`
- default single-post template: `posts set-default-template --lineage-id <id>`

## Placeholders

`data-vp-source="post"` means the current queried `WP_Post` object, which may be a page or post.

```html
<h1 data-vp-source="post" data-vp-field="post_title">Fallback title</h1>
```

## Compatibility

`skill.json` is the skill version source. If `whoami` reports `data.compatibility.skill.minimum_version` greater than this skill version, warn the user to update the skill.
