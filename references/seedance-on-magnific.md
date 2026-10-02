# Seedance 2.5 on Magnific

Facts observed 2026-09; verify live with `video_models_show` before each project.

General Seedance prompt craft (shot language, event density, filter vocabulary, retake protocol, IP-safe rewrites) belongs to **`seedance-studio`**. This file covers only how Magnific exposes the model.

## Slug and modes

- Slug: `bytedance-seedance-pro-2.5`.
- Resolutions: `480p`, `720p`. Upscale afterwards (see `upscale.md`).
- Modes:
  - T2V — prompt only.
  - I2V — one image as start.
  - R2V — references (images / one video / audio).
  - Keyframes — `start` and/or `end` frame.
- **Exclusive**: keyframes (start/end) and references cannot be combined in one job. Choose one:
  - Need exact first/last frame (e.g. partial regeneration splice) → keyframes.
  - Need identity/style/motion guidance from several assets → references.

## Durations

- Single jobs of 22 s and 24 s succeeded (observed 2026-09).
- Short keyframe jobs (e.g. 7 s) are the cheap path for fixing one segment.

## References and tagging

| Type | Tag in prompt | Notes |
|---|---|---|
| Image | `@Image 1`, `@Image 2`, ... | Character, costume, location, style sheets. Order = upload/argument order. |
| Video | `@Video 1` | Typically a previz from `previz-maker`: camera, timing, positions only — never its colours or proxy shapes. User must have visually approved it. |
| Audio | (audio reference slot) | ≥2 s. Guides rhythm/tempo only; no lip-sync. |

Pass each reference as a creation `identifier` (from upload/finalize or a previous generation) or the `url` from `creations_get` / `creations_wait` — never `webUrl`.

Proxy-shape transfer risk: previz boxes and glow spheres can be copied literally. State the real form in the prompt ("remain human-shaped, not spheres"). See `seedance-studio`.

## Sound flags

- `withSoundEffects: true` + `noMusic: true` — keeps diegetic SFX, no generated score. Use when music is added in edit.
- Omit sound effects when audio will be fully replaced.

## Prompt structure pointer

For previz-driven R2V a four-block structure worked well: reference roles / intent / timed events with end states / fixed constraints. The canonical template and wording rules live in `seedance-studio`; do not duplicate them here.

## Pre-submit checklist

1. `video_models_show` confirms slug, duration, resolution.
2. References uploaded and finalized; tags in prompt match argument order.
3. No keyframes if references are present (and vice versa).
4. Sound flags set deliberately.
5. `simulate_cost` run with the exact parameters; add the video-reference gap (see `cost-and-approval.md`).
6. Previz visually approved; quote approved.
