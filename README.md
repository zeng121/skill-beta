# Fotor Skills

Current public skill version: `1.0.1`

This repository stores reusable [Agent Skills](https://skills.sh/) for Fotor AI.

## Version Management

Version management is intentionally simple and source-aligned:

- `skills/test-openapi/SKILL.md` frontmatter `metadata.version` is the canonical version source for the GitHub / `npx skills` distribution path.
- ClawHub releases should use the same version number.
- Git tags should stay aligned with the same version when publishing updates.
- For the GitHub / `npx skills` path, update notes should be maintained in the repository `CHANGELOG.md`.
- Avoid maintaining a separate root-level `VERSION` file.

## Current Skill

### `test-openapi`

An async-first workflow for Fotor OpenAPI using the `fotor-sdk` package.

Current compatibility target: this skill is adapted to `fotor-sdk` `0.1.3`.

It supports:

- Text-to-Image (`text2image`)
- Image-to-Image (`image2image`)
- Image Upscale (`image_upscale`)
- Background Removal (`background_remove`)
- Text-to-Video (`text2video`)
- Single Image-to-Video (`single_image2video`)
- Start/End Frame Interpolation (`start_end_frame2video`)
- Multiple Image-to-Video (`multiple_image2video`)

The skill includes setup scripts, execution tooling, model references, parameter documentation, and install-source-aware update checking.

## Repository Structure

```text
.
├── README.md
└── skills/
    └── test-openapi/
        ├── SKILL.md
        ├── agents/
        ├── references/
        └── scripts/
```

## Install

### ClawHub

```bash
clawhub install test-openapi
```

### GitHub / `npx skills`

```bash
npx skills add https://github.com/zeng121/skill-beta.git --skill test-openapi
```

## Quick Start

1. Enter the skill directory:

```bash
cd skills/test-openapi
```

2. Upgrade the SDK to the latest version:

```bash
python scripts/ensure_sdk.py
```

3. Configure your API key. Recommended local `.env` setup:

```bash
cat > .env <<'EOF'
FOTOR_OPENAPI_KEY=<your_api_key>
EOF

set -a && source .env && set +a
```

4. Run a sample task:

```bash
cat <<'EOF' | python scripts/run_task.py
{"task_type":"text2image","params":{"prompt":"A cat astronaut","model_id":"seedream-4-5-251128"}}
EOF
```

## Batch Execution

You can run multiple tasks in parallel:

```bash
cat <<'EOF' | python scripts/run_task.py --concurrency 5
[
  {"task_type":"text2image","params":{"prompt":"A neon city","model_id":"seedream-4-5-251128"},"tag":"img-1"},
  {"task_type":"text2video","params":{"prompt":"A futuristic skyline","model_id":"kling-v3","duration":5},"tag":"vid-1"}
]
EOF
```

## Input and Output Format

- Input: JSON object (single task) or JSON array (batch).
- Output: structured JSON with fields such as `task_id`, `status`, `success`, `result_url`, `error`, `elapsed_seconds`, and `tag`.

## Scripts

- `scripts/ensure_sdk.py`
  - Upgrades `fotor-sdk` to the latest PyPI version each time it runs.
  - `--upgrade` is kept as an explicit alias for the same behavior.
- `scripts/upload_image.py`
  - Uploads a local image file and returns a reusable `file_url`.
  - Requires `--task-type` and maps tasks as follows:
    - `image2image` -> `img2img`
    - `image_upscale` -> `img_upscale`
    - `background_remove` -> `bg_remove`
    - `single_image2video` -> `img2video`
    - `start_end_frame2video` -> `img2video`
    - `multiple_image2video` -> `img2video`
- `scripts/run_task.py`
  - Runs one or more tasks from stdin or `--input`.
  - Supports `--concurrency`, `--poll-interval`, and `--timeout`.
- `scripts/check_skill_update.py`
  - Detects whether the skill was installed via ClawHub or via GitHub / `npx skills`.
  - Uses the matching version-check backend.
  - Supports a low-frequency cached update reminder flow.

## Model and Parameter References

Use these files when selecting models and building parameters:

- `skills/test-openapi/references/image_models.md`
- `skills/test-openapi/references/video_models.md`
- `skills/test-openapi/references/parameter_reference.md`

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

- Keep `skills/test-openapi/SKILL.md` `metadata.version` aligned with the published version.
- Keep GitHub tags / releases and ClawHub releases aligned to the same version.
- Update reference documents when model lists or capabilities change.
- Keep runnable command examples in `SKILL.md` and this README aligned.
