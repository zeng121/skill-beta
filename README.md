# Fotor Skills

This repository stores reusable [Agent Skills](https://agentskills.io/home) for Fotor AI.

## Current Skill

### `test-openapi`

An async-first Python workflow for Fotor OpenAPI using the `fotor-sdk` package.

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

The skill includes setup scripts, execution tooling, model references, and parameter documentation.

## Repository Structure

```text
.
├── README.md
└── test-openapi/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    ├── references/
    │   ├── image_models.md
    │   ├── parameter_reference.md
    │   └── video_models.md
    └── scripts/
        ├── ensure_sdk.py
        └── run_task.py
```

## Quick Start

1. Enter the skill directory:

```bash
cd test-openapi
```

2. Upgrade the SDK to the latest version:

```bash
python scripts/ensure_sdk.py
```

3. Set your API key:

```bash
export FOTOR_OPENAPI_KEY="<your_api_key>"
```

4. Run a sample task:

```bash
echo '{"task_type":"text2image","params":{"prompt":"A cat astronaut","model_id":"seedream-4-5-251128"}}' \
  | python scripts/run_task.py
```

## Batch Execution

You can run multiple tasks in parallel:

```bash
echo '[
  {"task_type":"text2image","params":{"prompt":"A neon city","model_id":"seedream-4-5-251128"},"tag":"img-1"},
  {"task_type":"text2video","params":{"prompt":"A futuristic skyline","model_id":"kling-v3","duration":5},"tag":"vid-1"}
]' | python scripts/run_task.py --concurrency 5
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

## Model and Parameter References

Use these files when selecting models and building parameters:

- `test-openapi/references/image_models.md`
- `test-openapi/references/video_models.md`
- `test-openapi/references/parameter_reference.md`

## Environment Variables

- `FOTOR_OPENAPI_KEY` (required): API key for authentication.
- `FOTOR_OPENAPI_ENDPOINT` (optional): API base URL. Defaults to `https://api-b.fotor.com`.

## Common Issues

- `FOTOR_OPENAPI_KEY not set`
  - Export the key before running `run_task.py`.
- `Unknown task_type`
  - Use one of the 8 supported task types listed above.
- Unsupported `model_id` or invalid parameters
  - Cross-check against `references/` docs.

## Contributing

When updating this repository:

- Keep task names and parameter examples consistent with `fotor-sdk`.
- Update reference documents when model lists or capabilities change.
- Keep runnable command examples in `SKILL.md` and this README aligned.
