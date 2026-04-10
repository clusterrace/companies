---
name: Implement Art Direction Verification Pipeline
assignee: technical-artist
project: new-game-kickoff
---

Implement the art direction verification pipeline described in the `/art-direction-verify` skill, consuming the `design/art/` artifacts produced by the Art Director.

## Why two layers

VLMs are advisory-grade for art QA (see VideoGameQA-Bench, 2025 — best regression model scored 45%). The pipeline must put deterministic code first and VLM second:

- **Layer 1 — Deterministic**: palette ΔE (CIEDE2000), WCAG UI contrast, reference similarity, hero element size. Never calls a VLM. Fast, reproducible, cheap.
- **Layer 2 — VLM rubric (advisory)**: only runs if Layer 1 passes. 3-sample majority vote per rubric item. Binary verdicts only. JSON output. Never blocks CI in this task — advisory mode only.
- **Layer 3 — Report**: JSON + HTML output for CI + human review.

## Deliverables

Commit to the project's code repository:

- `scripts/verify_art_direction.py` — the main pipeline
- `scripts/requirements.txt` — minimal deps (Pillow, numpy, PyYAML)
- `scripts/README.md` — how to run, what each check does, where reports go
- A first successful run against any existing capture set, with the report committed under `reports/art_direction/`

## Backend choice for Layer 2

Call `claude --print` as a subprocess, not the Anthropic SDK directly. This uses OAuth via the user's Claude Code session (same pattern as Paperclip's `claude_local` adapter) and avoids needing an API key in the environment. Alternatives (Ollama with a vision model, llama.cpp) can be added as pluggable backends with the same function contract.

Key CLI flags for the subprocess call:
- `--print` (non-interactive)
- `--model claude-haiku-4-5` (cheap, fast, good enough for binary rubric judgments)
- `--output-format json` (wrapped result)
- `--system-prompt <pinned prompt>` (role: "compliance checker, not art critic")
- `--allowedTools Read` (so Claude can read the image files)
- `--add-dir <capture_dir> --add-dir <style_guide_dir>` (grant file access)
- `--exclude-dynamic-system-prompt-sections --disable-slash-commands --no-session-persistence` (minimize context bloat and cost)

## Honest caveats

- **Advisory only** by default. Do not turn on `--blocking` mode in CI in this task.
- **Expect 10–30% false positive rate** on Layer 2 rubric items at first. Track it per item in `reports/art_direction/stability.json`.
- **Skip VLM on Layer 1 blockers** — save cost and avoid noise on captures that are already doomed.
- **Hallucinated visual defects** (especially clipping) are the #1 VLM failure mode. Pin this in the system prompt.

## Reference implementation

A working reference implementation lives in the `godot-e2e-test` project at `scripts/verify_art_direction.py`. Adapt it to this game's rubric and scene types — do not copy blindly.

Key adaptations expected:
- Scene type detection from capture filenames (your game's conventions)
- Layer 1 check thresholds tuned to your game's visual density and palette
- Rubric filtering by `applies_to_scenes`
- Optional: add CLIP cosine similarity if `open-clip-torch` is acceptable (swap out the histogram-intersection placeholder)

## Depends on

- `create-testable-art-direction` — without `design/art/` artifacts, nothing to verify against.

## Blocks

- `integrate-art-direction-verification`
