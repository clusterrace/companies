---
name: art-direction-verify
description: "Verify that rendered game screenshots match the testable art direction. Runs deterministic checks (palette ΔE, WCAG contrast, CLIP similarity, template match) first; only falls through to a VLM rubric if deterministic checks pass. Advisory mode only — never blocks CI. Auto-creates triage issues for failures. Use after any build that produces capture artifacts, or on-demand to audit the current game state."
argument-hint: "[--captures <dir>] [--scene <type>] [--advisory|--blocking] [--triage-issues]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion
---

# Art Direction Verification

This skill is the runtime counterpart to `/testable-art-direction`. It reads the
machine-readable art direction artifacts and a set of captured game screenshots,
runs a three-layer verification pipeline, and produces a report. It never edits
game code, never blocks CI by default, and never autonomously "fixes" anything.

## Critical honesty

The research (VideoGameQA-Bench, arXiv 2505.15952) is unambiguous: no current
vision-language model is production-ready for autonomous art QA. Best-in-class
models score 45–53% on visual regression and unit tests. The top false-positive
class is **hallucinated clipping** (40–44% of FPs). Therefore:

- **Default mode is ADVISORY.** VLM flags create triage issues, not CI blockers.
- **Deterministic checks are the load-bearing layer.** If Layer 1 passes
  cleanly over several weeks, you can promote individual Layer-2 items from
  advisory to blocking — but never wholesale.
- **A rubric item that never converges (high FP rate) should be retired**, not
  "fixed" by better prompting. Some "feels on-style" judgments are Band 4 in
  disguise and belong in the human-only bucket.

## Precondition Check

Before doing anything:

1. Read `design/art/palette.json`. If missing, fail with:
   > "No testable art direction found. Run `/testable-art-direction` first."
2. Read `design/art/rubric.yaml` and `design/art/thresholds.yaml`. Same failure if missing.
3. Glob `design/art/reference/*.png` — must contain at least one image.
4. Read `design/art/reference/MANIFEST.yaml` if present. Warn loudly if any references have `status: placeholder`.
5. Determine capture source:
   - If `--captures <dir>` passed, use that directory.
   - Else look for `screenshots/` at the project root.
   - Else fail with instructions to run the capture step first (or point at the Sight Lines workflow from KAMA-28).

## Layer 1 — Deterministic Checks

Spawn `technical-artist` via Task with the paths above and the list of capture files.
Ask:

> "Run the Layer 1 deterministic verification pipeline via
> `scripts/verify_art_direction.py --layer 1 --captures <dir> --style-guide design/art/`.
> Report the per-capture result for: (a) palette ΔE match, (b) WCAG contrast on any
> detected text regions, (c) CLIP cosine similarity to the nearest approved reference,
> (d) template match for required HUD elements per scene type.
> Return a JSON report with `pass`/`fail` per capture per check, plus the numeric
> measurement for each."

The `verify_art_direction.py` script lives in the project's `scripts/` directory.
If it does not exist yet, Technical Artist is responsible for creating it from
the reference implementation in `godot-e2e-test/scripts/verify_art_direction.py`
(the proof-of-concept version).

**Layer 1 hard rules:**
- Never call a VLM in Layer 1.
- Every check must return a number, not a judgment.
- If ColorThief / PIL / OpenCV / CLIP are missing, fail fast with install instructions, do not silently degrade.
- The script must be deterministic given fixed inputs. If it is not, that is a bug to file.

**Layer 1 outcomes:**
- **All checks pass** → proceed to Layer 2
- **Any blocker fails** → stop. Report. Do not invoke VLM. Advisory mode still creates a triage issue.
- **Only warnings fail** → proceed to Layer 2 but carry the warnings forward into the final report.

## Layer 2 — VLM Rubric (advisory)

Only if Layer 1 produced no blocker failures.

Spawn a fresh Claude sub-agent via Task with a carefully constrained prompt.
The prompt MUST include:

1. **Role pinning**: "You are an art-direction compliance checker, NOT an art critic. Your only job is to answer binary yes/no questions from a rubric against a provided screenshot, grounded in reference images from the style guide."
2. **The screenshot under test** (as an image attachment).
3. **Each reference image cited by applicable rubric items** (as image attachments, labelled by filename).
4. **The filtered rubric** — only items whose `applies_to_scenes` matches the current scene. Scene is determined by filename prefix convention (`01_main_menu_*.png` → menu) or by `--scene` flag override.
5. **The strict JSON output schema** — the sub-agent MUST return structured JSON with per-item verdict.
6. **The caution clause**: "You are known to hallucinate visual defects (clipping, floating objects, texture errors). Before calling a rubric item 'fail', verify the defect is clearly visible. If you cannot tell, answer 'fail' with rationale 'insufficient_evidence' — conservative by design."

**Consistency protocol:**
- Run **3 samples** at temperature 0.1.
- Majority vote per rubric item.
- For each item, record all 3 verdicts in the output so drift can be tracked.
- If two rubric items conflict within the same sample, record the conflict but do not attempt to resolve it.

**Required output schema for each VLM call:**

```json
{
  "capture": "screenshots/01_main_menu.png",
  "scene": "menu",
  "rubric_version": "1.0.0",
  "items": [
    {
      "id": "hero_silhouette_readable",
      "verdict": "pass",
      "rationale": "Player sprite at center is clearly distinguishable against the dark blue background — high luminance contrast and silhouette matches reference/approved_menu.png",
      "evidence_pixels_visible": true
    }
  ]
}
```

Non-conforming JSON output = test failure for that sample, not silent degradation.

## Layer 3 — Report and Triage

Merge Layer 1 and Layer 2 results per capture. Produce:

1. **Machine-readable JSON** at `reports/art_direction/<timestamp>.json` — for CI consumption
2. **HTML report** at `reports/art_direction/<timestamp>.html` — for human review, embeds the capture, the reference images it was compared against, the per-item verdicts, and the rationales
3. **Summary table** printed to stdout — PASS / WARN / FAIL count per check

For any failure (blocker or warning), check the `--triage-issues` flag:

- **With `--triage-issues`**: spawn `qa-tester` via Task with instructions to
  create a Paperclip issue per unique failing rubric-item × capture pair.
  Assignee: Art Director. Include: the capture file, the reference image cited,
  the rubric item ID, the verdict, and the rationale.
- **Without `--triage-issues`**: print a summary and exit. Suitable for local dev
  loops where the developer doesn't want to pollute the issue tracker.

## Failure semantics

| Layer 1 outcome | Layer 2 outcome | --advisory (default) | --blocking |
|---|---|---|---|
| All pass | All pass | exit 0 | exit 0 |
| All pass | Warnings only | exit 0, log warnings | exit 0, log warnings |
| All pass | Any blocker fail | exit 0, triage issues | **exit 1** |
| Blocker fail | skipped | exit 0, triage issues | **exit 1** |
| Warning fail only | All pass | exit 0, log warnings | exit 0 |

The `--blocking` flag is OFF by default and should only be turned on after a
specific rubric item has been stable (zero false positives) for at least N
consecutive runs. Track per-item stability in `reports/art_direction/stability.json`.

## What this skill does NOT do

- **Does not modify art assets.** Verification is read-only over the capture and style guide.
- **Does not update the art bible.** If the Art Director decides a failing rule should change, they update the bible via `/testable-art-direction --extend-existing`.
- **Does not create golden captures.** That's the QA Tester running a capture against an approved build and committing the result to `design/art/golden_captures/`.
- **Does not second-guess the rubric.** If an item is noisy, the response is to retire it or lower its severity, not to soften the VLM prompt.
- **Does not run without a testable art direction.** If `/testable-art-direction` has not been run, this skill fails fast — no implicit defaults.

## Known failure modes and mitigations

- **Hallucinated clipping** → The prompt explicitly warns the VLM about this. Track FP rate per item; any item hitting >20% FPs gets demoted from blocker to warning.
- **Capture vs. reference lighting variance** → CLIP similarity with a floor of 0.80 tolerates small variations. If similarity drops below threshold, Layer 1 fails BEFORE VLM is called, preventing false VLM judgments on legitimately-different frames.
- **Scene misclassification** → Filename prefix convention is the default; allow override via `--scene`. Do not infer scene from pixel content.
- **Over-strict palette ΔE** → `thresholds.yaml` owns this. Technical Artist can tune per-role, per-scene. Do not hide the threshold.
- **VLM temperature drift** → 3-sample voting cuts this significantly. Track per-run variance; if variance exceeds a threshold, surface it in the report.

## Related skills

- `/testable-art-direction` — must run first; owns the art direction spec
- `/smoke-check` — functional critical path smoke test, complementary not overlapping
- `/test-evidence-review` — reviews test files, not game visuals
- `/asset-audit` — naming and pipeline validation

## Integration with the existing visual validation pipeline

KAMA-26 and KAMA-28 established an SSH + xvfb + `--write-movie` capture pipeline.
This skill is a **post-step** in that pipeline:

1. KAMA-28 style capture → produces PNGs in `screenshots/`
2. `/art-direction-verify --captures screenshots/` → produces report + triage issues
3. QA Tester reviews the HTML report and confirms / dismisses each flag

Do not build a parallel capture infrastructure. Consume what KAMA-26 established.
