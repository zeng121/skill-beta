---
name: fotor-skills
description: Use when the user's intent is visual and the task can be solved with Fotor OpenAPI image or video generation, editing, transformation, enhancement, batch output, account credit lookup, or recharge guidance, including product photos, marketing creatives, posters,banners, social covers, background changes, upscaling, restoration, and other image- or video-related asset workflows.
version: 1.0.10
metadata:
  author: fotor-ai
  openclaw:
    requires:
      env:
        - FOTOR_OPENAPI_KEY
      bins:
        - uv
      config:
        - _meta.json
        - skills-lock.json
    primaryEnv: FOTOR_OPENAPI_KEY
    homepage: https://github.com/fotor-ai/fotor-skills
---

# fotor-skills

Async-first Python SDK for the Fotor OpenAPI. No MCP -- just an API key.

Use `uv` as the skill's bootstrap layer. Prefer a skill-local Python 3.12 environment and run bundled scripts from that local environment instead of the system Python.

## Setup

Keep setup lightweight and local to the skill directory.

Install `uv` first if it is missing:

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Typical first-run setup:

```bash
uv python install 3.12
uv venv --python 3.12 .venv
./.venv/bin/python scripts/ensure_sdk.py
```

Setup rules:

1. Prefer a local Python 3.12 environment in the skill directory.
2. Use `uv` to prepare Python 3.12 and create `.venv` when the local environment is missing.
3. Run bundled scripts from the local skill environment, not the system Python.
4. Ensure `FOTOR_OPENAPI_KEY` is set. If key setup is missing and the user is not technical, read `references/configure-fotor-openapi-key.md` and prefer the local `.env` happy path.

Current default interpreter paths:

- POSIX: `./.venv/bin/python`
- Windows: `.venv\\Scripts\\python.exe`

## Interaction Rules

- Speak in user-task language first. Do not lead with SDK, scripts, JSON, model IDs, or parameter tables unless they are needed to unblock the task or the user explicitly asks.
- Ask for only one missing blocker at a time.
- Once the minimum required information is present, execute immediately. Do not send vague transition messages like "I’m starting now" unless execution has actually started and a result or clear in-progress status will follow.
- If execution will take noticeable time, say that the task is running and give a short expectation such as "usually takes a few seconds to a few dozen seconds; I’ll send the result when it’s ready."
- If credentials are missing, resolve that blocker quickly and then return to the original task instead of turning the conversation into a long setup lesson.
- When the local skill environment is missing, prepare it with `uv` before installing dependencies or executing the task. Avoid installing into the system Python unless the user explicitly asks.
- Choose the model and default parameters internally unless the user explicitly requests a specific model or technical control.
- Return the result as soon as it is ready. Do not make the user ask follow-up questions like "where is the image?"
- If the user asks how to recharge, buy credits, top up, or purchase tokens, use `references/credits-and-recharge.md` and follow its recharge guidance flow.
- If a task fails because credits are insufficient, do not stop at the raw error. Use `references/credits-and-recharge.md` to explain the failure and provide recharge guidance.
- If an update reminder is available, keep it to one short non-blocking sentence and continue the current task.

## Scripts

### `scripts/ensure_sdk.py`

Cross-platform (Windows / macOS / Linux) script to install or upgrade `fotor-sdk` to the latest PyPI release with `uv pip install --python <interpreter>`. Run before every task.

- **No args** — install or upgrade to the latest PyPI release
- **`--upgrade`** — same behavior, kept as an explicit alias

### `scripts/run_task.py`

Execute one or more Fotor tasks from JSON. Handles client init, polling, and progress.

**Single task:**
```bash
echo '{"task_type":"text2image","params":{"prompt":"A cat","model_id":"seedream-4-5-251128"}}' \
  | ./.venv/bin/python scripts/run_task.py
```

**Batch (array):**
```bash
echo '[
  {"task_type":"text2image","params":{"prompt":"A cat","model_id":"seedream-4-5-251128"},"tag":"cat"},
  {"task_type":"text2video","params":{"prompt":"Sunset","model_id":"kling-v3","duration":5},"tag":"sunset"}
]' | ./.venv/bin/python scripts/run_task.py --concurrency 5
```

**Options:** `--input FILE`, `--concurrency N` (default 5), `--poll-interval S` (default 2.0), `--timeout S` (default 1200).

**Output:** JSON with `task_id`, `status`, `success`, `result_url`, `error`, `elapsed_seconds`, `creditsIncrement`, `tag`.

Automatic fallback:

- If a task fails on its primary model and the current `task_type + model_id` matches a built-in fallback mapping, `run_task.py` automatically retries once with the fallback model.
- If the failure is insufficient credits (`code=510` / `No enough credits`), `run_task.py` returns the failure immediately and does not retry on a fallback model.
- The output includes `fallback_used`, `original_model_id`, and `fallback_model_id`.

### `scripts/upload_image.py`

Upload a local image file through Fotor's signed upload flow and return a reusable image URL.

```bash
./.venv/bin/python scripts/upload_image.py ./input.jpg --task-type image2image
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

### `scripts/check_skill_update.py`

Check whether the installed skill has a newer version available for the current install source.

```bash
./.venv/bin/python scripts/check_skill_update.py --mark-notified --check-interval-hours 24
```

For development/testing when install-source metadata is unavailable:

```bash
./.venv/bin/python scripts/check_skill_update.py --install-source skills-github --slug fotor-skills --current-version 1.0.0 --github-source fotor-ai/fotor-skills --mark-notified --check-interval-hours 24
```

The script:

- Detects the install source first: `clawhub` or `skills-github`
- For `clawhub`, reads installed `_meta.json` and fetches the latest version via `clawhub inspect <slug> --json`
- For `skills-github`, reads local `SKILL.md` frontmatter `metadata.version`, finds the GitHub source, and fetches the remote `SKILL.md` version plus `CHANGELOG.md` highlights when available
- Prints JSON with `install_source`, `current_version`, `latest_version`, `update_available`, and `should_notify`
- Stores the last-notified version in a local state file when `--mark-notified` is used
- Caches the last successful version check and supports a minimum recheck interval via `--check-interval-hours` (default 24)
- Includes `changelog_preview` so the reminder can mention the main highlights without dumping the full changelog
- Supports development/testing overrides such as `--install-source`, `--slug`, `--current-version`, and `--github-source`

## Reference Files

Only read the reference files that match the current need. Do not load all of them by default.

### Task Execution References

Read these when choosing a model, validating parameters, or mapping an ambiguous user request to a recommended workflow:

- `references/image_models.md` -- image model IDs, T2I/I2I capabilities, per-model parameter constraints (resolution, ratios, input limits, max refs)
- `references/video_models.md` -- video model IDs, T2V/I2V/SE/MI capabilities, per-model parameter constraints (duration, resolution, ratios, input limits, audio)
- `references/parameter_reference.md` -- full function signatures and parameter tables for all 8 task types
- `references/image_scenarios.md` -- scenario-to-model mapping for image generation (T2I, I2I, utilities); read when user intent is ambiguous
- `references/video_scenarios.md` -- scenario-to-model mapping for video generation (T2V, I2V, SE, MI); read when user intent is ambiguous

### Operational References

Read these only when the user asks about installation, upgrade, workspace layout, or credential setup. Do not read them for normal image/video task execution.

- `references/credits-and-recharge.md` -- credit lookup, payment links, recharge guidance, and insufficient-credit handling
- `references/install-or-upgrade.md` -- how to install or upgrade `fotor-skills` for both ClawHub and `npx skills`, and how to phrase a concise non-pushy upgrade reminder
- `references/configure-fotor-openapi-key.md` -- how to set, persist, verify, and safely handle `FOTOR_OPENAPI_KEY`

## Workflow

1. If `uv` is missing, install it first using the platform-appropriate installer shown in Setup.
2. If Python 3.12 is not already available to `uv`, run `uv python install 3.12`.
3. If `.venv` is missing, create it with `uv venv --python 3.12 .venv`.
4. Run `./.venv/bin/python scripts/ensure_sdk.py` before every task to install or upgrade the latest `fotor-sdk` with `uv`.
5. After `.venv` is ready and `ensure_sdk.py` completes, silently run `./.venv/bin/python scripts/check_skill_update.py --mark-notified --check-interval-hours 24`. Do not inspect the state file manually. Only if it reports `should_notify: true`, read `references/install-or-upgrade.md` and send one short non-blocking update reminder. If the check fails or times out, ignore it and continue without mentioning the failure.
6. Verify `FOTOR_OPENAPI_KEY` is set.
7. If the user asks about account credits, recharge, buying credits, or payment links, read `references/credits-and-recharge.md`, then use the appropriate balance or payment-links flow from that reference.
8. For image-based tasks that start from a local file, first run `./.venv/bin/python scripts/upload_image.py <local-file> --task-type <task-type>` and keep the returned `file_url`.
9. Read the appropriate model reference to choose `model_id`. Each model's per-model spec section lists supported resolutions, aspect ratios, duration, input constraints, and max reference images.
10. If user intent is ambiguous (no specific model requested), consult the scenario files (`image_scenarios.md` / `video_scenarios.md`) for recommended model + params.
11. **Validate parameters** against the chosen model's spec before calling -- check resolution, aspect ratio, duration, and multi-image limits.
12. **Quick path** -- pipe JSON into `./.venv/bin/python scripts/run_task.py` (works for both single and batch).
13. **Custom path** -- write inline Python using the SDK directly (see examples below), still preferring the local `.venv` interpreter.
14. Check `result_url` in output. Chain `image_upscale` if higher resolution needed.

If the user asks to check account credits or remaining credits, read `references/credits-and-recharge.md` and use the SDK client flow described there instead of `run_task.py`.

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

## Credits and Recharge

For any balance lookup, recharge guidance, or insufficient-credit case, read `references/credits-and-recharge.md`.

Keep `SKILL.md` focused on routing:

- Use the credits reference when the user asks about remaining balance, total credits, recharge, top-up, or payment links.
- Use the same reference when a task fails with `code=510` or `No enough credits`.
- Keep detailed API examples, field meanings, and user-facing recharge wording in the reference instead of expanding this main skill file.

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
result.creditsIncrement # int | float: credits consumed by this task
result.metadata         # dict (includes "tag" from TaskRunner)
```

## Error Handling

- **Single task**: catch `FotorAPIError` (has `.code` attribute).
- **Batch**: check `result.success` per item; runner never raises on individual failures.
- **NSFW**: appears as `error="NSFW_CONTENT"` in TaskResult.
- **Insufficient credits**: if `result.error`, exception text, or a combined fallback error contains `code=510` or `No enough credits`, treat it as a recharge case. Tell the user credits are insufficient, then fetch and present payment links.

For troubleshooting, enable SDK debug logging: `logging.getLogger("fotor_sdk").setLevel(logging.DEBUG)`.
