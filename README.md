# Sora Video Generation Skill

A GitHub Copilot skill for generating and managing short video clips through OpenAI's Sora API (`sora-2` / `sora-2-pro`). The bundled CLI (`scripts/sora.py`) handles creation, polling, remixing, downloading, and batch runs — all from the command line.

![Sora](assets/sora.png)

---

## Features

- **Generate** video clips from a text prompt (async or synchronous create-and-poll)
- **Remix** an existing video by ID with targeted prompt changes
- **Poll / Status** — wait for a job to finish or inspect its current state
- **Download** the final video, thumbnail, or spritesheet
- **Batch** — submit many prompts at once from a JSONL file
- **Prompt augmentation** — the CLI reformats short prompts into a structured production spec automatically
- **Dry-run mode** — preview the full request without touching the API

---

## Prerequisites

| Requirement | Notes |
|---|---|
| Python 3.9+ | Comes with most modern systems |
| [`uv`](https://github.com/astral-sh/uv) | Used to run the CLI with its `openai` dependency |
| `OPENAI_API_KEY` | Must be set in the environment before any live API call |
| Sora API access | Organization-verified OpenAI account required |

> **Never paste your API key in chat.** Set it as an environment variable and confirm when ready.

---

## Setup

```bash
# 1. Set your API key
export OPENAI_API_KEY="sk-..."

# 2. (Optional) point a variable to the CLI for convenience
export SORA_CLI="$(git rev-parse --show-toplevel)/scripts/sora.py"

# 3. If uv cache hits permission errors, redirect it
export UV_CACHE_DIR="/tmp/uv-cache"
```

---

## Quick Start

```bash
# Dry-run — no API call, no network required
python "$SORA_CLI" create --prompt "Test shot" --dry-run

# Create a 4-second 1280×720 video
uv run --with openai python "$SORA_CLI" create \
  --model sora-2 \
  --prompt "Wide tracking shot of a teal coupe on a desert highway" \
  --size 1280x720 \
  --seconds 4

# Create, wait until done, and download in one step
uv run --with openai python "$SORA_CLI" create-and-poll \
  --prompt "Close-up of a steaming coffee cup on a wooden table" \
  --size 1280x720 \
  --seconds 8 \
  --download \
  --out coffee.mp4
```

---

## CLI Reference

All commands are subcommands of `scripts/sora.py`.

| Command | Description |
|---|---|
| `create` | Submit a new video job (async) |
| `create-and-poll` | Submit + poll until complete + optionally download |
| `poll` | Wait for an existing job by ID |
| `status` | Retrieve current job details |
| `download` | Download video, thumbnail, or spritesheet |
| `list` | List recent jobs |
| `delete` | Delete a job |
| `remix` | Remix a completed video by ID |
| `create-batch` | Submit many jobs from a JSONL file |

### Common flags

| Flag | Default | Description |
|---|---|---|
| `--model` | `sora-2` | `sora-2` or `sora-2-pro` |
| `--size` | `1280x720` | Supported sizes vary by model (see `references/video-api.md`) |
| `--seconds` | `4` | `"4"`, `"8"`, or `"12"` |
| `--prompt` | — | Inline prompt text |
| `--prompt-file` | — | Path to a text file containing the prompt |
| `--no-augment` | — | Skip automatic prompt augmentation |
| `--dry-run` | — | Preview the request without calling the API |
| `--json-out` | — | Write the JSON response to a file |
| `--download` | — | Download the asset after polling completes |
| `--out` | — | Output file path for the downloaded asset |
| `--variant` | `video` | `video`, `thumbnail`, or `spritesheet` |

For the full command catalog see [`references/cli.md`](references/cli.md).

---

## Prompt Augmentation

The CLI automatically reformats a short prompt into a structured production spec. You can also supply individual augmentation flags:

```bash
uv run --with openai python "$SORA_CLI" create \
  --prompt "A minimal product teaser shot of a matte black camera" \
  --use-case "landing page hero" \
  --camera "85mm, slow orbit" \
  --lighting "soft key, subtle rim" \
  --constraints "no logos, no text"
```

If your prompt is already structured, pass `--no-augment` to avoid double-wrapping.

---

## Remix Workflow

```bash
# 1. Create the base clip
uv run --with openai python "$SORA_CLI" create-and-poll \
  --prompt "Ceramic mug on a sunlit wooden table" \
  --size 1280x720 --seconds 4 --download --out base.mp4

# 2. Remix (change only the subject)
uv run --with openai python "$SORA_CLI" remix \
  --id video_abc123 \
  --prompt "Same shot and framing; replace the mug with an iced americano in a glass."

# 3. Poll and download the remix
uv run --with openai python "$SORA_CLI" poll \
  --id video_def456 --download --out remix.mp4
```

---

## Batch Runs

```bash
mkdir -p tmp/sora
cat > tmp/sora/prompts.jsonl << 'EOB'
{"prompt":"A neon-lit rainy alley, slow dolly-in","seconds":"4"}
{"prompt":"A warm sunrise over a misty lake, gentle pan","seconds":"8"}
EOB

uv run --with openai python "$SORA_CLI" create-batch \
  --input tmp/sora/prompts.jsonl \
  --out-dir out \
  --concurrency 3

rm -f tmp/sora/prompts.jsonl
```

Each line is a JSON object with at least a `prompt` key. Optional keys: `model`, `size`, `seconds`, `input_reference`, and augmentation fields.

---

## Defaults

| Setting | Value |
|---|---|
| Model | `sora-2` |
| Size | `1280x720` |
| Duration | `4` seconds |
| Variant | `video` |
| Poll interval | `10` seconds |
| Batch concurrency | `3` |

---

## Guardrails

- Content must be suitable for audiences under 18.
- No copyrighted characters or copyrighted music.
- No real people (including public figures).
- Input images containing human faces are rejected by the API.

---

## Reference Files

| File | Purpose |
|---|---|
| [`references/cli.md`](references/cli.md) | Full CLI command catalog with examples |
| [`references/video-api.md`](references/video-api.md) | API parameters — models, sizes, duration, variants |
| [`references/prompting.md`](references/prompting.md) | Prompt structure and iteration guidance |
| [`references/sample-prompts.md`](references/sample-prompts.md) | Copy-paste prompt recipes |
| [`references/cinematic-shots.md`](references/cinematic-shots.md) | Templates for filmic shots |
| [`references/social-ads.md`](references/social-ads.md) | Templates for short social ad beats |
| [`references/troubleshooting.md`](references/troubleshooting.md) | Common errors and fixes |
| [`references/codex-network.md`](references/codex-network.md) | Network and sandbox troubleshooting |
| [`SKILL.md`](SKILL.md) | Copilot skill definition |

---

## License

See [LICENSE.txt](LICENSE.txt).
