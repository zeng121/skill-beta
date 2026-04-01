# Changelog

## v1.0.8 - 2026-04-01
- Align `SKILL.md` frontmatter with ClawHub's documented skill format by using a top-level `version` field
- Declare `metadata.openclaw` runtime requirements for `FOTOR_OPENAPI_KEY`, `uv`, local config files, and homepage metadata

## v1.0.7 - 2026-03-31
- Include `creditsIncrement` in `scripts/run_task.py` output so each task result reports consumed credits
- Cover account credit lookup in `SKILL.md`, including `get_credits_sync()` usage and the returned `businessId` / `total` / `remaining` fields

## v1.0.6 - 2026-03-31
- Recompute `update_available` from cached `latest_version` and the current local version to avoid stale update reminders after upgrading

## v1.0.5 - 2026-03-31
- Refine the update-check workflow so it runs silently after `ensure_sdk.py` and is defined only in the workflow section
- Clarify that `check_skill_update.py` handles cached state, 24-hour rechecks, and duplicate reminder suppression internally

## v1.0.4 - 2026-03-31
- Sync `skills/fotor-skills/SKILL.md` metadata version to `1.0.4` to match the current ClawHub release version

## v1.0.3 - 2026-03-31
- Align image workflow guidance to default `aspect_ratio="1:1"`
- Remove image size calculation details from references because the SDK resolves image dimensions internally

## v1.0.1 - 2026-03-30
- Add install-source-aware update checking for both ClawHub and GitHub / `npx skills`
- Add concise non-pushy upgrade guidance via `references/install-or-upgrade.md`
- Use repository `CHANGELOG.md` as the GitHub-side update summary source
- Prefer a local `.venv` for bundled Python scripts and document the workflow clearly
- Refresh README versioning guidance and remove the root `VERSION` file

## v1.0.0 - 2026-03-30
- Initial release
