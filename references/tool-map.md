# Magnific MCP tool map

Tool names as exposed by the Magnific MCP server (observed 2026-09; verify live — call only tools present in the client's `tools/list`). "Paid" means it spends credits and needs the approval gate.

Every response may carry an `instruction` field: follow it. Never regenerate queued creations. Never show internal identifiers to the user.

## Account and estimation (read-only)

| Tool | Use when |
|---|---|
| `account_balance` | Before quoting any paid batch; check that balance covers the adjusted estimate. Do not repeat the balance in public outputs. |
| `simulate_cost` | Estimate one generation/upscale call with the exact parameters you intend to send. |
| `simulate_spaces` | Estimate a Spaces canvas run before `spaces_run`. |
| `simulate_flows` | Estimate a saved flow before `flows_run`. |

## Model discovery (read-only)

| Tool | Use when |
|---|---|
| `video_models_list`, `video_models_show` | Confirm slug, durations, resolutions, reference slots before a video job. |
| `images_models_list`, `images_models_show`, `images_models_settings` | Choose an image model and its parameters. |
| `video_upscale_models_list` | Confirm upscale modes and resolution enum. |
| `images_upscale_modes_list`, `images_upscale_presets_list` | Image upscale options. |

## Generation (paid)

| Tool | Use when |
|---|---|
| `video_generate` | T2V / I2V / R2V / keyframe video, including Seedance 2.5. |
| `images_generate` | Text-to-image; image edits when references are passed (e.g. a new start frame for partial regeneration). |
| `images_expand`, `images_relight`, `images_restyle`, `images_variations`, `images_change_camera`, ... | Targeted image edits. |
| `video_extend` | Extend an accepted clip. |
| `video_hdr` | Any HDR request (never `video_upscale` for HDR). |

## Post-processing (paid)

| Tool | Use when |
|---|---|
| `video_upscale` | Video upscale; see `upscale.md`. |
| `images_upscale` | Image upscale. |
| `video_concatenate` | Server-side join; local ffmpeg splice is usually more precise for frame-matched cuts. |

## Files and creations

| Tool | Use when |
|---|---|
| `creations_request_upload` | Start a local-file upload; needs `mimeType`; returns a presigned PUT URL. PUT once. |
| `creations_finalize_upload` | Finish the upload (`path` or `uploads[]`); returns a creation usable as a reference. |
| `creations_upload_file`, `creations_upload_image` | Alternative upload paths if exposed; prefer the request/finalize pair for large video. |
| `creations_show` | Inline preview for UI-capable clients right after generation. |
| `creations_wait` | Poll 1–8 ids, long-poll ≤25 s; loop until terminal. Gives the final asset URL. |
| `creations_get` | Fetch one creation's details/URL. |
| `creations_search` | Find existing results, e.g. prior upscales (`toolNames` including the video upscaler) before re-running. Data-only. |
| `creations_register_download` | Call before saving any file locally. |

## Spaces (canvas)

| Tool | Use when |
|---|---|
| `spaces_list`, `spaces_show`, `spaces_state`, `spaces_get_nodes` | Inspect an existing Space the user named. |
| `spaces_create` | New Space for a multi-node pipeline. |
| `spaces_edit` | Add/modify nodes; query ≤4000 chars; one node per edit; check `spaces_edit_status`. |
| `spaces_add_creations` | Place existing creations on the canvas. |
| `spaces_run`, `spaces_run_status` | Run (paid) after `simulate_spaces` + approval. If it fails for insufficient credits while upscales hold reservations, fall back to direct `video_generate`. |

## Not used by this skill without explicit user request

`folders_delete`, `library_delete`, `library_share`, `tags_delete`, `creations_deliver`, `creations_move`, `projects_move`, stock downloads, dubbing confirm. Refuse destructive/sharing calls and ask the user to do them.
