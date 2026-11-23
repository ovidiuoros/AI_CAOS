# Research: driver monitoring – regulations, production systems and where AI_CAOS stands
 
Team research notes for the AI Drive track (CodeLap Hackathon 2026, Porsche Engineering Romania).
The first part covers the rules and the systems that exist today. The second part compares AI_CAOS with the
**Euro NCAP 2026** requirements and explains where the project goes further and what is still missing.
 
**Contents:** [Why it matters](#why-it-matters) · [Euro NCAP 2026](#euro-ncap-2026-driver-engagement) ·
[Production systems](#production-systems) · [Driver recognition in production cars](#driver-recognition-and-profiles-in-production-cars) ·
[CV metrics](#computer-vision-metrics) · [AI_CAOS vs Euro NCAP](#ai_caos-vs-euro-ncap-2026) ·
[Recognition, profiles and glasses](#our-recognition-profiles-and-glasses-phase) · [Differentiation](#where-we-differentiate) ·
[Gaps](#gaps-and-next-steps) · [Tech stack](#tech-stack-planned-vs-final) · [Sources](#sources)
 
---
 
## Why it matters
 
- **EU General Safety Regulation (EU) 2019/2144** requires:
  - **DDAW** (Driver Drowsiness and Attention Warning): mandatory for new vehicles since 2022/2024.
  - **ADDW** (Advanced Driver Distraction Warning, camera-based): mandatory for all new registrations from July 2026.
- **Euro NCAP 2026** turns driver monitoring from a small bonus into a core part of the star rating (details below).
- Human error (distraction, fatigue, phone use) causes most crashes.
---
 
## Euro NCAP 2026: Driver Engagement
 
From 2026 Euro NCAP rates cars in four stages: **Safe Driving, Crash Avoidance, Crash Protection and Post-Crash
Safety**, each scored out of 100. Driver monitoring sits in **Safe Driving → Driver Engagement**. In 2023 a basic
drowsiness / inattention warning was worth up to 2 points; in 2026 driver monitoring is worth up to **25 points**,
and a 5-star car needs continuous **eye and head tracking**. The protocol also adds points for detecting
**alcohol / drug impairment** and for bringing an **unresponsive driver** to a controlled stop.
 
### Scoring (Driver Engagement protocol v1.2, July 2026)
 
| Part | Points |
|---|---:|
| **Transient states** | **15** |
| Long distraction | 5 |
| Short distraction (VATS: visual attention time sharing) | 5 |
| Phone use | 5 |
| **Non-transient states** | **10** |
| Impairment (2 for alcohol / drugs, 2 for drowsiness) | 4 |
| Microsleep | 2 |
| Sleep | 2 |
| Unresponsive driver | 2 |
 
### What each state means
 
| State | Euro NCAP definition |
|---|---|
| Long distraction | one continuous glance away from the road of 3–4 s (after 4 s looking at the road); head turns ("owl"), eyes-only moves ("lizard") and leaning are all tested |
| Short distraction (VATS) | 10 s of glances away from the road in total within any 30 s |
| Phone use | glances at a phone: basic (lap, knees, dashboard mount) and advanced (phone held near the wheel, windscreen or cluster) |
| Drowsiness | must be flagged at the latest at **KSS > 7** (Karolinska Sleepiness Scale), from 50 km/h |
| Microsleep | eyes closed 1–2 s |
| Sleep | eyes closed 3 s or more |
| Unresponsive driver | gaze does not return to the road within 3 s of a distraction warning, or eyes closed 6 s or more |
| Impairment (alcohol / drugs) | from 50 km/h, with up to 10 minutes of learning at the start of each trip |
 
### General requirements
 
- On by default at the start of **every trip**. It cannot be switched off with one short button press, and the
  driver cannot change its sensitivity.
- Measuring starts after 1 minute of driving at 10 km/h or more. Distraction, microsleep, sleep and unresponsive
  driver must be handled from 20 km/h.
- **Warnings:** visual plus sound and/or vibration, immediately. Microsleep and sleep warnings must feel **more
  urgent** than distraction warnings. For drowsiness, a rest-area suggestion in the navigation is accepted.
- **Reaction:** the car makes its safety systems more sensitive (earlier collision warning / braking, lane
  keeping) for a distracted or impaired driver.
- **Unresponsive driver:** the car must start its assisted mode within 5 s of the final warning and slow down to
  a stop (or under 10 km/h) while keeping the lane.
- A transient alert ends **at least 2 s after** the state ends.
- **Camera blocked:** if the system stops working, the driver must be told within 10 s.
### Which drivers it must work for
 
| Condition | Requirement |
|---|---|
| Age, sex, height, skin tone | 16–80 years, all sexes, small to tall drivers, all skin tones (Fitzpatrick 1–6) |
| Light | from bright daylight (100,000 lux) down to night (1 lux) |
| **Glasses** | clear glasses, including thick frames, **must work** |
| **Sunglasses** | very dark lenses (under 15% light): the car must at least **tell the driver** it cannot see the eyes |
| Beard, face mask | must work |
| Hat, long fringe over the eyes, heavy eye make-up | must at least tell the driver if it does not work |
| Talking | must work; eating, singing, smoking, eye rubbing and sneezing are not assessed |
| **Driver identification / profiles** | **not covered** by the driver monitoring assessment |
 
---
 
## Production systems
 
| System | Sensing | Detects | Reaction |
|---|---|---|---|
| Mercedes Attention Assist, Bosch drowsiness detection | steering-input patterns, time driven | fatigue | coffee-cup icon, chime, break suggestion |
| Subaru DriverFocus, Mazda DMS, BMW Driving Assistant Pro, GM Super Cruise, Ford BlueCruise | IR driver-facing camera | eyes closed, gaze off road | escalating visual → audio → seat/wheel haptics → hands-on request → controlled stop |
| Volvo / Mercedes emergency stop assist | camera + no steering input | unresponsive driver | hazard lights, slows to a stop in lane, emergency call |
| Tier-1 suppliers: Seeing Machines, Smart Eye, Cipia, Tobii | NIR camera + neural nets | gaze zones, PERCLOS, phone, smoking, seatbelt, occupancy | defined by the carmaker |
| Lane Departure Warning / Lane Keep Assist | front camera | lane drift (indirect sign of fatigue) | steering-wheel vibration |
 
### The common alert pattern: an escalation ladder
 
1. **L1:** icon on the cluster
2. **L2:** chime + text message
3. **L3:** haptic (seat or steering-wheel vibration) + voice
4. **L4:** intervention: slow down, hazard lights, SOS call
Cooldowns between alerts prevent alarm fatigue (drivers ignore or disable systems that nag).
 
---
 
## Driver recognition and profiles in production cars
 
Some carmakers already recognize the driver's face with the driver monitoring camera, but **only for comfort**:
 
| System | What recognition is used for |
|---|---|
| Subaru DriverFocus (since the 2019 Forester) | up to 5 drivers; restores seat, mirrors, climate and the last infotainment screen |
| Mazda Driver Personalization System | stores the face as numbers in a driver profile; restores seat and mirror settings; may fail with sunglasses or a mask |
| Hyundai Mobis driver identification | several registered drivers, planned link to automatic seat and mirror adjustment |
| Genesis GV60 Face Connect | NIR camera face recognition to unlock the car |
 
**Takeaway:** in production cars the profile changes the **seat and the mirrors**, not the **safety thresholds**.
Euro NCAP does not assess identification at all. This is where AI_CAOS takes a different path (see below).
 
---
 
## Computer-vision metrics
 
All of them work with a laptop webcam. The last column shows what AI_CAOS finally uses.
 
| Metric | How | Typical threshold | In AI_CAOS |
|---|---|---|---|
| **EAR** (eye aspect ratio) | vertical / horizontal eye-landmark distances | < ~0.2 = eye closed | replaced by **lid gap / iris diameter**, with a per-driver threshold from calibration |
| **PERCLOS** | % of time eyes are ≥80% closed over 60 s | > 15% = drowsy | dropped; replaced by slow blinks, blink rate and the combined "drowsy" alert |
| Blink rate and duration | from the eye-closure time series | long, slow blinks = fatigue | ✅ slow blinks > 0.4 s, blink rate < 5 / min |
| **MAR** (mouth aspect ratio) | mouth-landmark distances | yawn = MAR high for > ~1 s | replaced by MediaPipe's `jawOpen` blendshape (yawn) + lip gaps (talking) |
| **Head pose** | yaw / pitch / roll | looking away > ~2 s, pitch nodding | ✅ from MediaPipe's face transformation matrix; head drop and nods |
| Gaze | iris landmarks (MediaPipe) | off-road gaze zone | ✅ head + iris, gaze zones learned at calibration |
| Phone use | YOLO (`cell phone`) + hand near the ear / face | phone near the driver | ✅ YOLO26 (optional) + hand at the ear |
| Posture | MediaPipe Pose | slumping | not used; head tilt (roll) instead |
 
**Per-driver calibration** makes every threshold much more robust across people and camera positions. AI_CAOS
does this for every driver and saves it in the driver's profile.
 
---
 
## AI_CAOS vs Euro NCAP 2026
 
✅ = covered · 🟡 = partly covered · ❌ = not covered
 
| Euro NCAP requirement | AI_CAOS | Status |
|---|---|:---:|
| Long distraction: one glance away of 3–4 s | `long_glance` after 3 s, head and iris (eyes-only glances count too) | ✅ |
| Short distraction (VATS): 10 s away within 30 s | `on_road_ratio` is measured over the last 30 s, but no alert is raised from it yet | 🟡 |
| Phone use | YOLO phone in the hand / at the face, or a hand held at the ear | ✅ |
| Microsleep: eyes closed 1–2 s | `microsleep` at 1–3 s, plus repeated microsleep within 10 min | ✅ |
| Sleep: eyes closed ≥ 3 s | `sleep` at 3 s + emergency actions in code | ✅ |
| Drowsiness (KSS > 7) | combined `drowsy` alert: 2+ signs among yawning, eye rubbing, slow blinking, low blink rate, head nodding | 🟡 not validated against KSS |
| Unresponsive driver: controlled stop in lane | at 3 s of closed eyes: hazard lights, light braking (0.3), vibration, cruise off (simulated vehicle) | 🟡 brakes, but no stop manoeuvre |
| Alcohol / drug impairment | – | ❌ |
| Microsleep / sleep warnings more urgent than distraction | alert levels `critical` > `warning` > `info`; siren for microsleep, beeps for warnings | ✅ |
| Alert ends ≥ 2 s after the state ends | every alert is held 2 s after its condition ends (`ALERT_HOLD_S`) | ✅ |
| Driver told within 10 s if the camera is blocked | `camera_blocked` warning after 1 s | ✅ |
| Works with clear glasses | glasses are detected and the eyes are still checked through them | ✅ |
| Very dark sunglasses: tell the driver | sunglasses are detected and the eyes are no longer checked (no false microsleep); the camera window shows "eyes not detected (sunglasses)", but the driver gets no spoken notice | 🟡 |
| Talking must work | talking is detected and used for the `distraction` alert (talking while looking away) | ✅ |
| Light from 100,000 lux to 1 lux | automatic contrast boost (CLAHE) and a `low_light` alert; a normal webcam has no infrared, so real night driving is not possible | 🟡 |
| On by default for every trip | an **unknown face is not monitored** until it is added as a driver | ❌ (see gaps) |
| Face mask, beard, hat | not tested | – |
 
---
 
## Our recognition, profiles and glasses phase
 
### Phase 1: who is driving
 
- **Face recognition with OpenCV SFace:** each face becomes 128 numbers ("face print"). The current face is compared
  with the saved drivers twice a second; the same answer 3 times in a row (~1.5 s) switches the driver.
- Only frontal views are compared (head turned less than 25°), because side views match badly.
- **Passengers are ignored:** the largest face is followed, and the system waits for the driver instead of jumping
  to another face when he is briefly out of view.
### Phase 2: a profile per driver
 
- `profiles.json`: name, glasses yes / no, face prints. Stored **only locally**, gitignored.
- `calib_<name>.json`: **personal calibration**: normal head pose on the road, eye-closed threshold, yawn
  threshold and the driver's own gaze zones (road, left, right, up, down), learned in about 35 s.
- When the driver changes, his calibration is loaded and all the trackers start again, so one person's thresholds
  are never used for another.
### Phase 3: glasses
 
- **Clear glasses:** detected from the dark band the frame bridge makes on the nose, between the eyes.
- **Sunglasses:** the eye area becomes much darker than the cheeks, compared with the driver's own value from
  calibration. With sunglasses the eyes are not checked, so they cannot cause a false microsleep.
- **Glasses missing:** if the driver's profile says he wears glasses and none are seen, the `glasses_missing`
  warning starts ("Put your glasses on!") and the copilot reminds the driver.
- **Why it matters:** on an EU driving licence, **code 01.01** means the holder must drive with glasses
  (01.02 contact lenses, 01.06 glasses or contact lenses). Euro NCAP only asks that monitoring *works* with
  glasses; it never checks whether a driver who **needs** them is wearing them.
 
---
 
## Where we differentiate
 
| Idea | Why it is different |
|---|---|
| **The profile changes the safety, not only the seat** | Production cars use face recognition for comfort settings. AI_CAOS loads the driver's own eye, yawn, head-pose and gaze calibration, so the thresholds fit the person. |
| **Glasses-required check** | Links the profile to the licence restriction (code 01): the car can remind the driver to put the glasses on. Not part of Euro NCAP or of the systems listed above. |
| **LLM copilot with context** | The message depends on the driver state, the last 10 minutes and the road situation, and it escalates when a warning is ignored, instead of the same fixed beep every time. |
| **The LLM proposes, the code decides** | Every message and action is checked (allowed actions, safe direction only, level at most one step higher, no actions for `info`). Emergencies never wait for the LLM. |
| **Fully offline and private** | Vision, recognition and the LLM all run on the laptop. Face prints never leave the device. |
| **Extra fatigue signs** | Eye rubbing, hand at the mouth and head tilt are detected, although Euro NCAP does not assess them. |
| **No false microsleep when the eyes are hidden** | Sunglasses, a hand on the eyes, a covered camera or a head turned too far stop the eye check instead of raising a false alarm. |
 
### Ideas for later
 
- **Conversational alertness check:** the copilot asks a short question; reply time and coherence feed the attention score.
- Break suggestions that explain *why* (for example "3 yawns and 2 slow blinks in 5 minutes").
- Optional RAG over a small "car manual / traffic rules" text for passenger questions.
---
 
## Gaps and next steps
 
Ordered by how much they matter for Euro NCAP.
 
1. **Monitor unknown faces too.** Euro NCAP wants monitoring on for every trip. An unknown driver should be
   monitored with default thresholds (and offered to save a profile), not ignored.
2. **VATS alert:** raise a warning when the time looking away reaches 10 s within 30 s (`on_road_ratio` already
   measures the window).
3. **Unresponsive driver:** after the emergency actions, simulate a full stop in the lane (under 10 km/h), and
   detect "no reaction within 3 s of a warning".
4. **Spoken sunglasses notice:** tell the driver once that the eyes cannot be checked through dark sunglasses.
5. **Drowsiness validation:** compare the `drowsy` alert with KSS self-ratings in test sessions.
6. **Impairment (alcohol / drugs):** not covered; would need new signals (for example gaze stability, reaction
   times) and data.
7. **Night driving:** needs an infrared camera; a laptop webcam cannot reach 1 lux.
---

## Sources
 
- Euro NCAP, *Safe Driving – Driver Engagement Protocol v1.2* (July 2026): https://cdn.euroncap.com/cars/assets/Euro_NCAP_Protocol_Safe_Driving_Driver_Engagement_v1_2_ebce03a443.pdf
- Euro NCAP, *Euro NCAP announces 2026 protocol changes* (26 Nov 2025): https://www.euroncap.com/en/press-media/press-releases/euro-ncap-announces-2026-protocol-changes-to-tackle-modern-driving-risks
- ETSC, *Euro NCAP: New 2026 protocols target distraction, impairment, and speeding* (Jan 2026): https://etsc.eu/euro-ncap-new-2026-protocols-target-distraction-impairment-and-speeding/
- Smart Eye, *What's changing in Euro NCAP's 2026 safety ratings?*: https://smarteye.se/blog/euro-ncap-2026-whats-changing/
- Smart Eye, *Driver Monitoring 2.0: How Euro NCAP is raising the bar in 2026*: https://smarteye.se/?p=14540
- Commission Directive (EU) 2015/653, driving licence codes (01.01 glasses): https://legislation.gov.uk/eudr/2015/653/annex/data.html
- Subaru DriverFocus: https://www.secureidnews.com/?p=33774
- Mazda Driver Personalization System (owner's manual): https://www.mazda.ca/en/digital-owners-manual/2026/cx-5/contents/3809166475.html
- Hyundai Mobis driver identification: https://optics.org/news/10/7/25
- Genesis GV60 Face Connect: https://v.daum.net/v/20210917121200445
---
 
