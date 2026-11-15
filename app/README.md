# BLINC app

The desktop app. Electron, plain JavaScript, no framework.

## Run it

From the repository root, `./start.sh` does everything. To run only the app:

```bash
cd app
npm install
cp .env.example .env      # DEEPSEEK_API_KEY and ELEVENLABS_API_KEY
npm start
```

API keys are read by the main process only. They never reach the page that draws the interface.

## Layout

| File | What it does |
|---|---|
| `main.cjs` | Electron main process: window, saved settings, and the calls that need API keys |
| `preload.cjs` | The small, fixed set of functions the interface is allowed to call |
| `src/face-tracker.js` | Webcam + MediaPipe; turns each frame into eyelid, eyebrow and gaze readings |
| `src/face-lock.js` | Keeps following the calibrated user's face and ignores everyone else |
| `src/closure.js` | Rescales raw eyelid readings to one person's open and closed levels |
| `src/calibration.js` | The 25-second calibration: resting face, eyes closed, eyebrows |
| `src/gesture-engine.js` | Readings in, gestures out. Pure logic, no browser needed, fully unit tested |
| `src/mapping.js` | Which gesture does what on each screen. One table, easy to change |
| `src/app.js` | The screens: Home, Drive, Talk, Snake, Settings, Calibrate, Practice, Dev |
| `src/audio.js`, `src/mic-worklet.js` | Microphone capture and voice detection for Talk |
| `lib/llm.js` | Builds the prompt and asks the language model for three replies |
| `lib/speech.js` | Speech to text and text to speech |
| `lib/butter-client.js` | Talks to the robot: heartbeat, drive commands, speed |

## Changing what a gesture does

Edit the table in `src/mapping.js`. For example, the Talk screen:

```js
talk: {
  BROW_TAP: 'cycle',      // quick eyebrow raise: next reply
  BROW_SELECT: 'say',     // hold eyebrows 1 s: say it
  DOUBLE_BLINK: 'say',
},
```

## Tests

```bash
npm test          # gesture engine (simulated people) and services
npm run e2e       # end-to-end, needs a recorded face video and voice clip (see test/e2e/run-e2e.js)
```

`test/sim.js` is the simulator: it generates eyelid and eyebrow readings for different kinds of faces, with natural blinking, dropped frames and noise.

## Debug recordings

Calibration, Practice, Drive, Snake and the Dev tab save their raw readings to `app/debug/` when you leave the screen. The folder is ignored by git. We used these recordings to tune the controls on real faces, and they are how we found out that winks were not reliable enough to keep.
