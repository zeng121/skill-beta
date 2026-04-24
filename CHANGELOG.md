# Changelog

## v1.0.18 - 2026-04-23
- Bump the published skill version to `1.0.18`
- Add `gpt-image-2` to the image model references and update eligible image generation / editing scenarios to use it where the documented parameter bounds are satisfied
- Add `doubao-seedance-2-0-260128` (Seedance 2.0) to the video model references and make it the primary model across the video scenario references
- Consolidate automatic model fallback mappings into `skills/fotor-skills/references/fallback_models.json` so `run_task.py` and the reference docs share a single fallback source of truth

## v1.0.17 - 2026-04-16
- Refresh `skills/fotor-skills/SKILL.md` for better ClawHub discovery with a more business-facing structure, stronger search-intent coverage, and a more SEO-oriented Fotor-branded description, while publishing version `1.0.17`

## v1.0.16 - 2026-04-10
- Bump the published skill version to `1.0.16`
- Normalize the Fotor developer homepage URL to include a trailing slash in `skills/fotor-skills/SKILL.md` and `skills/fotor-skills/references/get_api_key.md`

## v1.0.15 - 2026-04-09
- Bump the published skill version to `1.0.15`
- Update GitHub / `npx skills` install commands to use `--copy -y` in `README.md` and `skills/fotor-skills/references/install-or-upgrade.md`

## v1.0.14 - 2026-04-09
- Bump the published skill version to `1.0.14`
- Update the `skills/fotor-skills/SKILL.md` description to point users to the Fotor developer page for API key application and feature details
- Remove `metadata.openclaw.requires.config` from `skills/fotor-skills/SKILL.md`

## v1.0.13 - 2026-04-08
- Bump the published skill version to `1.0.13`
- Add `skills/fotor-skills/references/get_api_key.md` for homepage and API key onboarding
- Remove `skills/fotor-skills/references/configure-fotor-openapi-key.md` and fold key guidance into the new onboarding reference
- Update `skills/fotor-skills/SKILL.md` and `README.md` to match the new routing and published version

## v1.0.12 - 2026-04-07
- Bump the published skill version to `1.0.12`
- Refresh the `skills/fotor-skills/SKILL.md` description with a broader product-style summary covering AI photo editing, AI video generation, and common marketing and e-commerce asset workflows

## v1.0.11 - 2026-04-03
- Bump the published skill version to `1.0.11`
- Update `scripts/check_skill_update.py` to read the top-level `version` field first and fall back to legacy `metadata.version` for GitHub-based installs
- Align the update-check documentation in `SKILL.md` and `README.md` with the top-level `version` field

## v1.0.10 - 2026-04-03
- Bump the published skill version to `1.0.10`
- Add recharge guidance to `SKILL.md`, including calling `GET /v1/payment/links` when users ask how to buy credits or top up
- Consolidate credit lookup and recharge guidance into a single `Credits and Recharge` section in `SKILL.md`
- Extract detailed credit lookup and recharge behavior into `references/credits-and-recharge.md` so `SKILL.md` stays focused on routing rules
- Keep the API-returned `urlExpireTime` field in recharge results for internal use, while simplifying user guidance to say payment links are valid for 30 minutes
- Document that insufficient-credit failures (`code=510` / `No enough credits`) should trigger recharge guidance instead of ending at the raw task error
- Update `scripts/run_task.py` so insufficient-credit API errors return immediately and do not trigger model fallback retries
- Add regression coverage for `scripts/run_task.py` to verify that `code=510` skips fallback while non-credit API errors can still retry on fallback models

## v1.0.9 - 2026-04-02
- Bump the published skill version to `1.0.9`
- Align the credit lookup example in `SKILL.md` with the default `FOTOR_OPENAPI_ENDPOINT` value of `https://api-b.fotor.com`

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
