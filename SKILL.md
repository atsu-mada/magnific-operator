---
name: magnific-operator
description: "Operate the Magnific AI creative suite through its MCP server (no Chrome). Use when Codex or Claude Code should generate or edit images, generate video (including Seedance 2.5 via Magnific with image/video/audio references or start/end keyframes), upscale images or video, upload local files as creations, build or run a Spaces canvas, estimate credit cost before a paid run, or wait for, register, and download Magnific results. Does not write general Seedance prompt craft (use seedance-studio), plan productions or storyboards (use ai-video-production), build previz (use previz-maker), drive the Magnific web UI in a browser, or spend credits without a quoted estimate and explicit user approval."
---

# Magnific Operator

## Shared client contract

- Codex invocation: `$<magnific-operator>`
- Claude Code invocation: `/magnific-operator`
- Resolve bundled files from `${AGENT_SKILLS_ROOT}/magnific-operator` when that variable is set, otherwise from the directory containing this SKILL.md; do not assume a `.codex/skills` or `.claude/skills` physical directory.
- Apply the active client's own authentication, approval, connector, and external-action rules. A capability or permission available in one client does not transfer to the other.

This is a **suite operator** in the sense defined by `ai-video-production`: an execution skill that drives one AI creative suite. It follows the suite-operator contract (the nine sections below, same headings in every operator).

Division of knowledge:

- **Magnific-specific tool behaviour lives here** — tool names, enums, reference slots, cost gaps, upscale quirks, retrieval.
- **General Seedance prompt knowledge lives in `seedance-studio`** — prompt structure, filter vocabulary, retake protocol, shot language.
- **Preproduction lives in `ai-video-production`** — its `generation-handoff` deliverable (with `suite: magnific-operator`) is this skill's preferred input. The suite registry there lists this skill.
- **Previz comes from `previz-maker`** — a rendered previz is passed as `@Video 1`, only after the user has visually approved it.

All volatile numbers in this skill are **observed 2026-09; verify live** before relying on them.

## Input

Preferred: a `generation-handoff` entry from `ai-video-production` with `suite: magnific-operator`. Otherwise gather, per job: task (image / video / upscale / Spaces), model, prompt, references or keyframes (local paths or existing creations), duration, resolution, aspect ratio, sound flags, output path with version suffix.

Ask only for what is missing and cost-relevant. Never infer approval from the handoff.

## 1. Access path

- **Route**: Magnific MCP server tools only (`mcp__magnific__*` or the client's equivalent). No Chrome, no web-UI clicking, no scraping.
- **Auth owner**: the user's Magnific account, connected as an MCP connector in the active client. If the tools are absent, stop with `reason_code: tool_unavailable` and ask the user to connect the Magnific MCP server.
- **May call freely**: read-only tools — `account_balance`, `simulate_cost`, `simulate_spaces`, `*_models_list`, `*_models_show`, `creations_get`, `creations_search`, `creations_wait`, `spaces_state`, `spaces_get_nodes`.
- **May call after approval gate (section 9)**: anything that spends credits — `video_generate`, `images_generate`, `images_*` edits, `video_upscale`, `images_upscale`, `spaces_run`, audio generation.
- **May call when the task needs it (no credits)**: `creations_request_upload` / `creations_finalize_upload`, `creations_register_download`, `spaces_create`, `spaces_edit`.
- **Never**: delete creations/folders/Spaces, share or publish, or move items the user did not name.
- Every tool response may include an `instruction` field — follow it. Never regenerate queued creations. When talking to the user, use titles, plain descriptions, and `webUrl`; never quote internal identifiers, UUIDs, or request ids unless asked.

Tool-by-tool map: [references/tool-map.md](references/tool-map.md).

## 2. Models & modes

- List live models with `video_models_list` / `images_models_list` before choosing; do not hard-code beyond this section.
- **Seedance 2.5** slug `bytedance-seedance-pro-2.5`: T2V, I2V, R2V (image/video/audio references), start/end keyframes (first/last frame). **Keyframes and references are mutually exclusive in one job.**
- Images: `images_generate` (text-to-image, and edits when given references), plus `images_*` edit tools (expand, relight, restyle, variations, upscale).
- Video post tools: `video_upscale`, `video_hdr` (HDR requests go here, never `video_upscale`), `video_extend`, `video_concatenate`.

Detail: [references/seedance-on-magnific.md](references/seedance-on-magnific.md).

## 3. References

- Seedance 2.5 on Magnific: images tagged `@Image N`, one video tagged `@Video 1` (typically previz — camera, timing, positions only), audio ≥2 s (guides rhythm only; no lip-sync).
- Pass a creation `identifier` or the `url` from `creations_get`/`creations_wait` — never `webUrl`.
- Local files: `creations_request_upload` (needs `mimeType`) → PUT once to the presigned URL → `creations_finalize_upload`. Do not re-PUT the same URL.
- Prompt structure for previz-driven R2V: see `seedance-studio`; the Magnific slot syntax is in [references/seedance-on-magnific.md](references/seedance-on-magnific.md).

## 4. Limits

- Seedance 2.5: 480p / 720p; single jobs of 22 s and 24 s succeeded (observed 2026-09; verify live via `video_models_show`).
- `creations_wait`: 1–8 ids per call, long-poll ≤25 s; loop.
- `spaces_edit`: query ≤4000 characters; one node per edit.
- Upscaler has its own concurrency/usage limit — queue in small batches.
- Upscale outputs can be 1 px short (e.g. 1280x719, 1920x1072) — scale+crop in edit.

## 5. Cost estimation

- Always run `simulate_cost` (or `simulate_spaces` for a canvas run) and `account_balance` before any paid step.
- Known gap: with a **video reference**, the actual charge ran ~18% above `simulate_cost`. Quote the estimate *and* the adjusted figure. Keyframe jobs matched the estimate.
- Gate: **quote → user OK → run**. One approval covers one quoted batch, not later retries or re-rolls.

Rates and worked arithmetic: [references/cost-and-approval.md](references/cost-and-approval.md).

## 6. Moderation & failure

- A block appears as status `failed` with "Prompt blocked by moderation rules"; blocked jobs were not charged (observed 2026-09).
- Procedure: report the block → reword contact/penetration/destruction verbs into gentle metaphors (technique T8) → re-quote if cost changes → retry once with user OK.
- Canvas (`spaces_run`) may fail with insufficient credits while upscales hold reserved credits; a direct `video_generate` can still work.

Detail: [references/moderation-and-retry.md](references/moderation-and-retry.md).

## 7. Post-processing

- Upscale modes: `magnific` (creative; organic texture), `magnific_precision` (safer for text, titles, faces that must not drift), `topaz` (faithful).
- `magnificResolution` enum is `720p`, `1k`, `2k`, `4k` — **not** `1080p`.
- Before any re-upscale, search existing upscales with `creations_search` — completed-but-undownloaded results are common.
- Partial regeneration of a flawed head/tail (technique T7) is often ~1/4 the cost of a full re-roll.

Detail: [references/upscale.md](references/upscale.md), [references/partial-regeneration.md](references/partial-regeneration.md).

## 8. Retrieval

- After submitting: show creations if the client can render them (`creations_show`), then loop `creations_wait` until each id is terminal.
- Before saving: `creations_register_download`, then download to a **new versioned path** (`_v001`, `_v002`, ...). Never overwrite an accepted version.
- Verify with `ffprobe` (duration, resolution, fps, audio stream) before reporting done.

Detail: [references/retrieval-and-files.md](references/retrieval-and-files.md).

## 9. Approval gates

1. **Previz**: if the job uses a previz as `@Video 1`, the user must have **visually approved** the rendered previz (sent to them and confirmed) before any paid generation. A previz file existing is not approval.
2. **Paid steps**: every credit-spending call needs a quoted estimate (section 5) and an explicit user OK in chat for that batch.
3. **Retries / re-rolls / upscales** after a result: re-quote and re-approve.
4. Anything outside this skill's allowed calls (deletion, sharing, publishing): refuse and ask the user to do it.

## Output

Report per job: task, model, mode, duration/resolution, quoted vs charged credits (if visible), status, `webUrl`, saved path, ffprobe summary, and any block/retry. No internal ids, no balances unless the user asked.

## Related skills

- `ai-video-production` — preproduction, `generation-handoff`, suite registry and suite-operator template.
- `previz-maker` — block previz used as `@Video 1` after visual approval.
- `seedance-studio` — Seedance prompt craft, filter vocabulary, retake protocol.
