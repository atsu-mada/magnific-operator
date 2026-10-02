# Retrieval and files

Observed 2026-09; verify live.

## Uploads

1. `creations_request_upload` with the correct `mimeType` (`image/png`, `video/mp4`, `audio/wav`, ...). Returns a presigned PUT URL.
2. PUT the file once:

   ```bash
   curl -sS -X PUT -H "Content-Type: video/mp4" --upload-file previz_v002.mp4 "$PRESIGNED_URL"
   ```

   Do not re-PUT the same URL; request a new one on failure.
3. `creations_finalize_upload` (`path` or `uploads[]`). Use the returned creation `identifier` as a reference.

Never place presigned URLs, identifiers, or local absolute paths in user-facing reports.

## Wait loop

- Right after submission: `creations_show` for inline preview if the client renders UI; text-only clients share `webUrl`.
- `creations_wait` accepts 1–8 identifiers and long-polls ≤25 s. Loop until every id is terminal (completed / failed). Batch in groups of 8.
- Never re-submit a queued or processing creation. A slow job is not a failed job.
- On `failed`, read the message: moderation → `moderation-and-retry.md`; credits → `cost-and-approval.md`.

## Register and download

1. `creations_register_download` for each creation **before** saving.
2. Download the final asset URL (from `creations_wait` / `creations_get`, not `webUrl`) to a versioned path:

   ```bash
   curl -sSL -o "renders/shot03_seedance_720p_v002.mp4" "$ASSET_URL"
   ```

## Versioned naming

`<shot>_<tool-or-model>_<res>[_<variant>]_vNNN.<ext>`, e.g. `shot03_seedance_720p_v002.mp4`, `shot03_upscale-1k-precision_v001.mp4`.

- Never overwrite an accepted version; always bump.
- Keep a sidecar note (prompt, parameters, quoted/charged credits) next to the file if the project has a manifest; `ai-video-production` defines the handoff/manifest format.

## Verify

```bash
ffprobe -v error -show_entries format=duration:stream=codec_type,width,height,r_frame_rate \
  -of default=nw=1 shot03_seedance_720p_v002.mp4
```

Check: expected duration (±1 frame), resolution (watch for 1 px short upscales), fps, audio stream present/absent as requested. Report the summary with the saved path.
