# Changelog

## 0.1.4

- Synced compatibility version with VibePresto plugin and CLI 0.1.4.
- Keeps deployment guidance aligned with the current editor preview, page activation, and plugin compatibility workflow.
- Added guidance for `pages set-vibepresto` activation, existing bundle version selection, and page-level plugin hook compatibility mode.
- Clarified that plugin hook compatibility includes plugin-owned markup, scripts, and styles while demoting WordPress CSS below bundle CSS with cascade layers.

## 0.1.2

- Added `skill.json` as the canonical machine-readable skill version source.
- Documented plugin-driven compatibility checks and upgrade guidance for the published skill.
- Expanded the skill from simple static uploads to framework-aware static deployment workflows.
- Added guidance for route-manifest deployments, dry runs, and multi-page page-mapping flows.
- Standardized the skill around the published CLI as the primary automation surface.
