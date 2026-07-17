# VibePresto Skill

Agent skill for building, verifying, and uploading trusted static page bundles to WordPress with the published VibePresto CLI.

## Install

Install the skill directly from its public GitHub repository:

```bash
npx skills add vibepresto/skill --skill vibepresto
```

The skill uses `npx vibepresto`, so the CLI does not need to be installed globally. A WordPress site with the compatible VibePresto plugin is required for login and uploads.

## Verify

List installed skills and confirm that `vibepresto` is present:

```bash
npx skills list
```

## Update

Update an existing installation from the public repository:

```bash
npx skills update vibepresto
```

See [`SKILL.md`](./SKILL.md) for the workflow provided to agents.
