# Moderation and retry

Observed 2026-09; verify live.

For the general Seedance filter vocabulary (risky verbs, IP-safe rewrites, safe substitutes by category) see **`seedance-studio`**. This file covers how blocks surface on Magnific and the retry procedure.

## What a block looks like

- `creations_wait` / `creations_get` returns status `failed` with the message "Prompt blocked by moderation rules".
- Blocked jobs were **not charged** in observed cases. Confirm with `account_balance` if the user cares.
- Other failures (timeouts, insufficient credits, invalid parameters) show different messages — do not reword prompts for those.

## Retry procedure

1. Report the block to the user in plain words (no internal ids).
2. Identify likely triggers: contact, penetration, violence, destruction, body-horror verbs; minors; real people; brand/IP names.
3. Reword with technique **T8** (below). Keep the event, timing, and end state identical; change only the wording.
4. If the parameters (and so the cost) are unchanged, a single retry can proceed under the original approval only if the user agreed to "retry once on block" up front. Otherwise ask.
5. Retry once. If blocked again, stop and propose alternatives (different framing, split the beat, remove the reference that may trigger it).

Do not loop retries automatically.

## T8 — moderation-safe rewording

Replace contact / penetration / destruction verbs with gentle, material metaphors that describe the same visual:

| Risky | Safer |
|---|---|
| "enters his body" | "overlaps and melts into him" |
| "pierces", "stabs through" | "passes gently through, like light through paper" |
| "collapses", "explodes" | "unravels into glowing paper confetti" |
| "crushes", "smashes" | "folds inward and settles" |
| "grabs her by the throat" | "reaches toward her; she steps back" |

Pattern: name the visual result (overlap, dissolve, unravel, fold) rather than the physical harm. Keep the material vocabulary consistent with the film's look (paper, light, cloth, water).

## Non-moderation failures

- **Insufficient credits on `spaces_run`** while upscales hold reserved credits: wait for upscales to finish, or run the same generation directly with `video_generate`.
- **Parameter errors**: re-check enums (`video_models_show`, `video_upscale_models_list`) — e.g. `1k`, not `1080p`.
