# CPR Trainer Specs

Browser based AR response simulator for a hackathon demonstration with CPR mannequins and AED trainers. This is **not a medical device** or a live emergency response service.

## Run locally

Requires Node.js 20 or later.

```bash
npm install
npm start
```

Open [http://localhost:4173](http://localhost:4173). Run `npm test` for the state engine checks.

## Demo flow

1. Activate emergency training mode.
2. Record the observed mannequin state: unresponsive and not breathing normally.
3. Connect the **mock** dispatcher. The app never dials 911.
4. Select a fictional AED trainer location and inspect the simulated route.
5. Start the visual CPR rhythm. Tap the button or press Space to see practice tap rate. The app does not measure compression depth or recoil.
6. Mark the AED trainer as arrived, then confirm its training steps. The AED trainer gives the operative prompts; the app never decides whether a shock is needed.
7. Copy the session URL and open it in another browser or phone on the same reachable host. Assign roles, then close a teammate's tab to show role reassignment.
8. In **Training Assistant**, ask “What’s next?” or “Who is doing what?” The answer reads the live demo session. CPR, AED, mock calling, and real-emergency questions use fixed training copy.

For a second device on the same Wi-Fi, find the Mac's LAN address and open `http://<your-mac-ip>:4173/?session=<same-id>`. The server listens on port 4173; a local firewall prompt may appear.

## Training assistant API

The browser demo exposes `POST /api/assistant` and a question panel. Without an API key it returns a clearly labeled local scripted answer, so the demo works offline. With `OPENAI_API_KEY` set in the server process, team-role summaries can be rephrased by the OpenAI Responses API; `OPENAI_MODEL` defaults to `gpt-6-luna`. Restart `npm start` after setting either variable. The server sends only an approved role summary and language to the model, not the raw question, audio, camera frames, or the full session. The API key stays on the server. Model errors and timeouts fall back to the scripted answer.

The model never updates workflow state or answers CPR, AED, simulated-call, or real-emergency questions. This is an explanatory demo feature, not an emergency assistant. The current local HTTP server is for a trusted demo network; connecting the Spectacles Lens to it will require a reachable HTTPS deployment and on-device testing.

## Architecture

- `shared/engine.js`: deterministic event reducer, AED demo locations, task model, and training sequence.
- `server.js`: small Node HTTP and WebSocket server. It owns session state, broadcasts snapshots, and serves the simulator.
- `public/`: responsive browser interface with an AR-style training view, simulated floor map, optional browser speech commands, and shared team controls.
- `tests/`: critical state transitions and guardrails.
- `shared/training-assistant.js`: bounded question routing and current-state answers. The server can optionally rephrase role summaries through a model API.

Session state is kept in server memory and resets when the server restarts. There is no account system or durable data store. The WebSocket endpoint has no authentication and is intended for a trusted hackathon demo network only.

## Real vs simulated

| Capability | Current implementation |
| --- | --- |
| Emergency call | Mock dispatcher only; no phone integration or dialing |
| AED locations | Fictional training building; no real registry or verified availability |
| CPR rhythm | Visual 110 BPM metronome and keyboard or button taps |
| CPR quality | No depth or recoil measurement; no real sensor adapter |
| AED analysis | Training sequence only; no shock decision |
| AR view | Browser simulation and a Spectacles training overlay; no spatial mannequin alignment yet |
| Voice | Optional browser speech recognition when available; manual controls always present |
| Multi-person | Live WebSocket session and task assignment; no access control |
| Language model | Optional server-side Responses API for team-role wording; scripted fallback is always available |

Medical copy is limited to an adult training scenario and follows [American Heart Association adult basic life support guidance](https://cpr.heart.org/en/resuscitation-science/cpr-and-ecc-guidelines/adult-basic-life-support) and [Red Cross AED steps](https://www.redcross.org/take-a-class/aed/using-an-aed/aed-steps). In any real emergency, call local emergency services directly and follow dispatcher and AED instructions. Do not wait for this prototype.

## Spectacles prototype

The native [Lens Studio project](./spectacles/SecondChance/SecondChance.esproj) runs on connected Spectacles (2024). Lens Studio 5.15.4 sent the current illustrated version over USB and logged a successful launch with no Logger errors. Pinching thumb and index finger advances five training screens: check response, call 911 and get an AED, place hands, push hard and fast, and use an AED. The wearer verified pinch in an earlier version; fit and image visibility of the current version still need wearer confirmation. The script attempts voice commands, but voice recognition has not been verified on Spectacles. The native Lens has no camera tracking, model API, real call, or mannequin sensor integration. The [exported Lens zip](./spectacles/build/SecondChance.zip) contains the current script and seven illustration textures.

Open the project in Lens Studio while the app is closed, connect Spectacles over USB, and click **Preview Lens**. See [Snap's wired connection guide](https://developers.snap.com/spectacles/get-started/start-building/connecting-lens-studio-to-spectacles). The editor's desktop preview cannot run the wearable-only pinch API; test hand input on the glasses. Lens Studio 5.15.4 has intermittently crashed during scene edits on this Mac, so the project script is maintained as a source file and the exported zip is rebuilt from it.

See [project.md](./project.md) for status and next steps.

See [DEMO.md](./DEMO.md) for the timed six-reveal presentation script.
