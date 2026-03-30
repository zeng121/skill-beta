# Configure `FOTOR_OPENAPI_KEY`

Use this guide when the user asks how to set the API key required by the Fotor SDK and the scripts in this skill.

## Default Recommendation

For most users, especially non-developers, prefer a local `.env` file in the working directory plus a short verification step. Do not start by listing multiple equivalent setup methods.

## Quick Start for Most Users

Tell the user to do exactly this:

1. Create a file named `.env` in the directory where they will run the skill scripts
2. Put this line into the file:

```env
FOTOR_OPENAPI_KEY=your-real-key-here
```

3. Load the file into the current shell:

```bash
set -a && source .env && set +a
```

4. Verify the key is visible:

```bash
python - <<'PY'
import os
print('OK: FOTOR_OPENAPI_KEY is set' if os.getenv('FOTOR_OPENAPI_KEY') else 'Missing: FOTOR_OPENAPI_KEY')
PY
```

5. Run a tiny smoke test:

```bash
python scripts/ensure_sdk.py

echo '{"task_type":"text2image","params":{"prompt":"A red apple on a white table","model_id":"seedream-4-5-251128"}}' \
  | python scripts/run_task.py
```

This flow is the best default because it is persistent enough for repeated use, easy to explain, and easy to verify.

## If the User Already Gave the Key to the Assistant

If the user has already provided the real API key and wants help getting started quickly, prefer creating a local `.env` file for them instead of explaining multiple environment-variable strategies. Keep the key out of chat where possible and avoid echoing it back unnecessarily.

## Alternative Methods

Only mention these if the default `.env` flow is not suitable.

### Temporary Session-Only Setup

Good for one-off testing. The key disappears when the shell session ends.

```bash
export FOTOR_OPENAPI_KEY='your-real-key-here'
```

### Shell Profile Setup

Good for users who want the key available in every terminal session, but this is more advanced and shell-specific.

Add the export line to a shell profile such as `~/.bashrc` or `~/.zshrc`, then reload the shell.

## Common Failure Modes

- Verification says `Missing: FOTOR_OPENAPI_KEY`
  - The `.env` file was not sourced in the current shell
  - The variable name was typed incorrectly
- Auth failure / 401 / 403
  - The key is wrong, expired, revoked, or copied with extra whitespace
- Works once, then stops working in a new terminal
  - The key was only set for one shell session

## Safety Notes

- Never paste the real key into public chat logs, screenshots, or committed source files
- Prefer a local `.env` file or environment variable over hardcoding the secret in Python code
- Add `.env` to `.gitignore` if the repo does not already ignore it
