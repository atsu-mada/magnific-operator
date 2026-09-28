# Upscale

Observed 2026-09; verify live with `video_upscale_models_list`.

## Modes

| Mode | Character | Suits |
|---|---|---|
| `magnific` | Creative; adds texture and detail, may reinterpret | Organic textures, landscapes, stylized material (paper, fabric) |
| `magnific_precision` | Conservative; less hallucinated detail | Text, titles, logos, faces that must not drift |
| `topaz` | Faithful enhancement | Live-action-like footage, when fidelity beats added detail |

Test one short clip per mode on the most demanding shot before batching.

## Parameters and the enum trap

- `magnificResolution` enum: `720p`, `1k`, `2k`, `4k`. **`1080p` is not a valid value** — use `1k` for ~1920 wide.
- `targetResolution`: target pixel **width**.

## Off-by-one output sizes

Outputs can be 1 px (or a few px) short: e.g. 1280x719, 1920x1072. Normalize in edit:

```bash
ffmpeg -i in.mp4 -vf "scale=1920:-2:flags=lanczos,crop=1920:1080" -c:v libx264 -crf 16 -preset slow -c:a copy out_v001.mp4
```

(If the scaled height is below 1080, scale by height first: `scale=-2:1080,crop=1920:1080`.) Always ffprobe the result.

## Cost (observed 2026-09)

- 720p: ≈1,719 per 4 s clip, ≈2,075 per 5 s clip.
- 1k: ≈430 credits / s.

Quote with `simulate_cost` before every batch.

## Concurrency

The upscaler has its own concurrency/usage limit separate from generation. Submit in small batches, wait for each batch with `creations_wait`, then submit the next. Running upscales reserve credits — this can make a concurrent `spaces_run` fail for insufficient credits.

## Check for existing upscales first

Before any (re-)upscale:

1. `creations_search` with `toolNames` including the video upscaler, filtered to the relevant time range/source.
2. If a completed result exists for the same source and settings, `creations_register_download` and download it instead.
3. Only upscale what is genuinely missing.

Completed-but-undownloaded upscales were observed; re-running them wasted credits.
