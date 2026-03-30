---
name: test-openapi
description: Use when the user's intent is visual and the task can be solved with Fotor OpenAPI image or video generation, editing, transformation, enhancement, or batch output, including product photos, marketing creatives, posters,banners, social covers, background changes, upscaling, restoration, and other image- or video-related asset workflows.
---

# test-openapi

Async-first Python SDK for the Fotor OpenAPI. No MCP -- just an API key.

## Setup

Create a virtual Python environment using any available method, such as `venv`, and run the following commands inside it:

1. **Install or upgrade the latest `fotor-sdk` before every task**
   ```bash
   python scripts/ensure_sdk.py
   ```
2. **Ensure `FOTOR_OPENAPI_KEY`** is set in environment.

## Scripts

### `scripts/ensure_sdk.py`

Cross-platform (Windows / macOS / Linux) script to install or upgrade `fotor-sdk` to the latest PyPI release. Run before every task.

- **No args** — install or upgrade to the latest PyPI release
- **`--upgrade`** — same behavior, kept as an explicit alias

### `scripts/run_task.py`

Execute one or more Fotor tasks from JSON. Handles client init, polling, and progress.

**Single task:**
```bash
echo '{"task_type":"text2image","params":{"prompt":"A cat","model_id":"seedream-4-5-251128"}}' \
  | python scripts/run_task.py
```

**Batch (array):**
```bash
echo '[
  {"task_type":"text2image","params":{"prompt":"A cat","model_id":"seedream-4-5-251128"},"tag":"cat"},
  {"task_type":"text2video","params":{"prompt":"Sunset","model_id":"kling-v3","duration":5},"tag":"sunset"}
]' | python scripts/run_task.py --concurrency 5
```

**Options:** `--input FILE`, `--concurrency N` (default 5), `--poll-interval S` (default 2.0), `--timeout S` (default 1200).

**Output:** JSON with `task_id`, `status`, `success`, `result_url`, `error`, `elapsed_seconds`, `tag`.

Automatic fallback:

- If a task fails on its primary model and the current `task_type + model_id` matches a built-in fallback mapping, `run_task.py` automatically retries once with the fallback model.
- The output includes `fallback_used`, `original_model_id`, and `fallback_model_id`.

### `scripts/upload_image.py`

Upload a local image file through Fotor's signed upload flow and return a reusable image URL.

```bash
python scripts/upload_image.py ./input.jpg --task-type image2image
```

The script:

- Calls `/v1/upload/sign` with the mapped upload `type` and `suffix`
- Uploads the local file to the signed target
- Prints JSON containing `file_url` and `upload_url`

Use `file_url` as the `image_url`, `start_image_url`, `end_image_url`, or an item inside `image_urls` for image-based tasks.

Supported task-to-upload mapping:

- `image2image` -> `img2img`
- `image_upscale` -> `img_upscale`
- `background_remove` -> `bg_remove`
- `single_image2video` -> `img2video`
- `start_end_frame2video` -> `img2video`
- `multiple_image2video` -> `img2video`

## Reference Files

Read these before selecting a model or constructing parameters:

- `references/image_models.md` -- image model IDs, T2I/I2I capabilities, per-model parameter constraints (resolution, ratios, input limits, max refs)
- `references/video_models.md` -- video model IDs, T2V/I2V/SE/MI capabilities, per-model parameter constraints (duration, resolution, ratios, input limits, audio)
- `references/parameter_reference.md` -- full function signatures and parameter tables for all 8 task types
- `references/image_scenarios.md` -- scenario-to-model mapping for image generation (T2I, I2I, utilities); read when user intent is ambiguous
- `references/video_scenarios.md` -- scenario-to-model mapping for video generation (T2V, I2V, SE, MI); read when user intent is ambiguous

## Workflow

1. Run `python scripts/ensure_sdk.py` before every task to install or upgrade the latest `fotor-sdk`.
2. Verify `FOTOR_OPENAPI_KEY` is set.
3. For image-based tasks that start from a local file, first run `python scripts/upload_image.py <local-file> --task-type <task-type>` and keep the returned `file_url`.
4. Read the appropriate model reference to choose `model_id`. Each model's per-model spec section lists supported resolutions, aspect ratios, duration, input constraints, and max reference images.
5. If user intent is ambiguous (no specific model requested), consult the scenario files (`image_scenarios.md` / `video_scenarios.md`) for recommended model + params.
6. **Validate parameters** against the chosen model's spec before calling -- check resolution, aspect ratio, duration, and multi-image limits.
7. **Quick path** -- pipe JSON into `scripts/run_task.py` (works for both single and batch).
8. **Custom path** -- write inline Python using the SDK directly (see examples below).
9. Check `result_url` in output. Chain `image_upscale` if higher resolution needed.

Built-in automatic fallback mappings:

- `text2image`: `gemini-3.1-flash-image-preview` -> `seedream-5-0-260128`
- `image2image`: `gemini-3.1-flash-image-preview` -> `seedream-5-0-260128`
- `text2video`: `doubao-seedance-1-5-pro` -> `kling-v3`
- `single_image2video`: `doubao-seedance-1-5-pro` -> `kling-v3`
- `start_end_frame2video`: `kling-video-o1` -> `viduq2-turbo`
- `multiple_image2video`: `kling-v3-omni` -> `kling-video-o1`

## Available Task Types

| task_type | Function | Required Params |
|-----------|----------|-----------------|
| `text2image` | `text2image()` | `prompt`, `model_id` |
| `image2image` | `image2image()` | `prompt`, `model_id`, `image_urls` |
| `image_upscale` | `image_upscale()` | `image_url` |
| `background_remove` | `background_remove()` | `image_url` |
| `text2video` | `text2video()` | `prompt`, `model_id` |
| `single_image2video` | `single_image2video()` | `prompt`, `model_id`, `image_url` |
| `start_end_frame2video` | `start_end_frame2video()` | `prompt`, `model_id`, `start_image_url`, `end_image_url` |
| `multiple_image2video` | `multiple_image2video()` | `prompt`, `model_id`, `image_urls` (≥2) |

For full parameter details (defaults, `on_poll`, `**extra`), read `references/parameter_reference.md`.

## Inline Python Examples

When `scripts/run_task.py` is insufficient (custom logic, chaining, progress callbacks):

### Client Init

```python
import os
from fotor_sdk import FotorClient
client = FotorClient(api_key=os.environ["FOTOR_OPENAPI_KEY"])
```

### Single Task

```python
from fotor_sdk import text2image
result = await text2image(client, prompt="A diamond kitten", model_id="seedream-4-5-251128")
print(result.result_url)
```

### Batch with TaskRunner

```python
from fotor_sdk import TaskRunner, TaskSpec
runner = TaskRunner(client, max_concurrent=5)
specs = [
    TaskSpec("text2image", {"prompt": "A cat", "model_id": "seedream-4-5-251128"}, tag="cat"),
    TaskSpec("text2video", {"prompt": "Sunset", "model_id": "kling-v3", "duration": 5}, tag="sunset"),
]
results = await runner.run(specs)
```

### Video with Audio

```python
from fotor_sdk import text2video
result = await text2video(client, prompt="Jazz band", model_id="kling-v3",
                          audio_enable=True, audio_prompt="Smooth jazz")
```

## TaskResult

```python
result.success          # bool: True when COMPLETED with result_url
result.result_url       # str | None
result.status           # TaskStatus: COMPLETED / FAILED / TIMEOUT / IN_PROGRESS / CANCELLED
result.error            # str | None (e.g. "NSFW_CONTENT")
result.elapsed_seconds  # float
result.metadata         # dict (includes "tag" from TaskRunner)
```

## Error Handling

- **Single task**: catch `FotorAPIError` (has `.code` attribute).
- **Batch**: check `result.success` per item; runner never raises on individual failures.
- **NSFW**: appears as `error="NSFW_CONTENT"` in TaskResult.

For troubleshooting, enable SDK debug logging: `logging.getLogger("fotor_sdk").setLevel(logging.DEBUG)`.
