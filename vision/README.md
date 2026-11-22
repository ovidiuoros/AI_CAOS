# Vision: driver monitoring from the webcam
 
> Watches the driver through the laptop webcam, recognizes who is driving, detects fatigue and distraction, and writes everything to `state.json` several times a second for the LLM and the dashboard.
 
```bash
uv run python face_basic.py
```
 
The MediaPipe models download by themselves on the first run. The face-recognition model (**SFace**) also downloads once. Phone detection is optional and needs `ultralytics` (`uv add ultralytics`).
 
---
 
## How it works
 
Every camera frame goes through the same steps:
 
```text
camera ─► MediaPipe (face + hands) ─► pick the driver ─► who is it? (faceid)
                                              │
            eyes · blinks · gaze · head · mouth · glasses · hands · phone
                                              │
                                       alerts ─► state.json ─► LLM / dashboard
                                              │
                                    drawing ─► app window
```
 
Only the driver is analysed: the largest face, kept even when other faces enter the frame. A face that does not match any saved driver is shown as **"unknown"** and is not monitored.
 
**Start:** no calibration at start. If drivers are saved, the last one is selected with his saved calibration, and recognition switches to whoever sits in front of the camera (~1.5 s). With no driver saved, the app waits: type a name and press **Add and calibrate** (calibration, then the face is learned).
 
---
 
## Files
 
| File | What it does |
|---|---|
| `face_basic.py` | Main program. The frame loop: runs every module in order, fills the state, writes `state.json`, draws on the image and shows the window. |
| `camera.py` | Camera capture on its own thread; prepares frames for MediaPipe (light measurement, automatic contrast boost in low light, timestamps); camera covered / too dark. |
| `tracking.py` | Picks the driver among the faces in the frame and keeps the same one; small helpers for MediaPipe face results. |
| `faceid.py` | Recognizes the driver from his face (OpenCV SFace) and learns the face of a new driver. |
| `calibration.py` | Per-driver calibration: normal head pose, eye-closed and yawn thresholds, gaze zones. Saved in `calib_<name>.json`. |
| `gaze.py` | Where the driver looks (road / left / right / up / down), eyes-on-road ratio, closed eyes, blinks, microsleep. |
| `head.py` | Head yaw / pitch / roll, head drop (nodding off), nods per minute, head tilt. |
| `mouth.py` | Yawning, mouth open, talking. |
| `eyewear.py` | Glasses and sunglasses on the driver; glasses missing (the profile says he wears them). |
| `hand_face.py` | Hands relative to the face: near the face, at the ear (phone call), at the eyes (rubbing), at the mouth. |
| `objects.py` | Phone detection with YOLO on a background thread (optional). |
| `alerts.py` | Turns the detections into ONE alert level + reason, counters and an event log. |
| `state.py` | The `state.json` schema: every key with its default value and meaning. |
| `drivers.py` | Driver profiles and the app window (camera on the left, driver panel on the right). |
| `utils.py` | Shared helpers: `Held` ("true for N seconds"), model download, beep. |
 
> [!TIP]
> Every line marked `TUNE:` is a value that can be adjusted (thresholds, durations, speed settings).
 
---
 
## The interface
 
One window, two parts:
 
### Left: camera
 
- **Top bar:** DRIVER CAMERA, and on the right the current gaze and the eyes-on-road percentage.
- **Above the driver's face:** head angles, eye openness (with the glasses / sunglasses checks), time since the last blink, gaze.
- **Under the mouth:** mouth values and state (closed / open / talking / yawning).
- **Bottom:** FPS.
- **Under the image, Warnings:** the 3 most important active alerts as colored tags (`CRITICAL`, `WARNING`, `INFO`); all of them are in `state.json`.
### Right: Driver Monitor panel
 
- **Active driver:** name, status (`RECOGNIZED`, `PICKED BY HAND`, `NEW DRIVER`, `UNKNOWN FACE`, `NO PROFILE`), face-match bar (the black tick is the recognition threshold; updated twice a second).
- **Glasses:** detected now or not, and "Wears glasses" for the profile (warns when they are missing).
- **Drivers:** the saved drivers; **Use selected** picks one by hand (recognition still checks him), **Delete** (deleting the driver being calibrated cancels the calibration), **Recalibrate**, **Re-learn face**.
- **New driver:** type a name, **Add and calibrate** (calibration, then the face is learned).
### Keys
 
| Key | Action |
|:---:|---|
| <kbd>q</kbd> | quit |
| <kbd>f</kbd> | show / hide the face drawing (points, box, eye and mouth lines; the text stays) |
| <kbd>c</kbd> | cancel a calibration (the old values are kept) |
 
---
 
## `state.json`
 
Flat JSON, written atomically (a reader never sees a half-written file). All keys and their meaning are in `state.py`. Main groups:
 
| Group | Keys |
|---|---|
| **alerts** | `alert_level`, `alert_reason`, `active_alerts`, `events` - react to changes of `alert_level` / `alert_reason`. |
| **driver** | `driver` (name, "unknown" until recognized), `face_learned`, `calibrated`, `face_found`, `face_lost_sec`. |
| **eyes** | `eyes_closed_sec`, `eyes_open_sec`, `blink_rate`, `slow_blinks_1min`, `microsleep`, `sleep`, `microsleep_count_10min`. |
| **glasses** | `glasses`, `sunglasses`, `glasses_missing`. |
| **head** | `head_yaw` / `head_pitch` / `head_roll`, `head_drop`, `head_down_sec`, `head_nods_1min`, `head_tilt`. |
| **mouth** | `yawning`, `yawn_count_5min`. |
| **gaze** | `gaze` (`on_road` \| `left` \| `right` \| `up` \| `down`), `looking_away_sec`, `long_glance`, `on_road_ratio`, `distraction` (talking while looking away from the road). |
| **hands / phone** | `hand_near_face`, `hand_at_ear`, `hand_at_eyes`, `eye_rub_count_5min`, `phone_detected`, `phone_use`. |
 
---
 
## Alerts
 
In priority order (the first active one is `alert_reason`):
 
| Alert | Level | When |
|---|:---:|---|
| `sleep` | 🔴 critical | eyes closed over 3 s |
| `repeated_microsleep` | 🔴 critical | a 2nd (or later) microsleep within 10 min, while it happens |
| `microsleep` | 🔴 critical | eyes closed 1-3 s |
| `head_drop` | 🔴 critical | head falls forward fast |
| `drowsy` | 🔴 critical | 2 or more fatigue signs at once (frequent yawning, frequent eye rubbing, slow blinking, low blink rate, head nodding) |
| `driver_missing` | 🔴 critical | driver's face missing over 3 s (not shown when the camera is covered) |
| `phone_use` | 🟠 warning | phone in the hand / at the face, or a hand held at the ear |
| `long_glance` | 🟠 warning | looking away from the road for 3 s |
| `distraction` | 🟠 warning | talking while looking away from the road for 1 s (e.g. at a passenger) |
| `glasses_missing` | 🟠 warning | the profile says glasses, none seen |
| `head_nodding` | 🟠 warning | 2 head nods in a minute |
| `frequent_yawning` | 🟠 warning | 3 yawns in 5 min |
| `frequent_eye_rubbing` | 🟠 warning | 3 eye rubs in 5 min |
| `slow_blinking` | 🟠 warning | 3 slow / heavy blinks in a minute (a blink is timed from closing to opening; slow = over 0.4 s) |
| `low_blink_rate` | 🟠 warning | under 5 blinks in the last minute (staring) |
| `camera_blocked` | 🟠 warning | camera covered |
| `yawning`, `rubbing_eyes`, `hand_at_mouth`, `phone_visible`, `head_tilt`, `hand_near_face`, `low_light` | 🔵 info | happening now |
 
> [!NOTE]
> An alert stays active 2 s after its condition ends, so it does not flicker. Closed eyes are not checked when the eyes cannot be seen (head turned too far, sunglasses, camera covered, hand on the eyes), so these cannot cause a false microsleep.
 
---
 
## Files created while running
 
| File | Content |
|---|---|
| `state.json` | the current driver state |
| `profiles.json` | saved drivers: glasses yes / no and face prints |
| `calib_<name>.json` | each driver's calibration (`calib.json` when no driver is selected) |
| `*.task`, `*.onnx`, `*.pt` | downloaded models |
