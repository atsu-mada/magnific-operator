# Cost estimation and approval

All rates are **observed 2026-09; verify live** with `simulate_cost`. Pricing changes without notice; treat these only as sanity checks.

## Observed rates

| Job | Observed cost |
|---|---|
| Seedance 2.5, 720p | ≈440 credits / s |
| Seedance 2.5, 480p | ≈200 credits / s |
| Seedance 2.5 keyframe (first/last frame), 7 s, 720p | 3,080 (matched estimate) |
| Image edit (`images_generate` with references) | ≈75 credits / image |
| Video upscale 720p | ≈1,719 / 4 s clip, ≈2,075 / 5 s clip |
| Video upscale 1k | ≈430 credits / s |

## Known estimate gap: video reference

When a job includes a **video reference** (`@Video 1`), the actual charge was about **18% above** `simulate_cost`:

| Duration | `simulate_cost` | Charged |
|---|---|---|
| 24 s, 720p | 10,560 | 12,480 |
| 22 s, 720p | 9,680 | 11,440 |

Keyframe jobs and image-reference-only jobs matched the estimate in the same period.

Quote both numbers: "estimate 10,560; with video reference expect ~12,500".

## Gate: quote → approve → run

1. Build the exact call parameters.
2. `simulate_cost` (or `simulate_spaces` for a canvas run). For several jobs, sum them.
3. Apply known gaps (video reference +~18%).
4. `account_balance`: confirm the adjusted total fits, leaving room for reserved credits held by running upscales.
5. Tell the user: jobs, parameters, estimate, adjusted estimate, whether the balance covers it. Do not publish the balance in any shared artifact.
6. Wait for an explicit OK in chat for **this** batch.
7. Run. Afterwards, report quoted vs charged if visible.

Re-rolls, retries after moderation, extra upscales, and partial regenerations are **new batches** — re-quote and re-approve.

## Cost-saving choices

- Fix a flawed head/tail with partial regeneration (`partial-regeneration.md`) — observed about 1/4 the cost of a full re-roll.
- Iterate at 480p, finalize at 720p.
- Search existing upscales before upscaling again (`upscale.md`).
- Upscale only the takes that survive the edit.
