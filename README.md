# PixVerse API — Python client

[![Python 3.8+](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/) [![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE) [![Hosted on Synexa](https://img.shields.io/badge/hosted%20on-Synexa-6366f1.svg)](https://synexa.ai/explore/pixverse/pixverse-v6?utm_source=github&utm_medium=ugc&utm_campaign=pixverse-dev&utm_content=readme-badge&utm_term=tier-c)

PixVerse is a video generation platform whose models turn a still image and a prompt into a short video clip, in either a realistic or a stylised look, with optional generated audio. This repository is a small Python client for the PixVerse API as hosted on Synexa, so you can animate images with PixVerse V6 from a script with one `pip install` and an API token, without dealing with the platform's web app or its credit system.

You get a blocking `run()` that takes a first-frame image and a prompt and returns the video URL, a non-blocking create-and-poll path for batches, and webhook delivery for services that would rather be called back. The client has a single runtime dependency and no model weights. It is aimed at developers building social-content tooling, e-commerce video, ad variations or animated storyboards who want PixVerse as an HTTP call.

> **Try it now:** [https://synexa.ai/explore/pixverse/pixverse-v6](https://synexa.ai/explore/pixverse/pixverse-v6?utm_source=github&utm_medium=ugc&utm_campaign=pixverse-dev&utm_content=readme-top&utm_term=tier-c) — the hosted model behind this client. New accounts get a free trial credit.

## Contents

- [Why this client](#why-this-client)
- [Installation](#installation)
- [Quickstart](#quickstart)
- [Hosted models](#hosted-models)
- [Parameters](#parameters)
- [Advanced usage](#advanced-usage)
- [About PixVerse](#about-pixverse)
- [Use cases](#use-cases)
- [FAQ](#faq)
- [License](#license)

## Why this client

- **The weights are not published.** PixVerse V6 is a proprietary hosted model; the only way to run it programmatically is through an endpoint, and this client wraps the Synexa one.
- **No GPU or video serving stack.** Video diffusion models of this class need datacenter GPUs with tens of gigabytes of VRAM, plus a decoder, an audio model and a job queue. The hosted endpoint runs on Synexa's fleet and you pay per clip.
- **No cold start on your side.** The model is resident on the endpoint; a single-clip request and a thousand-clip batch see the same latency profile, and there is nothing to warm up.
- **Predictable cost.** `pixverse/pixverse-v6` is billed at $0.025 per run, so the cost of a campaign of clips is known before you submit it.

## Installation

```bash
pip install git+https://github.com/pixverse-dev/pixverse-api.git
```

Then set your API key (create one at [synexa.ai](https://synexa.ai?utm_source=github&utm_medium=ugc&utm_campaign=pixverse-dev&utm_content=readme-apikey&utm_term=tier-c)):

```bash
export SYNEXA_API_KEY="sk-..."
```

## Quickstart

```python
import pixverse_api

output = pixverse_api.run({
    "prompt": "A cinematic shot of a lighthouse at dawn, soft fog, warm light",
    "image_url": "https://example.com/input.png"
})
print(output)   # URL(s) of the generated result
```

Or with an explicit client:

```python
from pixverse_api import Client

client = Client(api_key="sk-...")
output = client.run({"prompt": "A cinematic shot of a lighthouse at dawn, soft fog, warm light", "image_url": "https://example.com/input.png"})
```

## Hosted models

| Model | Category | What it does | Price / run |
|---|---|---|---|
| [`pixverse/pixverse-v6`](https://synexa.ai/explore/pixverse/pixverse-v6?utm_source=github&utm_medium=ugc&utm_campaign=pixverse-dev&utm_content=readme-models&utm_term=tier-c) | image-to-video | PixVerse V6 animates a still image into stylised or realistic video, with optional audio. | $0.025 |

The default model is **`pixverse/pixverse-v6`**; pass `model="owner/name"` to `run()` to use another one from the table.

## Parameters

### `pixverse/pixverse-v6`

| Field | Type | Required | Default | Range | Description |
|---|---|---|---|---|---|
| `prompt` | string | yes | `Slow dolly in on the subject as the wind…` | — | Text prompt for the video generation. Limited to 2048 UTF-8 encoded bytes. Because the limit counts bytes rather than characters, emoji and non-Latin or accented characters (which use multiple bytes each) can push a visually short prompt over the cap. |
| `resolution` | string | no | `720p` | 360p, 540p, 720p, 1080p | The resolution of the generated video |
| `duration` | integer | no | `5` | 1, 15 | The duration of the generated video in seconds. v6 supports values from 1 to 15 seconds |
| `negative_prompt` | string | no | — | — | Negative prompt to be used for the generation. Limited to 2048 UTF-8 encoded bytes. Because the limit counts bytes rather than characters, emoji and non-Latin or accented characters (which use multiple bytes each) can push a visually short prompt over the cap. |
| `style` | string | no | — | anime, 3d_animation, clay, comic, cyberpunk | The style of the generated video |
| `seed` | integer | no | `random` | — | The same seed and the same prompt given to the same version of the model will output the same video every time. |
| `generate_audio_switch` | boolean | no | `False` | — | Enable audio generation (BGM, SFX, dialogue) |
| `generate_multi_clip_switch` | boolean | no | `False` | — | Enable multi-clip generation with dynamic camera changes |
| `thinking_type` | string | no | — | enabled, disabled, auto | Prompt optimization mode: 'enabled' to optimize, 'disabled' to turn off, 'auto' for model decision |
| `image_url` | file | yes | — | — | First frame of the video (.jpg/.png/.webp) |

## Advanced usage

**Submit without blocking, then poll:**

```python
prediction = client.run(input, wait=False)      # returns immediately
prediction = client.wait(prediction, timeout=300)
print(prediction["output"])
```

**Webhook on completion:**

```python
client.run(input, wait=False, webhook="https://your-app.example/hooks/synexa")
```

**Errors:**

```python
from pixverse_api import ModelError, PredictionTimeout

try:
    output = client.run(input)
except ModelError as e:
    print("failed:", e, e.prediction and e.prediction.get("id"))
except PredictionTimeout:
    print("still running — poll later")
```

Status values you will see on a prediction: `starting` → `processing` → `succeeded` | `failed`.

## About PixVerse

PixVerse ([app.pixverse.ai](https://app.pixverse.ai)) is a generative video platform developed by AIsphere. It offers text-to-video and image-to-video generation through a web app and mobile apps, with a library of effect templates, lip-sync, clip extension and a range of visual styles from photoreal to anime. The platform has shipped a numbered series of model versions; V6 is the version exposed by the endpoint this client calls.

PixVerse V6 takes a first-frame image plus a prompt and produces a video of 1 to 15 seconds at a selectable resolution. The endpoint exposes controls for `style`, a `negative_prompt`, a `seed` for reproducible output, a `generate_audio_switch` that adds background music, sound effects and dialogue, a `generate_multi_clip_switch` that lets the model cut between camera angles within one generation, and a `thinking_type` setting that decides whether the model rewrites your prompt before generating (`enabled`, `disabled` or `auto`). Prompts and negative prompts are capped at 2048 UTF-8 bytes; because the limit is in bytes, emoji and non-Latin characters count for more than one each.

Typical outputs are short vertical or landscape clips for social feeds, product spins from a single packshot, animated key art for games and film pitches, and multi-shot clips for ads where the camera changes without a second render. Limits to plan for: a single run produces a single clip, the maximum duration is 15 seconds, and complex multi-subject motion is best described in short, concrete sentences.

The hosted endpoint used by this client is `pixverse/pixverse-v6` on Synexa, which is PixVerse's own V6 model served through Synexa's API. It is the same generation model the PixVerse app uses, without the app's template library, editing timeline or account credits; those remain on PixVerse's own platform at [app.pixverse.ai](https://app.pixverse.ai).

**Official project:** https://app.pixverse.ai

## Use cases

- **Animate product packshots** — pass the packshot as `image_url` with a prompt such as "slow rotation, studio lighting" and a short `duration` for a listing video.
- **Social-feed clips from key art** — animate an illustration with `style` set to match, `generate_audio_switch` on for background music, and 9:16 output for reels.
- **Ad variations at scale** — enqueue one image with twenty prompt variants through the poll path and receive the clips by webhook.
- **Multi-shot teasers** — enable `generate_multi_clip_switch` so a single 15-second run cuts between camera angles instead of stitching several clips.
- **Storyboard previews** — turn each storyboard frame into a 3 to 5 second clip to check pacing before committing to a full production.
- **Reproducible A/B tests** — fix `seed` and vary only the prompt to measure how a wording change affects motion.

## FAQ

**Is there a PixVerse API?**

Yes. PixVerse offers a developer API on its own platform, and PixVerse V6 is also hosted on Synexa as `pixverse/pixverse-v6`. This client talks to the Synexa endpoint, which takes a first-frame image and a prompt and returns a video.

**How much does the PixVerse API cost through this client?**

The hosted `pixverse/pixverse-v6` endpoint is billed at $0.025 per run, where one run produces one clip. There is no subscription or credit pack; you pay per completed generation.

**Can I run PixVerse without a GPU?**

With this client, yes. Generation happens on Synexa's GPUs; your machine only needs Python and network access. PixVerse's models are not available for local use.

**Does this client work with the PixVerse app or ComfyUI?**

No. It does not log into the PixVerse app, use app credits or drive ComfyUI; it only calls the hosted endpoint. Templates and the editing timeline stay in the app.

**What input formats does it accept?**

`image_url` accepts .jpg, .png and .webp and is the first frame of the video. `prompt` is required and limited to 2048 UTF-8 bytes. Optional fields include `resolution`, `duration` (1 to 15 seconds), `style`, `negative_prompt`, `seed`, `generate_audio_switch`, `generate_multi_clip_switch` and `thinking_type`.

**Is this the official PixVerse SDK?**

No. This is an independent client that wraps the Synexa-hosted endpoint. PixVerse's official product and API are at https://app.pixverse.ai.

## Related

- [PixVerse](https://app.pixverse.ai) — official app and developer platform
- [Synexa Python client](https://github.com/synexa-ai/synexa-python) — the general-purpose SDK this client builds on
- [pixverse/pixverse-v6 on Synexa](https://synexa.ai/explore/pixverse/pixverse-v6) — the hosted PixVerse V6 endpoint
- [kling/kling-video-v3-pro on Synexa](https://synexa.ai/explore/kling/kling-video-v3-pro) — image-to-video with native audio and end-frame control
- [bytedance/seedance-2.5 on Synexa](https://synexa.ai/explore/bytedance/seedance-2.5) — text-to-video up to 30 seconds with synchronised audio

## License

MIT. This is an independent, community-maintained client and is not affiliated with or endorsed by the authors of PixVerse. Model weights and trademarks belong to their respective owners.

_Last reviewed: 2026-09-22_
