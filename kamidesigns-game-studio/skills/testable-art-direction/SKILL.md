---
name: testable-art-direction
description: "Author an art direction document that is MACHINE-VERIFIABLE, not just inspirational. Extends /art-bible with a four-band testability taxonomy, a machine-readable palette, a binary rubric, and a curated reference image set. Use when starting a new game project and you need the art direction to be enforceable by automated verification pipelines, not just human reviews."
argument-hint: "[--extend-existing | --from-scratch] [--project <slug>]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, Bash, AskUserQuestion
---

# Testable Art Direction

This skill is the testable companion to `/art-bible`. It exists because the default
art bible skill produces a prescriptive prose document that humans can follow — but
automated pipelines cannot. Research (VideoGameQA-Bench, arXiv 2505.15952) shows
vision-language models are currently advisory-grade for visual QA, so any art
direction that is auto-verified must be written in a way that deterministic tools
can catch most of the work and the VLM only judges the genuinely subjective parts.

**Non-goal:** this is not a replacement for `/art-bible`. It is a strict superset.
Run `/art-bible` first if you do not already have an art bible draft. This skill
then either extends it with the testability additions, or builds it from scratch
if the project has no art bible yet.

## The Four-Band Testability Taxonomy

Every rule in the art direction must be assigned to exactly one band. Band tags
are the contract with downstream verification. Without them, nothing can be
automated.

| Band | Example | Verified by |
|---|---|---|
| **1_numerical** | "Primary palette must match these 6 hex values within ΔE ≤ 10" | Code (deterministic) |
| **2_structural** | "Hero character occupies 15–30% of frame height", "No more than one saturated hue per frame" | Code (computer vision) |
| **3_holistic** | "Silhouette must be readable against a flat background", "HUD density must not exceed combat readability threshold" | VLM rubric (advisory) |
| **4_aspirational** | "Feels heroic and melancholy", "Captures the atmosphere of a dying empire" | Human only — NOT auto-tested |

Band 4 rules **must** be tagged `[band: 4_aspirational — human-only]` so that
`/art-direction-verify` knows to skip them entirely. If you cannot decide which
band a rule belongs to, it belongs in band 3 or 4. Be conservative — over-promising
testability is worse than under-promising.

## Phase 0: Precondition Check

Read `design/gdd/game-pillars.md`. If it does not exist, fail with:

> "No game pillars found. Run `/brainstorm` or manually author `design/gdd/game-pillars.md` first — an art direction cannot be tested without pillars to anchor the rules to."

Read `design/gdd/game-concept.md` if present. Extract:
- Game title
- Core fantasy / elevator pitch
- Visual identity anchor (if present from brainstorm output)
- Target platform

Read `design/art/art-bible.md` if it exists. If it does, set `mode = extend`.
If not, set `mode = from-scratch` and plan to spawn `/art-bible` inline first,
then layer testability on top.

## Phase 1: Directory Skeleton

Create the following directory structure (idempotent — do not overwrite):

```
design/art/
├── art-bible.md             # managed by /art-bible, extended by this skill
├── palette.json             # NEW — machine-readable palette
├── rubric.yaml              # NEW — binary verification rubric
├── thresholds.yaml          # NEW — tolerances for deterministic checks
├── reference/               # NEW — approved visual references (PNGs)
│   └── .gitkeep
└── golden_captures/         # NEW — populated by QA over time
    └── .gitkeep
```

If `mode = extend`, check which of these already exist. Only create what is
missing. Never overwrite `art-bible.md` — the existing `/art-bible` skill owns
that file and is gated by AD-ART-BIBLE sign-off.

## Phase 2: Tag Existing Art Bible Rules

If `mode = extend`, read `design/art/art-bible.md` and for every concrete,
declarative sentence that expresses a rule or constraint, propose a band tag.

Present the list of un-tagged rules to the Art Director via `AskUserQuestion`:

> "I found [N] rules in the existing art bible. Here are the first 10. For each,
> select the testability band. Rules in band 1–3 will be auto-verified. Band 4
> rules will be marked as human-only."

Batch the questions 10 at a time to avoid overwhelming the widget. After the
user decides, rewrite the art bible sentences with inline band tags:

```markdown
Primary palette: cool blues and warm amber, desaturated (ΔE < 10 tolerance). [band: 1_numerical]

The player character must have a readable silhouette against any background at
thumbnail size. [band: 3_holistic]

The world should feel lonely but not bleak. [band: 4_aspirational — human-only]
```

Write the updated art bible back to file. **Record original content in
`design/art/.art-bible-original.md`** before any edits, so the sign-off gate
can still diff.

If `mode = from-scratch`, spawn `/art-bible` via Task first, wait for its
Creative Director sign-off, then return and tag the fresh bible.

## Phase 3: Author `palette.json`

Spawn `art-director` via Task with the tagged art bible. Ask:

> "Extract the Color System section into a machine-readable JSON palette. For
> each semantic color role (primary, secondary, accent, danger, safe, neutral,
> background, etc. — whatever the game uses), provide: the hex value, the ΔE
> (CIEDE2000) tolerance, and a one-line intent description. Include a `version`
> field and a `scenes` object that specifies which roles apply to which scene
> types (e.g., menu, combat, cutscene). Use tolerances between 5 and 20 — tighter
> for hero colors, looser for environmental backgrounds."

Expected shape:

```json
{
  "version": "1.0.0",
  "default_tolerance_delta_e": 10,
  "roles": {
    "primary": {
      "hex": "#1a3a5c",
      "tolerance_delta_e": 8,
      "intent": "hero blue — carries player identity, used in UI accents and primary VFX"
    },
    "danger": {
      "hex": "#c23030",
      "tolerance_delta_e": 5,
      "intent": "threat / damage — never used for non-hostile elements"
    }
  },
  "scenes": {
    "menu": ["primary", "neutral", "background"],
    "combat": ["primary", "danger", "safe", "accent"]
  }
}
```

Write to `design/art/palette.json`. Validate that it is well-formed JSON.

## Phase 4: Author `thresholds.yaml`

This file holds the numeric thresholds for Band 1 and Band 2 rules. Spawn
`technical-artist` via Task with the tagged art bible and palette. Ask:

> "Produce a thresholds.yaml that defines the deterministic tolerances used by
> the verification pipeline. Cover: palette ΔE defaults (per role if needed),
> WCAG contrast minimums for UI text (normal and large), CLIP similarity floor
> for golden-capture comparison (0.80 is typical for same-scene matches),
> character-size bands (min/max fraction of frame height for hero characters,
> per scene type), and any other numerical constraint present in Band 1 or 2
> rules from the art bible."

Expected shape:

```yaml
version: 1.0.0
palette:
  default_delta_e: 10
ui_contrast:
  normal_text_min: 4.5
  large_text_min: 3.0
clip_similarity:
  golden_capture_min: 0.80
character_size:
  hero:
    min_frame_fraction: 0.15
    max_frame_fraction: 0.30
    applies_to: [combat, cutscene]
```

Write to `design/art/thresholds.yaml`. Validate YAML.

## Phase 5: Author `rubric.yaml`

This is the most important output of the skill. It is the binary checklist that
the VLM answers. Spawn `art-director` via Task with the tagged art bible.
Ask:

> "For every Band 3 (holistic) rule in the art bible, produce one rubric.yaml
> entry. Each entry is a SINGLE yes/no question (binary — never scalar), cites
> at least one reference image by filename, and specifies which scene types it
> applies to. If a rule cannot be phrased as a single yes/no question, split it
> into multiple rubric items. Order by severity: blockers first, warnings last.
> Aim for 5–15 rubric items total — more than 15 creates noise, fewer than 5
> probably means the art bible is under-specified."

Expected shape:

```yaml
version: 1.0.0
items:
  - id: hero_silhouette_readable
    band: 3_holistic
    question: "Is the player character silhouette clearly distinguishable from the background, as in reference/approved_combat.png?"
    references:
      - reference/approved_combat.png
      - reference/approved_menu.png
    applies_to_scenes: [combat, exploration]
    severity: blocker

  - id: hud_saturation_restraint
    band: 3_holistic
    question: "Is there at most ONE element in the HUD with saturation above 80%, as in reference/approved_combat_hud.png?"
    references:
      - reference/approved_combat_hud.png
    applies_to_scenes: [combat]
    severity: warning
```

Required fields per item: `id`, `band` (must start with `3_`), `question` (phrased
as yes/no), `references` (at least one), `applies_to_scenes`, `severity` (one of
`blocker`, `warning`, `info`).

Write to `design/art/rubric.yaml`. Validate that every rubric item's `references`
list points to files that exist (or are about to exist after Phase 6).

## Phase 6: Curate Reference Images

Reference images are the ground truth that the VLM and CLIP similarity checks
compare against. Without them, the pipeline is blind.

Use `AskUserQuestion` to ask the Art Director:

> "The rubric cites [N] reference images. Do you have approved visual references
> (concept art, moodboard images, mockups) for these? Either provide the files
> for me to copy into `design/art/reference/`, or run this skill again after
> you have them."

Required per major scene type (at minimum):
- 1 image for `menu`
- 1 image for `combat` or the primary gameplay scene
- 1 image for `cutscene` / narrative moments (if applicable)
- 1 UI-focused close-up per scene type

If no references exist yet and the project has no visual output, the skill can
**temporarily** accept "placeholder" references from the first playable build
and mark them `status: placeholder` in a new `design/art/reference/MANIFEST.yaml`.
Placeholder references must be replaced before the project exits pre-production —
emit a `story-readiness` style warning.

`MANIFEST.yaml` shape:

```yaml
references:
  - path: approved_main_menu.png
    status: approved   # or: placeholder
    approved_by: Art Director
    approved_at: 2026-04-15
    scene: menu
    notes: "Approved mockup from concept art session 2026-04-10"
```

## Phase 7: Creative Director Sign-Off Gate

**Review mode check** — apply before spawning the gate:
- `solo` → skip. Note in the report and proceed.
- `lean` → skip (not a phase gate).
- `full` → spawn as normal.

If `full` mode, spawn `creative-director` via Task using a new gate
**CD-TESTABLE-ART**. Pass: art bible path, palette.json, rubric.yaml,
thresholds.yaml, reference manifest.

Creative Director verdict options: `APPROVED`, `CONCERNS (accepted)`, `REVISE`.

Record the verdict in a status header at the top of `design/art/art-bible.md`:

```markdown
> **Testable Art Direction Sign-Off (CD-TESTABLE-ART)**: APPROVED 2026-04-15
> **Rubric version**: 1.0.0
> **Palette version**: 1.0.0
```

## Phase 8: Close

Emit the final artifact summary:

```
design/art/art-bible.md      (tagged with [band: N] markers)
design/art/palette.json      (N color roles, version 1.0.0)
design/art/rubric.yaml       (N rubric items, version 1.0.0)
design/art/thresholds.yaml   (numeric tolerances)
design/art/reference/        (N approved, M placeholder)
design/art/golden_captures/  (empty — populated by /art-direction-verify)
```

Use `AskUserQuestion` for next steps. Include these options:
- `[A] Run /art-direction-verify against existing screenshots (advisory mode)` — recommended if any game frames exist
- `[B] Capture golden reference screenshots from current build` — hand off to QA Tester
- `[C] Stop here — art direction is ready`

## What this skill does NOT do

- It does not write verification code (that's `/art-direction-verify` + Technical Artist)
- It does not run the VLM rubric (that's `/art-direction-verify`)
- It does not create or approve golden captures (that's QA Tester's domain)
- It does not override the existing `/art-bible` skill's authoring flow — it extends it

## Related skills

- `/art-bible` — run first if no art bible exists. This skill extends it.
- `/art-direction-verify` — consumes the outputs of this skill at runtime
- `/asset-spec` — per-asset visual specs, runs after this skill
- `/consistency-check` — cross-GDD text consistency, unrelated but complementary
