# Contributing to AI_CAOS
 
This file explains how the team works on the project: how to set it up, how to name branches and commits,
how a change gets from your laptop into `main`, and who owns which part of the code.
 
**Contents:** [Setup](#setup) · [Workflow](#workflow-from-task-to-main) · [Branches](#branches) ·
[Commits](#commits) · [Pull requests](#pull-requests) · [Never commit](#never-commit) ·
[Code style](#code-style) · [Ownership](#ownership) · [Issues](#issues) · [AI tools](#ai-tools)
 
---
 
## Setup
 
```bash
git clone https://github.com/andrei-r-76/AI_CAOS.git
cd AI_CAOS
uv sync                        # installs Python 3.11 and every dependency into .venv
```
 
Create a `.env` file in the repository root (it is gitignored):
 
```text
LLM_MODEL=llama3.2:3b
LLM_TIMEOUT_SEC=10
```
 
Start everything with `uv run python run.py`. The full setup (Ollama, webcam, optional YOLO) is in the
[README](README.md#requirements).
 
---
 
## Workflow: from task to `main`
 
1. **Pick or open an issue** for the task (see [Issues](#issues)).
2. **Create a branch** from the latest `dev`:
```bash
git switch dev
git pull --ff-only origin dev
git switch -c feat/<area>-<thing>
```
 
3. **Work in small commits** (see [Commits](#commits)) and push the branch:
```bash
git add <files>
git commit -m "Add per-driver calibration"
git push -u origin feat/<area>-<thing>
```
 
4. **Stay in sync** with `dev` while you work:
```bash
git fetch origin
git merge origin/dev
```
 
5. **Open a pull request into `dev`**, fill in the checklist (see [Pull requests](#pull-requests)) and ask
   for a review.
6. **Merge** after 1 approval. The branch is deleted automatically by `delete_feature.yml`.
7. **Milestone:** when `dev` works end to end, open a PR from `dev` into `main`, merge it with a merge commit
   and tag the result on `main` (for example `v0.1-hackathon`).
---
 
## Branches
 
```text
main  ← stable: only merges from dev at milestones, every milestone is tagged
 └─ dev  ← integration: every feature PR targets dev
     ├─ feat/vision-rework
     ├─ feat/llm-alert-states
     ├─ feat/dashboard-fix
     └─ feat/readme
```
 
**Rules**
 
- Every branch name starts with `feat/`, **also for fixes and docs**. The `pr_routing.yml` workflow only
  accepts pull requests from `feat/*` into `dev`, and from `dev` into `main`. Anything else fails the check.
- Name format: `feat/<area>-<thing>`, lowercase, words separated by `-`.
  Areas: `vision`, `llm`, `dashboard`, `run`, `readme`, `presentation`, `config`.
  | Good | Bad |
  |---|---|
  | `feat/vision-threshold` | `fix/threshold` (rejected by the workflow) |
  | `feat/dashboard-alerts` | `feat/Dashboard_Alerts` |
  | `feat/readme-run` | `my-changes` |
- Keep branches short-lived: a few hours, at most a day. Long branches are hard to merge.
- Never push directly to `main` or `dev`, and never force-push them. Both are protected from deletion.
---
 
## Commits
 
Small and frequent, with a short message in plain English that says **what changed**:
 
```text
Modify threshold for unknown driver
Add model version in terminal at warmup
Detect glasses from the frame bridge on the nose
```
 
- Start with a verb: `Add`, `Fix`, `Update`, `Remove`, `Move`, `Rename`.
- One line, ideally under 72 characters. Put longer explanations in the pull request, not in the commit.
- The area is already in the branch name (`feat/vision-...`), so the message doesn't need to repeat it.
- Avoid messages that say nothing, like `Save my changes`, `fix` or `update`.
- One commit = one logical change. Don't mix a bug fix and a new feature in the same commit.
- Don't squash feature PRs: we want the individual commits in the history.
- When you add a dependency, commit `pyproject.toml` and `uv.lock` together.
---
 
## Pull requests
 
**Title:** a short sentence like a commit message, for example `Add per-driver calibration`.
 
**Description:** what changed, why, how you tested it, and `Closes #<issue>`.
 
**Checklist before asking for a review**
 
- [ ] `uv run python run.py` starts and the change works with the real webcam
- [ ] no runtime or private files are committed (see [Never commit](#never-commit))
- [ ] new dependencies were added with `uv add`, and `uv.lock` is committed
- [ ] if you added or renamed an alert in `vision/alerts.py`, the `ALERTS` and `LEVEL_OF` tables in
      `dashboard/llm.py` have the same change (otherwise the copilot ignores the alert)
- [ ] if you added a key to `state.json`, it is described in `vision/state.py`
- [ ] the README or `vision/README.md` is updated if the behaviour, a command or a setting changed
**What the reviewer checks:** the code does what the PR says, it runs, nothing private is committed and the
names and comments are clear.
 
---
 
## Never commit
 
These files are created while the app runs. Most are already in `.gitignore`; check `git status` before you
commit anyway.
 
| File | Why |
|---|---|
| `.env` | local settings |
| `profiles.json` | **the drivers' face prints (biometric data)** |
| `calib_*.json`, `calib.json` | personal calibrations |
| `state.json` | live driver state, rewritten every 0.2 s |
| `*.task`, `*.onnx`, `*.pt`, `*.tflite` | downloaded models, they download again by themselves |
| `context/` | organizer materials under NDA |
 
---
 
## Code style
 
- Python 3.11, environment managed only by **uv**: `uv add <package>`, `uv run ...`. Never `pip install`.
- Comments, names, logs and UI text in **English**.
- One module = one job (see the module list in [vision/README.md](vision/README.md#files)).
  Keep files small; add a new module rather than growing an existing one into everything.
- Every adjustable value (threshold, duration, speed setting) is a constant at the top of the module,
  marked with a `TUNE:` comment that says what it does.
- Run everything from the repository root, so all parts use the same `state.json`.
---
 
## Ownership
 
| Area | Files | Owner |
|---|---|---|
| Vision: driver monitoring, recognition, calibration | `vision/` | [Ovidiu](https://github.com/ovidiuoros) |
| LLM copilot and safety checks | `dashboard/llm.py`, `dashboard/scenarios.py` | [Cristi](https://github.com/cristibutica) |
| Dashboard, voice alerts and simulated vehicle | `dashboard/app.py`, `dashboard/templates/`, `dashboard/static/` | [Szilárd](https://github.com/DaSzil) |
| Integration, launcher, GitHub workflows | `run.py`, `scenario_demo.py`, `.github/` | [Andrei](https://github.com/andrei-r-76) |
| Presentation | `presentation/` | [Ovidiu](https://github.com/ovidiuoros) |
| Docs and research | `README.md`, `docs/` | the whole team |
 
If you need a change in someone else's files, open an issue or ask the owner. Don't edit them yourself.
 
---
 
## Issues
 
One GitHub Issue per task or bug. Reference it in the PR with `Closes #12`, so it closes on merge.
 
When you report a bug, include:
 
- what you did and what you expected
- what happened instead (the terminal output or a screenshot)
- your OS and, for vision bugs, the light conditions and whether you wear glasses
---
 
## AI tools
 
- Agents follow `CLAUDE.md` (Claude Code) and `AGENTS.md` (Codex). Both have the same rules; when you change
  one, change the other too.
- Review AI-written code like any other code before you commit it.
- Never paste anything from `context/` (organizer materials, under NDA) into an AI tool.
 
