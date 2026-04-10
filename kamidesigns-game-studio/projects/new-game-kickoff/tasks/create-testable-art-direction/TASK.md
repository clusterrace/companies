---
name: Create Testable Art Direction
assignee: art-director
project: new-game-kickoff
---

Author an art direction document that is machine-verifiable, not just inspirational. Extends the standard art bible with a four-band testability taxonomy, a machine-readable palette, a binary rubric, and a curated reference image set. Run the `/testable-art-direction` skill to produce the deliverables.

## Why "testable"

Research (VideoGameQA-Bench, 2025) shows no current vision-language model is production-ready for autonomous art QA. For automated verification to work, every rule in the art direction must be tagged with one of four bands:

- **Band 1 — Numerical**: verified by code (palette ΔE, WCAG contrast)
- **Band 2 — Structural**: verified by code (element sizes, layout fractions)
- **Band 3 — Holistic**: verified by VLM rubric (advisory only)
- **Band 4 — Aspirational**: human-only, explicitly NOT auto-tested

Rules in bands 1–3 drive downstream automated checks. Band 4 rules are marked `[band: 4_aspirational — human-only]` so the verification pipeline knows to skip them.

## Prerequisites

- Game pillars must exist at `design/gdd/game-pillars.md` (output of the `define-game-pillars` task). The skill will fail fast without them.

## Deliverables

All under `design/art/`:

- `art-bible.md` — tagged with `[band: N]` markers on every declarative rule
- `palette.json` — machine-readable palette (hex, ΔE tolerances, per-role intent, per-scene role lists)
- `thresholds.yaml` — numeric tolerances (contrast minimums, size bands, similarity floors)
- `rubric.yaml` — binary (yes/no) VLM rubric items, each citing at least one reference image and listing `applies_to_scenes`
- `reference/*.png` — approved visual references (concept art, mockups, or approved captures)
- `reference/MANIFEST.yaml` — approval metadata per reference image
- `golden_captures/` — initially empty, populated later by QA

## Protocol

1. Verify `design/gdd/game-pillars.md` exists and is committed.
2. Run the `/testable-art-direction` skill. It will either extend an existing `art-bible.md` or author one from scratch. It spawns sub-agents (`art-director`, `technical-artist`) for each section.
3. Every rule gets a band tag. Band 4 rules are marked human-only.
4. Sign-off: Creative Director via CD-TESTABLE-ART gate (or skipped in solo/lean review mode).
5. Commit all artifacts. Push to origin. Reference this task in the commit message.

## Out of scope

- Building the verification pipeline (handoff to `technical-artist` in the `implement-art-direction-verification` task).
- Capturing golden reference screenshots (handoff to `qa-tester` in the `capture-golden-reference-screenshots` task).
- Running any VLM (that's the verification pipeline's job).

## Honest caveat

Not every rule in a game's art direction is automatable. Be ruthless about pushing rules toward bands 1–3 when possible (e.g., rephrase "feels tense" → "cool color temperature dominates, max 2 saturated warm hues") but accept that some rules genuinely belong in band 4 and can only be human-reviewed. Over-promising testability is worse than under-promising.

## Blocks

- `implement-art-direction-verification`
- `capture-golden-reference-screenshots`
- `integrate-art-direction-verification`
