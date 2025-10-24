# AI_COLLAB – Shared context for assistants

## What this project contains
- **paku-iot** = host side (Docker stack: Mosquitto, Postgres, Adminer, OTA static server).
- **paku-core** = ESP32 firmware (a.k.a. “EDGE/CORE” device code).

## Canonical paths & commands (Mac)
- **Project root:** `/Users/jossu/GIT/paku`
- **Host repo:** `/Users/jossu/GIT/paku/paku-iot`
- **Firmware repo:** `/Users/jossu/GIT/paku/paku-core`
- **Compose dir:** `/Users/jossu/GIT/paku/paku-iot/compose`

- Bring up dev stack:
  ```bash
  cd /Users/jossu/GIT/paku/paku-iot/compose
  docker compose -f dev.yaml up -d
  ```

- Mosquitto config used by dev:
  - `/Users/jossu/GIT/paku/paku-iot/compose/mosquitto/mosquitto.conf`

> **Note:** Paths inside `dev.yaml` are **relative to the compose dir** and still correct (`../data`, `../services`). No change needed there.

## Secrets & safety (DO NOT COMMIT)
- Device: `/Users/jossu/GIT/paku/paku-core/include/secrets.h` (gitignored).
- Host: `/Users/jossu/GIT/paku/paku-iot/compose/.env` for DB creds (gitignored).
- If suggesting commands or files, assistants must **never** print real secrets. Use placeholders.

## Branch policy
- Long-lived **dev** branch is OK (solo developer).
- PRs optional; can fast-forward merge dev→main when stable.

## Current decisions
- Using **Docker Desktop** locally.
- Using `compose/dev.yaml` (paths use `../data` and `../services` relative to `compose/`).
- MQTT broker: Eclipse Mosquitto on port 1883 with local persistence.
- Postgres 15 with Adminer on `http://localhost:8080`.
- OTA static server on `http://localhost:8081` (serving `/Users/jossu/GIT/paku/paku-iot/services/ota`).

## Open tasks (short)
1) Finish installing Docker Desktop.
2) Start stack:
   ```bash
   cd /Users/jossu/GIT/paku/paku-iot/compose && docker compose -f dev.yaml up -d
   ```
3) Verify broker works:
   ```bash
   docker exec -it paku-mosquitto mosquitto_sub -h localhost -t '$SYS/broker/version' -C 1 -W 5
   ```
4) Create `.env` in `/Users/jossu/GIT/paku/paku-iot/compose` when we’re ready (never commit).
5) Flash ESP32 from `/Users/jossu/GIT/paku/paku-core` with PlatformIO.

## How assistants should use this file
- **Always read/summarize this file first** before proposing changes.
- Propose commands with **full absolute paths** as above.
- Keep changes minimal and aligned with decisions here.
- Topics that have not yet been agreed on, but require discussion should be categorized and discussed under "Discussion - open topics". This is where improvement suggestions, found problems, etc. are added and discussed befor making decision. Once the decision is made and approved by human the topic is moved to decisions and removed from discussion. Everyone should tag themselves so that it is clear who made the comment or raised the topic. Human is called Jossu and uses @Jossu.

## Discussion - open topics