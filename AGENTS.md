# Agent Instructions for sd-webui-supermerger

sd-webui-supermerger is a local Stable Diffusion WebUI extension for merging checkpoints, LoRAs, and model weights from inside A1111-compatible WebUI environments. This checkout is used with Forge Neo under `C:\AI\sd-webui-forge-neo`.

Use this file as the primary repository context for coding agents. Keep changes narrow, testable, evidence-preserving, and compatible with the local WebUI install.

## Core Product Model

- Checkpoint files, LoRAs, VAEs, metadata, and generated outputs are user-owned local assets.
- Merge recipes, XY plots, random-ratio runs, and metadata are local evidence for comparing model behavior.
- The extension runs inside WebUI; it should not become a standalone service.
- The extension may load, merge, save, and generate through WebUI processing, but it must not add remote generation or upload behavior.
- The default posture is local-first, Windows-friendly, and conservative around model folders and GPU memory state.

## High-Value Docs And Files To Read

Before broad changes, read:

- `README.md` for user-facing behavior, setup, warnings, and workflows.
- `changelog.md` for upstream behavior history.
- `scripts/supermerger.py` for UI registration, tab layout, model metadata, and utility actions.
- `scripts/mergers/mergers.py` for checkpoint merge math, model loading, Forge/Neo handling, cache handling, generation, and save behavior.
- `scripts/mergers/pluslora.py` for LoRA merge/extract paths and bundled A1111 network behavior.
- `scripts/mergers/xyplot.py` for XY/XYZ plotting, random mode, seed handling, and grid generation.
- `scripts/mergers/model_util.py` for save/prune/fp16/metadata behavior.
- `scripts/A1111/` only when debugging bundled LoRA/network compatibility.
- Haoming02/sd-webui-forge-classic `neo` source when debugging Forge Neo compatibility.

Do not assume a compatibility issue is fixed unless the local code and current behavior show it.

## Current Stack

- Python inside the host WebUI virtual environment.
- Gradio UI components embedded through `modules.script_callbacks`.
- PyTorch, safetensors, and WebUI model-loading helpers.
- A1111/Forge modules such as `shared`, `sd_models`, `sd_vae`, `processing`, `devices`, `images`, and `extra_networks`.
- Windows-first local development under `C:\AI\sd-webui-forge-neo\extensions\sd-webui-supermerger`.
- Primary host target: the local Forge Neo install at `C:\AI\sd-webui-forge-neo`.

## Compatibility Rules

- Preserve A1111 compatibility where practical while adding Forge Neo compatibility.
- Prioritize the local Forge Neo branch for testing and compatibility fixes.
- Prefer capability checks such as `getattr(...)`, `hasattr(...)`, and feature detection when WebUI or Gradio APIs differ.
- Treat `shared.cmd_opts`, checkpoint path handling, `sd_models`, model cache state, LoRA/network APIs, and Gradio component APIs as version-sensitive.
- Do not assume `sd_models.checkpoints_loaded` exists; Forge Neo uses a different model-data surface.
- Do not assume old A1111 model loading, `read_state_dict`, or LoRA network mapping paths work under Neo.
- Be conservative around direct `shared.sd_model`, `sd_models.model_data`, backend memory management, and Forge reload hooks.
- Avoid broad rewrites of merge math, LoRA merge logic, or generation flow until current behavior is understood from logs and local tests.

## Development Rules

- Prefer small, focused diffs.
- Preserve user-created presets, logs, metadata, generated grids, merge outputs, model files, LoRAs, VAEs, caches, and local config.
- Do not delete checkpoint files, model folders, LoRAs, generated images, previews, caches, or local logs unless explicitly requested.
- Do not silently change merge math, alpha/beta semantics, MBW behavior, random mode, seed behavior, save naming, overwrite behavior, metadata behavior, or model-cache behavior.
- Treat model paths, checkpoint names, prompts, merge expressions, presets, sidecar metadata, and generation parameters as user-controlled input.
- Be careful with cross-drive paths and symlinks on Windows.
- Keep publication to the user's fork unless the user explicitly asks for upstream PR work.
- Leave unrelated working-tree dirt alone.

## GitHub Workflow And User Learning

The user wants GitHub issues, commits, and PRs to be used as an explicit learning surface. For non-trivial work, write clear issue and PR text that explains what happened, why the change exists, how it was validated, and what remains uncertain.

Treat GitHub as durable project-management state for this repository, not just as a publication target. Issues, labels, milestones, PR comments, and draft PRs are working state for the user and future agents.

Default publication target is the user's fork, `odyss3y/sd-webui-supermerger`. Keep the original upstream project as `upstream` when configuring remotes. Do not open upstream PRs unless explicitly asked.

When the user reports a runtime error, traceback, broken UI behavior, or Forge/A1111 compatibility issue:

- Treat it as a user-reported issue unless the user says not to.
- Create or draft a GitHub issue on the user's fork before or during the fix when practical.
- Preserve the exact traceback or log excerpt, trimmed only for secrets or excessive noise.
- Describe observed behavior, expected behavior, environment, suspected cause, fix scope, validation, and remaining uncertainty.
- Link the issue in the commit message or PR body when committing or opening a PR.
- After the fix, update the issue or PR with what changed and what still needs live Forge Neo verification.

Prefer draft PRs for larger or multi-step work. Keep PR descriptions practical and evidence-oriented. Include:

- Summary
- Why
- What Changed
- Validation
- Data / File Impact
- Compatibility Notes
- Unchanged / Non-goals
- Risks / Follow-ups

## UI Registration

The extension should register the SuperMerger UI through WebUI callbacks in `scripts/supermerger.py`.

If the tab does not appear, first inspect WebUI startup logs for:

- `*** Error loading script: supermerger.py`
- `ui_tabs_callback` errors
- Gradio API errors
- missing module imports
- `sd_models`, `backend`, `modules_forge`, or LoRA/network import errors

Fix import-time errors before debugging merge, LoRA, or generation behavior.

## Merge And File Handling

- Merges should not delete source checkpoints, LoRAs, VAEs, or generated evidence.
- Respect overwrite controls unless the task is explicitly about overwrite behavior.
- Preserve safetensors metadata behavior unless the task is explicitly about metadata.
- Preserve fp16/prune/copy-config behavior unless the task is explicitly about those save options.
- Treat live model loading, unloading, and cache clearing as stateful operations that can affect WebUI runtime and GPU memory.
- Avoid package installs, dependency upgrades, or environment rebuilds unless explicitly requested.

## Testing And Validation

Use the smallest validation that proves the changed surface:

```powershell
python -c "import ast, pathlib; files=list(pathlib.Path('scripts').rglob('*.py'))+[pathlib.Path('install.py')]; [ast.parse(f.read_text(encoding='utf-8'), filename=str(f)) for f in files]; print('syntax ok')"
git diff --check
```

When practical, restart Forge Neo and verify:

- no `supermerger.py` startup error
- the SuperMerger tabs appear
- checkpoint dropdowns load
- a minimal merge path can read checkpoints and load/save as expected
- LoRA merge/extract paths do not fall back to incompatible A1111-only code under Neo
- generation/XY plot paths do not reload the wrong model

If live WebUI validation is blocked, say exactly what was checked and what remains unverified.

## Decision Notes

For non-trivial changes, preserve the reason and constraints in a commit message, PR body, issue comment, or compact project note. Do not preserve chat transcripts, private reasoning, or chain-of-thought.

Use this format when a durable decision matters:

```text
Decision: <short title>
Prompt / trigger: <one sentence>
Intent inferred: <one sentence>
Decision: <what changed>
Why: <one or two bullets>
Guardrail: <what future agents must not break>
Follow-up: <next task, if any>
```

Keep it short. Preserve intent, scope, validation, compatibility boundaries, and deferred work.

## Definition Of Done

A non-trivial code change should include:

- focused implementation
- syntax/import/UI validation appropriate to the changed surface
- docs or changelog updates when user-facing behavior changes
- no deletion of user models, LoRAs, VAEs, generated assets, or local logs
- no silent semantic change to merge math, save naming, overwrite behavior, metadata behavior, or cache behavior
- note of any tests not run and why
- compact decision note when future agents need the rationale

## Known Priority Areas

- Forge Neo script-load compatibility.
- Gradio 4 component compatibility.
- Neo model loading/reload/cache behavior.
- LoRA/network mapping under Forge Neo model objects.
- Checkpoint and VAE path handling across WebUI variants.
- GPU memory cleanup and cache behavior during repeated merges.
- Metadata preservation for saved merged models.
