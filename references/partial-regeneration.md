# Partial regeneration on Magnific (T7)

Frame-matched partial regeneration: keep the accepted take, regenerate only the flawed head or tail segment, and splice at a matched frame. Observed cost ≈1/4 of a full re-roll (e.g. replacing the first 7 s of a 24 s take; observed 2026-09).

The general retake protocol (when to partial-regen vs re-roll, prompt wording) is in `seedance-studio`. This file is the Magnific procedure.

## When to use

- The flaw is confined to the start or end of an otherwise accepted take.
- The remaining segment has a clean frame where motion can continue.

## Procedure (head replacement example)

Accepted take `shot_v003.mp4`; flaw in 0–7 s; splice at 7.000 s.

1. **Extract the splice frame** from the accepted take:

   ```bash
   ffmpeg -ss 7.000 -i shot_v003.mp4 -frames:v 1 -q:v 1 splice_frame_7000.png
   ```

   For exact frame selection, compute from fps: frame N at `N / fps`. Check with `ffprobe -show_entries stream=r_frame_rate`.

2. **Upload** the frame: `creations_request_upload` (`mimeType: image/png`) → PUT once → `creations_finalize_upload`.

3. **Optional new start frame**: if the new segment should open differently, create one with `images_generate` using the splice frame (and character sheets) as references (≈75 credits/image). Get user sign-off on the still.

4. **Quote and approve**: `simulate_cost` for `video_generate` with keyframes; quote; wait for OK.

5. **Generate** with `bytedance-seedance-pro-2.5`, keyframes `start` = new start frame (optional), `end` = splice frame, duration = segment length (7 s). No references in the same job (exclusive with keyframes). Prompt describes only the segment's motion ending exactly in the splice frame's pose and camera.

6. **Retrieve**: `creations_wait` loop → `creations_register_download` → save as `shot_head_v001.mp4`.

7. **Splice** at the matched frame. Trim the new head to exactly the segment length and the accepted take from the splice time:

   ```bash
   # Drop the new segment's last frame if it duplicates the splice frame
   ffmpeg -i shot_head_v001.mp4 -t 7.000 -c:v libx264 -crf 14 -preset slow -c:a aac -b:a 192k head.mp4
   ffmpeg -ss 7.000 -i shot_v003.mp4 -c:v libx264 -crf 14 -preset slow -c:a aac -b:a 192k tail.mp4
   ffmpeg -i head.mp4 -i tail.mp4 -filter_complex \
     "[0:v][0:a][1:v][1:a]concat=n=2:v=1:a=1[v][a]" \
     -map "[v]" -map "[a]" -c:v libx264 -crf 14 -preset slow -c:a aac -b:a 192k shot_v004.mp4
   ```

   Re-encode rather than stream-copy concat: stream-copied AAC segments drift (observed up to ~0.6 s). Match fps/resolution first (`-vf fps=24,scale=...`) if the segments differ.

8. **Verify**: ffprobe total duration equals the original; step through ±0.3 s around the splice for pops, jumps, or colour shift. If a colour shift is visible, grade the new segment to match before splicing.

For tail replacement, mirror it: splice frame becomes the `start` keyframe, and the new segment is appended.

## Notes

- Keep the accepted take untouched; every output gets a new version number.
- Upscale after splicing, using the same mode/settings as the rest of the edit, or upscale both segments identically before splicing.
