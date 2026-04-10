---
name: Capture Golden Reference Screenshots
assignee: qa-tester
project: new-game-kickoff
---

Capture "golden" reference screenshots of the game from an Art-Director-approved build. These become the deterministic similarity baseline for Layer 1 of `scripts/verify_art_direction.py`.

## What a golden capture is (and isn't)

- **Is**: an actual screenshot from the live game, taken at a deterministic moment with fixed FPS and seed, explicitly approved by the Art Director as representing the intended visual state for that scene.
- **Is not**: concept art, a mockup, or an aspirational render. Golden captures are what the game *currently* looks like when it's considered correct — so future verification runs can detect regressions against them.

## Prerequisites

- Art direction exists under `design/art/` (output of `create-testable-art-direction`).
- `scripts/verify_art_direction.py` exists and runs on the current project (output of `implement-art-direction-verification`).
- The game has a playable build that the Art Director considers a visual baseline — any scene with an "approved" look, even if gameplay is incomplete.

## Capture protocol

1. Check out the approved build commit. Record the commit SHA.
2. Clear any stale engine caches (e.g. for Godot: `rm -rf .godot/`).
3. Capture screenshots for every required scene type (see deliverables) using the engine's frame-capture mechanism. For Godot: `xvfb-run godot --write-movie <path> --fixed-fps 1 --quit-after N`. Use a fixed FPS and a fixed seed to get reproducible frames.
4. Review each capture visually. Send to Art Director for explicit approval before committing.
5. Commit to `design/art/golden_captures/` with a MANIFEST.

## Deliverables

Under `design/art/golden_captures/`:

- One PNG per major scene type (exact list depends on the game, but typically includes: main_menu, level_select or equivalent, primary gameplay scene initial state, primary gameplay scene mid-play, win/success state, fail/game-over state, pause overlay, settings, any cutscene or narrative moment).
- `MANIFEST.yaml` listing every capture with:
  ```yaml
  captures:
    - path: main_menu_initial.png
      scene: menu
      approved_by: Art Director
      approved_at: <date>
      commit_sha_of_build: <sha>
      notes: "why this specific frame represents the approved state"
  ```

After committing, run `scripts/verify_art_direction.py --captures design/art/golden_captures/ --style-guide design/art/` and confirm every golden capture passes all Layer 1 checks against itself (sanity check that the pipeline is consistent with the baseline).

## Honest caveats

- **Golden captures are a living asset.** When the art direction genuinely changes, golden captures must be re-taken and re-approved by the Art Director. Do not edit golden images manually.
- **Do not use placeholder captures from broken builds as golden.** If the build has visible bugs, fix the bugs first. The golden set anchors everything downstream — a broken baseline means every future check is broken.
- **Lighting and randomness matter.** Two runs of the same scene can produce visually different frames if the game has any randomness (e.g. particle systems, ambient animation). Seed the RNG or capture immediately before any random process starts.

## Depends on

- `create-testable-art-direction`
- `implement-art-direction-verification`

## Blocks

- `integrate-art-direction-verification`
