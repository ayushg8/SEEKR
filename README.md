# SEEKR

A local-first ground-control station and simulator for a search-and-rescue drone swarm. It runs an event-sourced mission backend, a deterministic simulator with scripted faults, a React operator console, and a sandboxed local-AI advisor — all on one laptop, with hardware actuation deliberately blocked. The runnable app is in [`software/`](software/).

## What it is

SEEKR is the software a human operator would use to run a GPS-denied SAR drone search: track drones on a map, fuse their detections and occupancy maps, allocate search tasks, and replay any mission deterministically afterward for review. It is a ground-control + simulation + evidence system — not flight firmware. It commands no real hardware; the hardware boundary is hard-blocked in code.

## How it works

- **Event-sourced backend** (`software/src/server/`) — every mission action is a hash-chained event with startup verification and fail-closed restore. State streams live to the UI over WebSocket (Express 5 + `ws`).
- **Deterministic simulator** (`src/server/sim/`) — seeded RNG and scripted faults (link loss, etc.) drive drone movement, detections, and map deltas, so any scenario is exactly repeatable and replayable.
- **Domain logic** (`src/server/domain/`) — validators with battery-reserve / link / estimator thresholds, map fusion that rejects stale or low-confidence transforms, a task allocator, and a passive planner.
- **Flight core + SITL** (`src/flight/`) — an onboard-executive state machine (arm / takeoff / waypoint / RTH / land) with failsafe evaluation, run against PX4 / ArduPilot software-in-the-loop. Actuation is blocked in `safety.ts`.
- **AI advisor** (`src/server/ai/`) — a local Ollama model can only choose among pre-validated candidate plans built by deterministic rules; it cannot call command APIs. Falls back to a rules engine when the model is absent.
- **Operator console** (`src/client/`) — a React 19, map-first UI with drone / alert / detection / zone panels, a lazy-loaded Three.js spatial viewer, and live + replay modes.
- **Read-only adapters** (`src/server/adapters/`) — MAVLink (binary v1/v2 frame decode) and ROS2 SLAM / occupancy-grid ingest that post only to read-only endpoints.

Design rationale, safety analysis, and the regulatory case live in [`decision-doc/`](decision-doc/).

## Tech

Node 20+, TypeScript 5.9 (strict, ESM), Express 5, `ws`, Zod. React 19 + Vite + Three.js front end. Vitest + Playwright (~60 test files). Local AI via Ollama — no external API, zero cloud cost.

## Running it

```bash
cd software
npm ci
npm run setup:local
npm run dev          # starts the backend + operator console
```

See [`software/docs/OPERATOR_QUICKSTART.md`](software/docs/OPERATOR_QUICKSTART.md) for the operator flow and [`software/docs/DEVELOPER_QUICKSTART.md`](software/docs/DEVELOPER_QUICKSTART.md) for the development and audit workflow.

## Honest limits

This is simulation and ground control, not a flying drone. Command upload and hardware actuation are disabled. Real Jetson/Pi validation, MAVLink/ROS bench telemetry, HIL logs, and reviewed hardware-actuation policy are all still required before the system could be treated as physically complete.

## TODO — visual asset to add

- `docs/img/console.png` — the operator console running a simulated search (map + drone tracks + detections). Reference it here once added.
