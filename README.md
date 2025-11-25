
---

Paku Monorepo - Development Overview

Workspace
Always open this repository via the workspace file:
paku.ws.code-workspace

Code is edited in VS Code with help of copilot

Docker CLI is intentionally not installed inside the devcontainer.  
All docker-compose commands must be run on the host (not inside dev container).

This ensures:
	•	unified VS Code devcontainer environment
	•	both paku-iot (cloud stack) and paku-core (ESP32 firmware) visible
	•	correct folder mappings inside the container
	•	fully reproducible development setup

---

Runtime pipeline:
ruuvi-emulator → mosquitto → collector → postgres → grafana

---

Repository Structure

paku/
|
|- paku-iot/                      # Cloud-side stack (MQTT → collector → Postgres → Grafana)
|   |
|   |- stack/                     # New unified runtime stack
|   |   |- mosquitto/             # MQTT broker container (config later)
|   |   |- ruuvi-emulator/        # Fake Ruuvitag publisher
|   |   |- collector/             # MQTT → Postgres ingestion service
|   |   |- postgres/              # Postgres container (volume: paku_pgdata)
|   |   |- grafana/               # Grafana visualization (volume: paku_grafana)
|   |
|   |- compose/
|   |   |- stack.yaml             # Unified compose file for the entire cloud stack
|   |
|   |- docs/
|   |
|   |- _archive/                  # Old, unused material kept for reference
|       |- legacy_compose/
|       |- services/
|
|- paku-core/                     # ESP32-S3 firmware (PlatformIO)
|   |- src/
|   |- include/
|   |- platformio.ini
|
|- .devcontainer/                 # Shared dev environment for the whole monorepo
|   |- devcontainer.json
|   |- Dockerfile
|   |- docker-compose.yml         # Legacy, not used today
|
|- paku.ws.code-workspace         # Unified workspace (always open this)
|
|- AI_COLLAB.md
|- TASKS_FOR_AI.md
|- README.md

---

What Each Major Component Does

paku-iot/stack/
	•	ruuvi-emulator: sends fake MQTT sensor data until hardware pipeline is ready
	•	mosquitto: MQTT broker
	•	collector: subscribes to MQTT and writes points into Postgres
	•	postgres: main persistent data store
	•	grafana: visualization UI

paku-iot/compose/stack.yaml
Single entrypoint to run the entire cloud stack:
docker compose -f compose/stack.yaml up --build

paku-core/
ESP32 firmware project (PlatformIO). Reads sensors, formats telemetry, sends via MQTT.

.devcontainer/
Defines the reproducible dev environment used for both paku-iot and paku-core.

_archive/
Contains old services, old compose files, old dashboards.
Not used in the new architecture, but preserved for reference.

---

Development Workflow
	1.	Open the workspace: paku.ws.code-workspace
	2.	VS Code → “Reopen in Container” → choose Yes.
	3.	Devcontainer provides environment for editing code for:
			•	paku-iot (cloud)
			•	paku-core (ESP32)
	4. Run Docker on host
