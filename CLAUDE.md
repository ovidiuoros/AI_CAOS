# Agent instructions (AI_CAOS)
 
AI driver monitoring assistant, built for the CodeLap Hackathon 2026 (AI Drive track, Porsche Engineering Romania).
It runs **live** on a laptop webcam. There is no pre-recorded video.
 
## How it works
 
```text
webcam -> vision/face_basic.py -> state.json -> dashboard/app.py -> browser (http://127.0.0.1:5000)
                                                     |
                                               dashboard/llm.py (Ollama)
```
 
1. `vision/face_basic.py` recognizes the driver, detects fatigue and distraction and writes `state.json` every 0.2 s.
2. `dashboard/app.py` reads `state.json`, streams it to the browser, simulates the vehicle and runs the copilot.
3. `dashboard/llm.py` asks the local LLM for a short message and car actions, then checks them before they are used.
4. Eyes closed for 3 s trigger the emergency actions in code, without waiting for the LLM.
## Hard rules
 
1. **Never read, quote or use anything in `context/`.** It holds confidential organizer material (NDA). It is gitignored. Leave it that way.
2. **Branches:** work on a branch off `dev` named `feat/<area>-<thing>` (areas: `vision`, `llm`, `dashboard`, `run`, `readme`, `presentation`, `config`). Use `feat/` for fixes and docs too: the PR workflow rejects any other prefix. Never commit to `main` or `dev` directly, and never force-push them.
3. **Ownership:** only edit the files of the area you were asked to work on (see the ownership table in `CONTRIBUTING.md`).
4. **Commits:** small, one logical change each, with a short plain-English message that starts with a verb (`Add per-driver calibration`, `Fix gaze label when the head is turned`). Keep the `Co-Authored-By` trailer: the hackathon rules require declaring AI tools.
5. **Keep it simple:** one small working step at a time. Don't add new features, files or dependencies unless asked.
6. **Python is managed by uv only:** `uv add <pkg>` for dependencies (commit `pyproject.toml` and `uv.lock` together), `uv run ...` to run. Never `pip install`.
7. **Never commit runtime or private files:** `.env`, `profiles.json` (drivers' face prints, biometric data), `calib*.json`, `state.json`, downloaded models (`*.task`, `*.onnx`, `*.pt`, `*.tflite`).
## Run
 
From the repository root (all parts must share the same `state.json`):
 
```bash
uv sync                                   # install Python 3.11 and the dependencies
uv run python run.py                      # vision first, then the dashboard at http://127.0.0.1:5000
```
 
Parts separately:
 
```bash
uv run python vision/face_basic.py        # vision only, writes state.json
uv run python dashboard/app.py            # dashboard and copilot, reads state.json
uv run python dashboard/llm.py            # tests the copilot on fixed situations (needs Ollama)
```
 
The dashboard and copilot need a `.env` in the repository root with `LLM_MODEL` and `LLM_TIMEOUT_SEC`, and Ollama running with that model.
 
## Project layout
 
| Path | Purpose |
|---|---|
| `run.py` | Starts the vision program, then the dashboard. |
| `vision/` | Driver monitoring from the webcam. Every module is described in `vision/README.md`. |
| `vision/state.py` | The `state.json` schema: every key, its default value and its meaning. |
| `vision/alerts.py` | Alert codes, levels and priority order. |
| `dashboard/app.py` | Flask server, live stream to the browser, simulated vehicle. |
| `dashboard/llm.py` | Copilot: when to react, the prompt, the LLM call, safety checks and fallback messages. |
| `dashboard/scenarios.py` | Simulated road events. |
| `dashboard/templates/`, `dashboard/static/` | The dashboard page, voice alerts and alarm sounds. |
| `scenario_demo.py` | Turns a typed sentence into a road event name. |
| `docs/research.md` | Research on existing driver monitoring systems. |
 
## Conventions
 
- Comments, names, logs and UI text in English.
- One module = one job. Every adjustable value is a constant at the top of its module, marked with a `TUNE:` comment.
- A new `state.json` key is added to `vision/state.py` with a comment that explains it.
- When an alert is added or renamed in `vision/alerts.py`, update the `ALERTS` and `LEVEL_OF` tables in `dashboard/llm.py` the same way, otherwise the copilot ignores it.
- The LLM only proposes. Every message and action goes through the checks in `dashboard/llm.py`, and safety-critical behaviour stays in code.
- Update `README.md` or `vision/README.md` when a behaviour, command or setting changes.
## Testing
 
There are no automated tests. Before finishing a change:
 
1. Run `uv run python run.py` and check the change with the real webcam.
2. For copilot changes, run `uv run python dashboard/llm.py` (needs Ollama).
3. Check `git status`: no runtime or private files staged.
 
Known open issues are listed in the "Known issues" section of `README.md`.
 
