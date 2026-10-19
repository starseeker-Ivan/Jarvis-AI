Guys there's a problem with the file uploading. Give me a few days, I'll fix it.

After you install this, remember to take the folder outside the Jarvis-AI file.

# Jarvis — Personal AI Assistant (macOS)

A futuristic desktop AI assistant inspired by *concept* of JARVIS — but an

original assistant with its own name, personality, interface and codebase.

Talk to it with your microphone, type commands, and let it safely operate

your Mac.



```
IDLE ● LISTENING ● THINKING ● SPEAKING ● EXECUTING ● ERROR
```

## Features

**Voice**



* Hold-to-talk (mic button or **Space** when the input box is empty), continuous VAD listening, or wake-word mode (`LISTENING_MODE` in `.env`)

* Speech-to-text: OpenAI-compatible Whisper API **or** fully offline `faster-whisper`

* Text-to-speech: macOS `say` (offline, default) **or** Edge neural voices

* Interruption: start talking / press **Esc** and Jarvis stops speaking instantly

* The conversation panel shows both what you said and what Jarvis understood

**AI brain**



* Any OpenAI-compatible LLM (OpenAI, DeepSeek, Groq, local Ollama…)

* Conversation context + follow-up questions + remembered facts ("remember my name is Ivan" persists across restarts)

* Honest failure: Jarvis explains when it can't do something, never fakes success

* Graceful offline fallback: if the API fails, basic commands still work

**Computer control (permission-tiered)**



* Open/close apps, open websites, web search, read/create/append/list files

* Screenshots, system info (CPU/RAM/network/battery), calculations, timers & reminders, volume, music control (Music/Spotify), weather

* Delete / edit-existing-file / organize-files / shutdown / restart / sleep → **always ask for confirmation first**

* Arbitrary shell execution, system-settings changes, software installs → **blocked entirely**

**Vision**



* "Look at my screen" / "analyze this image: \~/Pictures/x.png" via a vision-capable model

* Jarvis only describes images it actually received — no guessing

**Interface**



* Dark sci-fi UI with a glowing, state-reactive central visualizer: animated particles, rotating arcs, circular audio waveform, ripple effects, error flash

* Live status: state, mic level, CPU/RAM/network/battery chips, clock & date

* Smooth \~30 fps animations, no interface freezes (everything async)

## Project structure



```
Jarvis/
├── main.py                 # entry point (+ --self-test headless check)
├── assistant.py            # AssistantCore orchestrator (threads, state, confirm gate)
├── config.py               # all settings, loaded from .env
├── requirements.txt        # core dependencies
├── requirements-optional.txt
├── .env.example            # copy to .env and fill in
├── ai/
│   ├── brain.py            # LLM client, action-JSON contract, offline brain, safe math
│   └── memory.py           # session history + persisted facts
├── voice/
│   ├── audio.py            # microphone engine + VAD + wake word
│   ├── speech_to_text.py   # Whisper API / faster-whisper
│   └── text_to_speech.py   # macOS say / edge-tts / OpenAI TTS (interruptible)
├── computer/
│   ├── apps.py             # apps, websites, media (AppleScript, no shell strings)
│   ├── files.py            # safe file operations (Trash-friendly delete)
│   ├── system.py           # sysinfo, screenshots, volume, timers, power
│   ├── security.py         # SAFE / CONFIRM / BLOCKED tiers
│   └── executor.py         # action registry + permission gate
├── vision/
│   └── vision.py           # screenshot & image analysis via vision LLM
└── ui/
    ├── interface.py        # main futuristic window
    ├── visualizer.py       # glowing central visualizer (QPainter)
    ├── signals.py          # thread-safe signal bus
    └── theme.py            # colors / stylesheet
```

## 1. Installation

Requires **Python 3.9+** (macOS system Python 3.9 works; Homebrew Python also fine).



```
cd ~/Documents/Jarvis

# create a virtual environment
/usr/bin/python3 -m venv .venv

# activate it (do this every time you run Jarvis)
source .venv/bin/activate

# install core dependencies
pip install -r requirements.txt
```

Optional upgrades (pick what you want):



```
pip install -r requirements-optional.txt
# faster-whisper  -> fully offline speech recognition (downloads a model on first use)
# edge-tts        -> more natural voices (needs internet)
# openwakeword    -> wake-word mode ("hey computer", "jarvis", ...)
```

## 2. API setup

Copy `.env.example` to `.env` and fill in what you use. **Never** commit `.env`.



```
cp .env.example .env
```

Minimal configuration for full intelligence — **OpenAI**:



```
OPENAI_API_KEY=sk-...
```

Other providers (any OpenAI-compatible endpoint):



| Provider       | OPENAI\_BASE\_URL                       | LLM\_MODEL                | Notes                                                     |
| -------------- | --------------------------------------- | ------------------------- | --------------------------------------------------------- |
| OpenAI         | *(default)* `https://api.openai.com/v1` | `gpt-4o-mini`             | also gives Whisper STT + vision                           |
| DeepSeek       | `https://api.deepseek.com/v1`           | `deepseek-chat`           | cheap; no STT/vision                                      |
| Groq           | `https://api.groq.com/openai/v1`        | `llama-3.3-70b-versatile` | fast Whisper STT too                                      |
| Ollama (local) | `http://localhost:11434/v1`             | `qwen2.5:3b`              | set `OPENAI_API_KEY=ollama`; fully offline, no key      |

### Local brain via Ollama (currently active in this project)

The project is already set up and verified against a local Ollama server:

1. The app is installed at `/Applications/Ollama.app` (server v0.34.4).
2. CLI/server binary lives inside the bundle - always run it from there (a copy
   elsewhere cannot find `llama-server`):
   ```bash
   /Applications/Ollama.app/Contents/Resources/ollama serve
   ```
3. Model already pulled: `/Applications/Ollama.app/Contents/Resources/ollama pull qwen2.5:3b`
4. `.env` is already configured:
   ```dotenv
   OPENAI_API_KEY=ollama
   OPENAI_BASE_URL=http://localhost:11434/v1
   LLM_MODEL=qwen2.5:3b
   ```
5. Verified: `curl http://localhost:11434/api/version` -> `{"version":"0.34.4"}`,
   and the app's own brain answers "What's your name?" with "My name is Jarvis."

> No API key at all? Jarvis still runs — in
> **offline mode**
> with basic commands
> (time, date, math, open apps/websites, search, screenshots, system status,
> timers, volume, music).

Other useful `.env` options:



```
ASSISTANT_NAME=Jarvis
TTS_ENGINE=macos_say        # macos_say | edge_tts | openai_tts
TTS_VOICE=Samantha          # macos: Samantha/Daniel/... ; edge: en-US-AriaNeural
LISTENING_MODE=push         # push | continuous | wake
STT_ENGINE=auto             # auto | whisper_api | faster_whisper
LLM_MODEL=gpt-4o-mini
VISION_MODEL=gpt-4o-mini
```

## 3. macOS permissions (do this once)



1. **Microphone** — System Settings → Privacy & Security → Microphone → allow your terminal.

2. **Screen Recording** — System Settings → Privacy & Security → Screen Recording → allow your terminal (needed only for `take_screenshot` / `analyze_screen`).

3. **Automation** — if you use shutdown/restart, macOS will ask to allow your terminal to control *System Events* → click Allow.

4. No extra permission is needed for `say`, volume, or opening apps.

## 4. Run



```
cd ~/Documents/Jarvis
source .venv/bin/activate
python3 main.py
```



* **Hold the ● MIC button** (or hold **Space** when the input box is empty) to talk — release to send.

* Type in the box and press **Enter** for keyboard input.

* **Esc** interrupts Jarvis mid-sentence.

Quick self-check before running (no GUI, no microphone):



```
python3 main.py --self-test
```

## 5. Example commands

Voice or typed:



* "Open Safari" · "Open [youtube.com](https://youtube.com)" · "Search for quantum computing news"

* **Workspaces** · "Set up my workspace" / "Open the work workspace" / "Open workspace dev" —
  opens a whole preset (apps + urls) in one command. "Save workspace as dev" snapshots your
  currently-open apps (needs Accessibility permission). "Close workspace" quits only the apps
  in the preset. Presets live in `workspaces.json` in the project root (defaults: `work`,
  `dev`, `daily`) — edit the file to customize; "List workspaces" shows them.

* **Phone bridge** · "Ping my phone" · "Notify my phone that dinner is ready" · "Send a
  notification to my phone saying coffee's done" — pushes a notification to your Android
  phone via [ntfy](https://ntfy.sh). Setup (once):

  1. Install the free **ntfy** app on Android.
  2. Subscribe to the topic `jarvis-ivanpoon` (or whatever `NTFY_TOPIC` is in `.env`).
  3. Say "Ping my phone" — done. Edit `NTFY_SERVER` / `NTFY_TOPIC` / `NTFY_TOKEN` /
     `NTFY_PRIORITY` in `.env` to taste; set `NTFY_ENABLED=false` to disable.

* "What's the weather in Tokyo?"

* "How much is 17% of 430?" · "What is sqrt(144) plus 6?"

* "Set a timer for 5 minutes" · "Remind me at 3pm to call Alex" · "What timers are active?"

* "Take a screenshot" · "Look at my screen" · "Analyze this image: \~/Pictures/photo.png"

* "What's my CPU usage?" · "System status"

* "Read \~/Documents/notes.txt" · "Create a file called todo.txt with: buy milk"

* "Turn the volume to 40" · "Play" · "Pause music" · "Next track"

* "Remember my name is Ivan" · "What's my name?"

* "Organize my Downloads folder" · "Delete \~/Desktop/old.txt" · "Shut down the computer"

  (the last three **require your on-screen confirmation**)

* "What can you do?"

### Games (arcade + talk games)

* "Play a game" - opens the arcade menu (Asteroid Defense, Target Practice)
* "Play asteroid defense" - arcade shooter (arrows/A+D move, space fire, ESC exit)
* "Play target practice" - 60 s click-the-targets minigame
* "Play the guessing game" - I think of a number 1-100; say "is it 42"
* "Rock paper scissors" - best-of-three; say rock / paper / scissors
* "Stop the game" / ESC - exits any game

### Random disaster events — now FIGHTABLE

Every 4-9 minutes a random disaster fires on its own — and every disaster
opens a **battle arena**: you actively defend the system with mouse + keyboard.
You either neutralize the threat (win) or take the hit (lose), and Jarvis
comments on the outcome. The six fightable events:

| Event | How to fight |
| ----- | ------------ |
| Alien invasion | Cannon defense: aim with mouse or ←/→/A/D, SPACE or click to fire lasers at descending UFOs |
| Meteor storm | Cannon defense: shoot meteors — they split into smaller fragments on impact |
| Drone raid | Cannon defense: cut down the hostile swarm |
| Ghost signal | Anomaly purge: click the red hostiles, NEVER the blue decoys (they cost a life) |
| EMP strike | Reboot sequence: click the power junctions as they light up, before they overload |
| Solar flare | Parry timing: press SPACE as the expanding flare ring crosses the shield ring |

All battles: **ESC** abandons (counts as a loss). Three hearts, a threat
progress bar, and a hard time cap — a battle can never hang forever.

Trigger or control them by voice:

* "alien invasion" / "meteor storm" / "drone raid" / "ghost signal" /
  "emp strike" / "solar flare" - fire a specific disaster now
* "random event" / "surprise me" - random disaster
* "disable random events" / "enable random events" - toggle the auto-scheduler
* **EVENT ▸ button** in the bottom bar, or **Ctrl+E** - fire one manually at
  any time (works even while the auto-scheduler is disabled)

Tuning lives in `config.py`: `RANDOM_EVENTS_ENABLED`, `RANDOM_EVENT_MIN_S=240`,
`RANDOM_EVENT_MAX_S=540`. Events pause while you're in a game or a confirmation.

### Easter eggs & events

Every easter egg, game and event below is voice/type-activated and pure
visual fun — see the **complete 32-egg reference table** in
[section 8 — Easter eggs](#8-easter-eggs).

* "solar eclipse" - the moon crosses the sun, stars come out, diamond ring
* "lightning storm" / "call the storm" - jagged bolts, rain, ground flashes
* "shockwave" / "repulsor blast" - expanding energy rings + screen shake
* "drone swarm" / "deploy drones" - a formation of glowing drones

## 6. Troubleshooting



| Problem                                 | Fix                                                                                                                             |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `ModuleNotFoundError: PySide6`          | Make sure `.venv` is active and `pip install -r requirements.txt` succeeded                                                     |
| Mic error / "no input device"           | Grant Microphone permission in System Settings, then restart Jarvis. Check `MIC_DEVICE` only if you have multiple mics            |
| Whisper API 401                         | Check `OPENAI_API_KEY` in `.env`; for non-OpenAI providers also set `OPENAI_BASE_URL`                                           |
| No voice output                         | `say` should work out of the box. Check system volume; try `TTS_VOICE=Samantha`; `TTS_ENGINE=edge_tts` for a more natural voice |
| faster-whisper downloads slowly         | It downloads the model once (e.g. `small` ≈ 460 MB) — be patient or switch to `tiny`/`base` via `WHISPER_LOCAL_MODEL`           |
| Screenshot fails                        | Grant Screen Recording permission; restart the terminal/Jarvis afterwards                                                         |
| Shutdown/restart does nothing           | Grant Automation permission for System Events; otherwise those actions are refused safely                                       |
| "Confirmation timed out"                | You have 60 s to click Confirm/Cancel (set `CONFIRM_TIMEOUT_SECONDS`)                                                           |
| Wake word never triggers                | `pip install openwakeword`, set `LISTENING_MODE=wake`, adjust `WAKE_WORD_SENSITIVITY` (0.3 = more sensitive)                    |
| Assistant replies oddly in offline mode | Add `OPENAI_API_KEY` to `.env` — offline mode only handles basic commands                                                       |
| PortAudio errors                        | `brew install portaudio` then reinstall sounddevice                                                                             |

## 7. Security model



* The LLM can only request actions from a **hard-coded whitelist**; it never executes code or shell commands.

* Every action is tagged **SAFE** (auto), **CONFIRM** (on-screen confirmation, 60 s timeout) or **BLOCKED** (refused — arbitrary shell, system settings, installs, app deletion).

* File paths are validated and resolved; subprocess calls always use argument lists, never shell strings.

* API keys live only in `.env` / environment variables.

* Deletion moves files to the **Trash** (recoverable), never permanent `rm`.

## Notes



* First run may be slow while the PySide6/Qt plugins load; subsequent starts are quick.

* Session facts persist in `session_memory.json`; screenshots land in `captures/`.

* This project targets macOS. Windows/Linux would need small changes in `computer/apps.py`, `computer/system.py` and the default TTS engine.


## 8. Easter eggs

Harmless, Iron-Man-flavored fun (original lines, nothing destructive). Works in
offline mode too. All 32 eggs live in one registry
(`computer/executor.py` → `EGG_REGISTRY`) — adding an entry there automatically
makes the brain recognize it and the UI label it.

Eggs with a **🎬 cutscene** play a 5-second cinematic video behind the HUD
(`ui/assets/cutscenes/<egg>.mp4`, auto-detected). **10 of the 32 eggs currently
ship a cutscene** (missiles, repulsor, flight, suit_up, clean_house, arc_reactor,
emp, lasers, warp, time_warp); the remaining 22 are queued and will be added as
they finish generating — drop the file in and it is picked up automatically.
Every other egg (and any egg whose video is missing) falls back to the built-in
procedural FX, so nothing ever "does nothing".

| Say | What happens |
|---|---|
| **🎬 "fire micro guns/missiles", "launch the missiles"** | radar lock → 3 missiles fire → staggered explosions + shockwave (cinematic cutscene) |
| **🎬 "repulsor", "hand beam", "unibeam"** | palm repulsor charges → focused beam discharge (cinematic cutscene) |
| **🎬 "flight mode", "fly mode", "engage flight", "take off"** | vertical ascent with engine trails (cinematic cutscene) |
| **🎬 "suit up", "armor up", "mark 42/7/85/3"** | armored suit assembles piece by piece + scan grid (cinematic cutscene) |
| **🎬 "clean house", "cleanup drones", "housekeeping"** | 8 patrol drones with scan beams (cinematic cutscene) |
| **🎬 "arc reactor", "power core", "reactor online"** | arc-reactor flare: rings collapse, vanes spin, white-hot core (cinematic cutscene) |
| **🎬 "fire an emp", "emp blast", "electromagnetic pulse"** | twin shockwave rings + white-out + afterglow (cinematic cutscene) |
| **🎬 "fire lasers", "shoulder cannon", "laser barrage"** | lock-on reticles, then laser beams sweep to targets (cinematic cutscene) |
| "stealth mode", "cloaking", "go dark" | cloak veil + scan grid shimmer + reveal |
| "overload", "max power", "reactor overload" | core heats up, then a very controlled explosion |
| "alien invasion", "mothership", "extraterrestrial" | red-alert cone scan → 3 UFOs lock on (fightable) |
| "meteor shower", "asteroid storm" | meteor streaks, fire bands, ground impacts (fightable) |
| "ghost signal", "static", "phantom signal" | static, glitch bands, cryptic scrolling text (fightable) |
| "solar flare", "coronal mass", "magnetic storm" | white-hot build-up → radial burst (fightable) |
| "drone swarm", "deploy drones", "swarm mode" | a formation of glowing drones orbiting the core |
| "solar eclipse", "blackout the sun" | moon crosses the sun, stars come out, diamond ring |
| "lightning", "thunderstorm", "storm protocol" | jagged glowing bolts, rain streaks, ground flashes |
| "shockwave", "repulsor blast", "sonic boom" | expanding energy rings + screen shake |
| "drone raid", "hostile drones", "drones inbound" | hostile swarm (fightable) |
| "emp strike", "kill the power", "grid outage" | power grid drops → reboot-sequence battle (fightable) |
| "engage warp", "warp speed", "hyperspace", "ludicrous speed" | **🎬** warp tunnel: ship punches into hyperspace, stars stretch into amber speed lines, violent emergence (cinematic cutscene) |
| "quantum tunnel", "wormhole", "dimensional rift" | quantum gate opens, light bends through it |
| "hologram", "holo projector", "show the schematic" | core schematic hologram, full fidelity |
| "digital rain", "enter the matrix", "hack the mainframe" | deliciously orange code rain |
| "black hole", "singularity", "event horizon" | dark disc + orange accretion ring + infalling light |
| "thanos snap", "snap my fingers", "the snap" | half the particles turn to dust — balanced |
| "fireworks", "celebrate", "party time", "new year" | programmed fireworks bursts |
| "disco", "party mode", "boogie", "stayin alive" | the core is now legally required to groove |
| "sonic cannon", "sonic blast", "resonance blast" | resonance wave away — ear protection recommended |
| "dum-e", "robot arm", "fetch the coffee" | DUM-E arm reaches in, claw extended |
| "time travel", "time warp", "rewind", "chrono" | **🎬** temporal rewind: chrono core spins up, time folds, reality snaps back (cinematic cutscene) |
| "heartbeat", "pulse check", "cardiac", "flatline" | cardiac monitor live, steady and confident |

These are pure UI/voice effects - no real missiles, no system changes.

## Incase of Ollama Crashed

Use 
nohup /Applications/Ollama.app/Contents/MacOS/Ollama > /tmp/ollama.log 2>&1 & disown

/
## Stop previous

pkill -f main.py
