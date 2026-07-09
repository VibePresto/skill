# VibePresto Skill

Codex skill for uploading single-page static bundles and framework-exported static page builds to a VibePresto-enabled WordPress site.

## Install

```bash
npx skills add vibepresto/skill
```

## What it does

- prefers the published VibePresto CLI
- checks authentication before upload or deploy
- checks plugin-declared compatibility metadata and can suggest a skill upgrade
- uses framework-aware `detect`, `build`, and `verify` flows before upload
- treats route-manifest and multi-page deployments as advanced compatibility features
- can use optional project-local defaults from `.vibepresto/config.json`
- uses workspace-local auth from `.vibepresto/auth.json`
- supports `--entry-html` when the main HTML file is not root `index.html`
- can mark a page as the WordPress Front page or Posts page through the CLI
- can assign bundles to single posts and set a default single-post template lineage
- validates WordPress `data-vp-*` placeholders in uploaded HTML
- uses JSON output for agentic workflows

## Typical workflow

```bash
npx vibepresto whoami --site https://your-site.example --json
npx vibepresto build --project-dir ./my-app --json
npx vibepresto verify --output-dir ./my-app/dist --json
npx vibepresto upload --site https://your-site.example --site-dir ./my-app/dist --page-id 123 --json
```

Default MVP model: **one WordPress page = one bundle lineage = one assignment**. For Home/About/Terms work, the skill should create or select each WordPress page and upload a separate page bundle for each one. Use route-aware `deploy` only when the user explicitly asks for one bundle mapped across multiple routes or when maintaining an existing multi-route deployment.

The canonical machine-readable version for the skill lives in [`skill.json`](./skill.json).

The [`agents/openai.yaml`](./agents/openai.yaml) file is OpenAI/Codex-specific UI metadata only. It is not the shared compatibility manifest, and it does not model Claude-style subagents.

If the project already has `.vibepresto/config.json`, the skill should prefer its saved environment defaults for:

- `site`
- `projectDir` and `outputDir`
- `defaults.entryHtml`
- `uploadTarget`
- `deployment.targets[]` for advanced route-aware deployments
- `singlePostTemplate.lineageId`

When deploying assets outside the logged-in workspace, pass `--workspace-dir <workspace>` so the CLI reads the intended config and auth files.

For a simple static page upload, the skill can still use:

```bash
npx vibepresto upload --site https://your-site.example --site-dir ./landing-page --page-id 123 --json
```

This assigns the bundle only; it does not change WordPress Reading Settings. Use `pages set-homepage` only when the homepage should intentionally change.

For nested entries:

```bash
npx vibepresto upload --site https://your-site.example --site-dir ./dist --entry-html nested/app.html --page-id 123 --json
```

For a single post, the skill can also use:

```bash
npx vibepresto upload --site https://your-site.example --site-dir ./post-template --post-id 789 --json
npx vibepresto posts set-default-template --site https://your-site.example --lineage-id 12 --json
```

Placeholder example:

```html
<h1 data-vp-source="post" data-vp-field="post_title">Fallback title</h1>
```

This `post` source maps to the current queried `WP_Post` object, which may be a `page` or `post` post type depending on the request.

The full skill definition lives in [SKILL.md](./SKILL.md).
