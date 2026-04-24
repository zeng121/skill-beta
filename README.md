# Fotor Skills

Current public skill version: `1.0.18`

This repository stores reusable [Agent Skills](https://skills.sh/) for Fotor AI.

Official product page: `https://developers.fotor.com/fotor-skills/`

## Version Management

- Keep `skills/fotor-skills/SKILL.md` top-level `version` aligned with the current published skill version.
- Keep `CHANGELOG.md` updated for GitHub / `npx skills` installs.
- Avoid a separate root-level `VERSION` file.

## Current Skill

### `fotor-skills`

An all-in-one AI photo editor and AI video generator workflow for generating, editing, transforming, and enhancing images and videos with the `fotor-sdk` package.

For API key application and product details, see `https://developers.fotor.com/fotor-skills/`.

SDK compatibility follows the latest `fotor-sdk` release installed by `scripts/ensure_sdk.py`.

It supports:

- Text-to-Image (`text2image`)
- Image-to-Image (`image2image`)
- Image Upscale (`image_upscale`)
- Background Removal (`background_remove`)
- Text-to-Video (`text2video`)
- Single Image-to-Video (`single_image2video`)
- Start/End Frame Interpolation (`start_end_frame2video`)
- Multiple Image-to-Video (`multiple_image2video`)
- Credit lookup (`get_credits_sync`)

The skill includes setup scripts, execution tooling, model references, and parameter documentation.

## Repository Structure

```text
.
├── README.md
└── skills/
    └── fotor-skills/
        ├── SKILL.md
        ├── agents/
        ├── references/
        └── scripts/
```

## Install

### ClawHub

```bash
clawhub install fotor-skills
```

### GitHub / `npx skills`

```bash
npx skills add https://github.com/fotor-ai/fotor-skills.git --skill fotor-skills --copy -y
```

## Quick Start

1. Enter the skill directory:

```bash
cd skills/fotor-skills
```

2. Create a local virtual environment with `uv`:

```bash
uv python install 3.12
uv venv --python 3.12 .venv
```

3. Upgrade the SDK to the latest version:

```bash
./.venv/bin/python scripts/ensure_sdk.py
```

4. Configure your API key. Recommended local `.env` setup:

```bash
cat > .env <<'EOF'
FOTOR_OPENAPI_KEY=<your_api_key>
EOF

set -a && source .env && set +a
```

5. Run a sample task:

```bash
cat <<'EOF' | ./.venv/bin/python scripts/run_task.py
{"task_type":"text2image","params":{"prompt":"A cat astronaut","model_id":"seedream-4-5-251128"}}
EOF
```

## Batch Execution

You can run multiple tasks in parallel:

```bash
cat <<'EOF' | ./.venv/bin/python scripts/run_task.py --concurrency 5
[
  {"task_type":"text2image","params":{"prompt":"A neon city","model_id":"seedream-4-5-251128"},"tag":"img-1"},
  {"task_type":"text2video","params":{"prompt":"A futuristic skyline","model_id":"kling-v3","duration":5},"tag":"vid-1"}
]
EOF
```

## Input and Output Format

- Input: JSON object (single task) or JSON array (batch).
- Output: structured JSON with fields such as `task_id`, `status`, `success`, `result_url`, `error`, `elapsed_seconds`, `creditsIncrement`, and `tag`.

## Scripts

- `scripts/ensure_sdk.py`
  - Install or upgrade `fotor-sdk` with `uv`.
- `scripts/upload_image.py`
  - Upload a local image file and return a reusable `file_url`.
- `scripts/run_task.py`
  - Run one or more OpenAPI tasks from JSON input.
- `scripts/check_skill_update.py`
  - Checks whether a newer version of the installed skill is available.

For credit lookup via `client.get_credits_sync()`, the SDK returns a dict like `{"businessId": "", "total": 2000, "remaining": 1973}`.

## Model and Parameter References

Use these files when selecting models and building parameters:

- `skills/fotor-skills/references/image_models.md`
- `skills/fotor-skills/references/video_models.md`
- `skills/fotor-skills/references/parameter_reference.md`

## Environment Variables

- `FOTOR_OPENAPI_KEY` (required): API key for authentication.
- `FOTOR_OPENAPI_ENDPOINT` (optional): API base URL. Defaults to `https://api-b.fotor.com`.

## Common Issues

- `FOTOR_OPENAPI_KEY not set`
  - Add it to `.env`, source the file, then retry.
- Unknown `task_type`
  - Use one of the 8 supported task types listed above.
- Unsupported `model_id` or invalid parameters
  - Cross-check against `references/` docs.

## Contributing

When updating this repository:

- Keep `skills/fotor-skills/SKILL.md` top-level `version` aligned with the published version.
- Keep `CHANGELOG.md` aligned with the published version.
- Update reference documents when model lists or capabilities change.
- Keep runnable command examples in `SKILL.md` and this README aligned.
