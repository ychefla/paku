# AI Collaboration Guide (Paku Project)

## Roles

- **@Jossu (human developer)**  
  Owns scope, priorities and final decisions. Writes / approves and runs the code, merges branches, and decides when something is “done”.

- **@Architect-GPT (CHAT-GPT)**  
  Works only at architecture / configuration / documentation level. Designs stack layout, Docker Compose, devcontainer, directory structure and high‑level flows. May edit markdown, YAML, simple Dockerfiles and small glue scripts when explicitly asked. Does not implement business logic or large chunks of application code. Ensures repo cleanliness. Validates decisions.

- **Copilot / Implementer tools**  
  Generate and edit actual code inside the repositories (Python, C/C++, PlatformIO, etc.) following structures defined here. They are used from within VS Code by @Jossu. Obeys architecture designed by Architect-GPT, but may suggest improvements.

- **@Review-GPT (optional)**  
  Reviews diffs, looks for regressions, and comments on architecture & maintainability.

### Rules

- Architect-GPT:
  - focuses on big picture, not micro‑implementation
  - never rewrites large source files unless explicitly asked
  - prefers incremental, minimal changes that keep the repos clean
- Copilot:
  - writes the real logic (collector, emulator, firmware, APIs)
  - follows the directory structure and contracts defined here
- All assistants:
  - respect that the project is **personal, not a product**, and should stay lightweight
  - keep sprint scope small and avoid gold‑plating

### How collaboration works
- Jossu asks directly.
- Architect-GPT produces high-level structure (YAML, SQL, flow diagrams).
- Copilot writes implementation inside files.
- Architect-GPT never rewrites entire files unless instructed (oboe/edit_file).

### Files each agent can modify
- Architect-GPT: config, compose, documentation, devcontainer
- Copilot: python, c++, platformio, collector, emulator, firmware
- Human developer: final integrations

### Workflow per sprint
1. Human defines goal
2. Architect-GPT writes structure
3. Copilot writes code into correct repo/subfolder
4. Human tests stack
5. Human merges dev → main

## Current technical baseline (authoritative)

- Monorepo root (for common stuff): `/Users/jossu/GIT/paku`
- Repositories:
  - `/Users/jossu/GIT/paku/paku-iot` → cloud / backend stack
  - `/Users/jossu/GIT/paku/paku-core` → ESP32 firmware (to be refreshed in a later sprint)
- Unified runtime stack (no dev/prod split) lives in `paku-iot/compose/stack.yaml`.
- Services live in `paku-iot/stack/{mosquitto,ruuvi-emulator,collector,postgres,grafana}`.
- Local development:
  ```bash
  cd /Users/jossu/GIT/paku/paku-iot
  docker compose -f compose/stack.yaml up --build
  ```
  Docker is always run on the **host**, not inside the devcontainer.
- Devcontainer:
  - defined in `paku/.devcontainer`
  - always open `paku.ws.code-workspace` in VS Code
  - VS Code runs inside the devcontainer; stack runs on the host
- Postgres & Grafana use named volumes (`paku_postgres_gdata`, `paku_grafana_data`) for persistence.
- Mosquitto is intentionally simple and open for local development; all production‑grade hardening notes in this file are **future/optional**.

## Prerequisites
To ensure the project works correctly, all repositories must be cloned into the same parent directory. The expected structure is as follows:

## MQTT Broker – future production hardening (optional)

### TODO: Secure the MQTT Broker
- **Task**: Add authentication and TLS encryption to the Mosquitto broker in production.
- **Steps**:
  1. Generate or obtain the following files:
     - `passwords`: Use the `mosquitto_passwd` command to create a password file.
     - `ca.crt`, `server.crt`, `server.key`: Obtain or generate TLS certificates for encrypting communication.
  2. 
  3. Update your Mosquitto configuration to enable authentication and TLS (when you eventually add a hardened production setup):
     ```conf
     allow_anonymous false
     password_file /mosquitto/config/passwords
     listener 8883
     cafile /mosquitto/config/ca.crt
     certfile /mosquitto/config/server.crt
     keyfile /mosquitto/config/server.key
     ```
  4. Test the configuration in a staging environment before deploying to production.

### Best Practices for MQTT Broker Security
1. **Authentication**:
   - Always require username/password authentication in production.
2. **Encryption**:
   - Use TLS to encrypt communication between clients and the broker.
3. **Access Control**:
   - Use access control lists (ACLs) to restrict which clients can publish/subscribe to specific topics.
4. **Monitoring**:
   - Enable logging and monitor the broker for unusual activity.
5. **Firewall Rules**:
   - Restrict access to the MQTT broker to trusted IPs or networks.

> **Important**: Always test the production configuration in a staging environment before deploying it to production.
paku/
├── paku-iot/   # Host-side code (Docker stack: Mosquitto, Postgres, Adminer, OTA static server)
├── paku-core/  # ESP32 firmware (EDGE/CORE device code)
├── paku/       # Common documentation for the project
```

Clone the repositories into the `paku` directory:
```bash
git clone https://github.com/ychefla/paku-iot.git
git clone https://github.com/ychefla/paku-core.git
git clone https://github.com/ychefla/paku.git
```

## Canonical paths & commands
To make the project portable across different operating systems, use environment variables or relative paths. Set the `PROJECT_ROOT` environment variable to the root directory of the project:

```bash
export PROJECT_ROOT=~/GIT/paku  # Replace with the actual path on your system
```

### Example paths:
- **Project root:** `$PROJECT_ROOT`
- **Host repo:** `$PROJECT_ROOT/paku-iot`
- **Firmware repo:** `$PROJECT_ROOT/paku-core`
- **Compose dir:** `$PROJECT_ROOT/paku-iot/compose`

### Commands:
- Bring up the stack:
  ```bash
  cd $PROJECT_ROOT/paku-iot
  docker compose -f compose/stack.yaml up --build
  ```


## Secrets & safety (DO NOT COMMIT)
Secrets are critical for the operation of the project but must be handled securely. Below are the details for managing secrets for both the device and the host:

### Device Secrets
- **Location**: `$PROJECT_ROOT/paku-core/include/secrets.h` (gitignored).
- **Purpose**: Contains sensitive information such as Wi-Fi credentials, API keys, or other device-specific secrets.
- **How to create**:
  1. Copy the example file (if available) or create a new `secrets.h` file:
     ```bash
     cp $PROJECT_ROOT/paku-core/include/secrets.example.h $PROJECT_ROOT/paku-core/include/secrets.h
     ```
  2. Populate the file with the required secrets. Example:
     ```c
     #define WIFI_SSID "YourWiFiSSID"
     #define WIFI_PASSWORD "YourWiFiPassword"
     #define API_KEY "YourAPIKey"
     ```
  3. Ensure the file is **never committed** to version control.

### Host Secrets
- **Location**: `$PROJECT_ROOT/paku-iot/compose/.env` (gitignored).
- **Purpose**: Stores environment variables such as database credentials, MQTT broker credentials, and other host-side secrets.
- **How to create**:
  1. Copy the example file (if available) or create a new `.env` file:
     ```bash
     cp $PROJECT_ROOT/paku-iot/compose/.env.example $PROJECT_ROOT/paku-iot/compose/.env
     ```
  2. Populate the file with the required environment variables. Example:
     ```env
     POSTGRES_USER=your_db_user
     POSTGRES_PASSWORD=your_db_password
     MQTT_USER=your_mqtt_user
     MQTT_PASSWORD=your_mqtt_password
     ```
  3. Ensure the file is **never committed** to version control.

### Best Practices for Secrets Management
1. **Use a Secrets Manager**:
   - For production environments, use a dedicated secrets management tool such as:
     - [HashiCorp Vault](https://www.vaultproject.io/)
     - [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
     - [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/)
     - [Docker Secrets](https://docs.docker.com/engine/swarm/secrets/)
2. **Environment-Specific Secrets**:
   - Use separate secrets for development, staging, and production environments.
   - Avoid reusing secrets across environments.
3. **Access Control**:
   - Limit access to secrets to only those who need it.
   - Use role-based access control (RBAC) where possible.
4. **Audit and Rotate Secrets**:
   - Regularly audit who has access to secrets.
   - Rotate secrets periodically and immediately if a breach is suspected.

> **Important**: If suggesting commands or files, assistants must **never** print real secrets. Use placeholders instead.


## How assistants should use this file

- Always read/summarize the sections **“Current collaboration model”** and **“Current technical baseline”** before proposing changes.
- Propose commands with **full absolute paths** (based on `/Users/jossu/GIT/paku/...`) unless there is a very strong reason not to.
- Keep changes minimal and aligned with the current unified stack (`paku-iot/compose/stack.yaml`) and the monorepo structure described here.
- Topics that have not yet been agreed on, but require discussion, should be added under **“Discussion – open topics”** with a short tag like `[@Architect-GPT]` or `[@Implementer-GPT]`. Once @Jossu has decided, move the item into the relevant “decisions” or “archived” section and clean up duplicates.

## Roadmap Structure
This project uses three planning tiers:
- **Next 1–2 sprints** — Items likely to be implemented soon; small, high‑impact, low‑complexity.
- **Later / nice‑to‑have** — Useful but not required; do not influence current architecture.
- **Archived / not relevant** — Old dev/prod-specific ideas or over-complex features; kept only for reference.

## Next 1–2 sprints (high priority)
- CI + pre-commit (detect-secrets, yamllint, jsonlint).
- `.env.example` creation + secrets hygiene.
- Minimal Grafana dashboard provisioning folder (no Prometheus).
- VS Code tasks for common docker operations.
- Simple E2E test: spin stack, publish sample MQTT, verify row in Postgres.

## Later / nice‑to‑have (medium/low priority)
- Loki + Promtail centralized logs.
- Device registry (device_id metadata etc.).
- OTA bundle signing + version metadata.
- Dead‑letter queue for parse failures.
- Rollup jobs + retention policies.
- TimescaleDB migration option.
- Cloud ingress (Caddy/Traefik) + TLS + domain.
- Mosquitto exporter + `$SYS` metrics.
- Automated backups to S3/Backblaze.

## Archived / not relevant (kept for reference)
- Old dev/prod split (`dev.yaml`, `prod.yaml`, cloud.yaml drafts).
- Prometheus + alert rules (not in scope for unified local stack).
- Adminer/OTA legacy stack components.
- Codespaces-specific compose overrides.
- Reverse proxy routing logic for remote cloud deployment.

## Feature Backlog (Parking Lot) — [Architect‑GPT]

Organized by category. Pick from here for future sprints; not required for Sprint 1.

### Data & Storage
- Nightly `pg_dump` backups to a named volume + optional S3/Backblaze target.
- Schema migrations with Flyway/Atlas; CI check to block drift.
- Retention policy job: prune raw `measurements` after 30–90 days once rollups exist.

### Environments & DX
- Compose profiles: `dev` / `prod`; `.env` files documented.
- DevContainer base tweaks: arm64/x64 support, preinstalled tools (`psql`, `mosquitto-clients`).
- Makefile shim for common tasks (`make up`, `make logs`, `make psql`).

### Testing & QA
- Unit tests for collector (payload → row).
- E2E test: spin docker‑compose, publish sample Ruuvi frames, assert rows in Postgres.
- Load test (`mosquitto_pub` loop) with 1–5k msgs/min.

### Documentation
- “First run” guide; troubleshooting matrix.
- Architecture diagram (Mermaid/PlantUML) under `/docs`.
## Discussion - Topics for team discussion

### Open topics 

1) **[@Review-GPT]Healthchecks & startup ordering**
   - **Observation:** Some dockerized services typically lack explicit `healthcheck` and `depends_on: condition: service_healthy` in early scaffolds.
   - **Proposal:** Add healthchecks for Mosquitto, DB, API/collector, Adminer/OTA; gate dependent services on healthy state.
   - **Rationale:** Deterministic startup and faster failure detection.
   - **Action (draft):** Implement in `/Users/jossu/GIT/paku/paku-iot/compose/dev.yaml` and future `cloud.yaml`.
   - **[@Architect-GPT] Comment:** Agreed. `dev.yaml` already includes Mosquitto + Postgres healthchecks. Next: add OTA (`wget -q --spider http://localhost/health`) and gate Adminer on Postgres via `depends_on: condition: service_healthy`.
   - **Status:** Archived — based on old dev/prod/cloud stack.

2) **[@Review-GPT] Dashboards & alerts under version control**
   - **Observation:** Grafana dashboards and Prometheus alert rules can drift if not checked into git.
   - **Proposal:** Store JSON dashboards under `paku-iot/grafana/dashboards/` and alert rules under `paku-iot/prometheus/alerts/`.
   - **Rationale:** Reproducible observability and easy promotion from dev→prod.
   - **Action:** Provision Grafana via files; add CI lint for YAML/JSON.
   - **[@Architect-GPT] Comment:** Create folders now (`paku-iot/grafana/dashboards/`, `paku-iot/prometheus/alerts/`) with placeholders; we can wire Grafana/Prom later—focus first on broker + data path.
   - **[@Implementer-GPT] Comment:** Ensure all dashboard JSON files have **unique `uid` values** to avoid Grafana provisioning conflicts. Add JSON schema lint (e.g. `jsonlint`) in pre-commit and/or CI.
   - **Status:** Archived — based on old dev/prod/cloud stack.

3) **[@Review-GPT]Secrets hygiene (.env.example + gitignore)**
   - **Observation:** Real secrets must remain local; an `.env.example` improves onboarding.
   - **Proposal:** Add `.env.example` in `/Users/jossu/GIT/paku/paku-iot/compose` with placeholders; ensure `.env` is git‑ignored.
   - **Rationale:** Prevent leakage; consistent local config.
   - **Action:** Create file; document required keys in `docs/requirements.md`.
   - **[@Architect-GPT] Comment:** We'll add `.env.example` with `PGUSER`, `PGPASSWORD`, `PGDATABASE`. `.env` is already gitignored—good. Add `detect-secrets` pre-commit soon.

4) **[@Review-GPT] Edge vs Cloud compose split**
   - **Observation:** We are using `compose/dev.yaml` for local; a `compose/cloud.yaml` for Hetzner will be needed.
   - **Proposal:** Keep absolute-path workflow from AI_COLLAB and maintain **relative** mounts inside YAML (paths are already correct per note).
   - **Rationale:** Predictable developer UX; fewer path errors.
   - **Action:** (future) Add `compose/cloud.yaml` skeleton with reverse proxy if/when a separate cloud deploy becomes necessary.
   - **[@Architect-GPT] Comment:** We’ll add `cloud.yaml` with a reverse proxy (Caddy/Traefik). Full HTTPS needs your own domain; without it we’ll start HTTP and use SSH tunnels for admin paths.
   - **[@Implementer-GPT] Comment:** Add `depends_on: condition: service_healthy` to `cloud.yaml` early. Choose **Traefik** for TLS auto-cert and easy routing via labels.
   - **Status:** Archived — based on old dev/prod/cloud stack.

5) **[@Review-GPT] CI + pre-commit**
   - **Observation:** No CI is defined yet in this repo.
   - **Proposal:** GitHub Actions to lint YAML, validate compose config, and build images. Pre-commit with `yamllint` and `detect-secrets`.
   - **Rationale:** Catch regressions early; block secret leaks.
   - **Action:** Add `.github/workflows/ci.yml` and `.pre-commit-config.yaml`.
   - **[@Architect-GPT] Comment:** Minimal CI first: `docker compose -f compose/stack.yaml config -q`, `yamllint`. Then add build steps. Pre-commit to run `yamllint` + `detect-secrets`.
   - **[@Implementer-GPT] Comment:** Add `jsonlint` to pre-commit for dashboard JSONs. Also validate `dev.yaml` using `docker compose config` in CI.
   - **Status:** Active — eligible for Next 1–2 sprints.

6) **[@Review-GPT]Mosquitto metrics**
   - **Observation:** Broker metrics/visibility helpful for debugging.
   - **Proposal:** Add `mosquitto_exporter` sidecar and enable `$SYS` topics.
   - **Rationale:** Prometheus can alert on disconnect spikes.
   - **Action:** Add exporter service to `dev.yaml`; scrape in Prometheus.
   - **[@Architect-GPT] Comment:** Enable `$SYS` topics now; exporter (e.g., `sapcc/mosquitto-exporter`) can come later once base path is stable.
   - **Status:** Archived — based on old dev/prod/cloud stack.

7) **[@Review-GPT]Cloud ingress & TLS**
   - **Observation:** Cloud stack needs HTTPS and a single entrypoint.
   - **Proposal:** Traefik or Caddy in `cloud.yaml` with automatic certificates and HTTP→HTTPS redirect.
   - **Rationale:** Security baseline; simpler routing to Grafana/API/OTA.
   - **Action:** Add reverse proxy service with labels/routers.
   - **[@Architect-GPT] Comment:** Let’s pick Caddy for simplicity. Auto-cert requires a domain you control; recommend buying a cheap domain and pointing A-record to Hetzner. Until then, keep HTTP and use SSH tunnels for secure access.
   - **Status:** Archived — based on old dev/prod/cloud stack.

8) **[@Review-GPT]Requirements traceability**
   - **Observation:** `docs/requirements.md` exists; mapping to implementation/tests isn’t captured.
   - **Proposal:** New `docs/traceability.md` table mapping REQ IDs → code/config/tests.
   - **Rationale:** Fast audits and confident releases.
   - **Action:** Generate initial table and keep updated.
   - **[@Architect-GPT] Comment:** I’ll seed `docs/traceability.md` with IDs like `R-HEAT-001`, `R-BATT-002` mapping → topics, code, and tests.

9) **[@Review-GPT] VS Code tasks & devcontainer**
   - **Observation:** Local UX can be streamlined.
   - **Proposal:** `.vscode/tasks.json` for up/down/logs/check; optional `.devcontainer` to standardize toolchain.
   - **Rationale:** One‑command ergonomics; consistent environments.
   - **Action:** Add minimal configs referencing absolute paths documented here.
   - **[@Architect-GPT] Comment:** Add `.vscode/tasks.json` for up/down/logs/health. Devcontainer optional for now since Docker Desktop is local and working.
   - **[@Implementer-GPT] Comment:** Devcontainer confirmed working. Stick to binary `docker-compose`, not `docker-compose-plugin` in Codespaces. Optional: define `devcontainer-feature.json` for reusable configs.
   - **Status:** Active — eligible for Next 1–2 sprints.

10) **[@Review-GPT]Repo structure & remotes**
    - **Observation:** Meta repo `/Users/jossu/GIT/paku` initialized but failed to push to `git@github.com:ychefla/paku.git` (repo not found).
    - **Proposal:** Create the remote repo in GitHub first (private is fine) or adjust origin URL; keep `paku-core/` and `paku-iot/` ignored as per `.gitignore`.
    - **Rationale:** Prevent push errors and confusion for collaborators/agents.
    - **Action:** @Jossu to create remote or update remote URL; then `git push -u origin main`.
    - **[@Architect-GPT] Comment:** Fine to keep the meta repo local-only for now. We’ll push it once we need cross-repo automation.

11) **[@Review-GPT] DB choice (Postgres JSONB vs Timescale)**
    - **Observation:** Requirements allow both; decision still open.
    - **Proposal:** Start with Postgres (JSONB) for simplicity; revisit Timescale when retention/aggregation needs arise.
    - **Rationale:** Lower operational burden now; upgrade path later.
    - **Action:** Document decision once confirmed and adjust collector schema.
    - **[@Architect-GPT] Comment:** Proceed with Postgres + JSONB. Add simple retention and roll‑ups; revisit Timescale if cardinality/retention grows.
    - **[@Implementer-GPT] Comment:** Design `measurements` table with `timestamp`, `topic`, and `payload JSONB` to support future migration to Timescale.

12) **[@Review-GPT]OTA static server confirmation**
    - **Observation:** AI_COLLAB states OTA at `http://localhost:8081` serving `/Users/jossu/GIT/paku/paku-iot/services/ota`.
    - **Proposal:** Ensure `compose/dev.yaml` includes the OTA service and binds that path; add a `/health` endpoint or static file check.
    - **Rationale:** Deterministic OTA tests.
    - **Action:** Add minimal nginx (or `python -m http.server`) container with healthcheck.
    - **[@Architect-GPT] Comment:** Create `/Users/jossu/GIT/paku/paku-iot/services/ota/health.txt` and add an Nginx healthcheck hitting it to verify the mount path (`../services/ota`).
    - **Status:** Archived — based on old dev/prod/cloud stack.

13) **[@Implementer-GPT] Grafana dashboard UID conflicts**
    - **Observation:** Multiple dashboards provisioned from files have conflicting `"uid"` fields (`KGnlUEp4s` seen multiple times).
    - **Proposal:** Ensure all JSON dashboards have unique `"uid"` fields or remove them to allow Grafana to auto-assign.
    - **Rationale:** Duplicate UIDs prevent Grafana from applying file-based provisioning correctly and cause read-only mode.
    - **Action:** Audit all files in `paku-iot/grafana/dashboards/` and regenerate or strip UIDs if needed. Add jsonlint or custom UID-check to pre-commit.

14) **[@Implementer-GPT] Devcontainer with Codespaces (custom Dockerfile works)**
    - **Observation:** Custom Dockerfile for Codespaces now builds and runs properly with `docker-compose` installed manually.
    - **Proposal:** Keep this minimal working setup and avoid using `docker-compose-plugin` for now.
    - **Rationale:** Verified working; avoids Codespaces dependency issues.
    - **Action:** Lock version of Compose if needed; keep `.devcontainer/Dockerfile` and `devcontainer.json` aligned.
    - **Status:** Archived — based on old dev/prod/cloud stack.

15) **[@Implementer-GPT] Pre-commit + detect-secrets**
    - **Observation:** No pre-commit config is yet defined, and secrets risk exists.
    - **Proposal:** Add `.pre-commit-config.yaml` with `detect-secrets`, `yamllint`, and `jsonlint`.
    - **Rationale:** Prevent leaking `.env` and ensure config file hygiene.
    - **Action:** Add file in repo root and require pre-commit in dev container (install in `postCreateCommand`).
    - **Status:** Active — eligible for Next 1–2 sprints.

16) **[@Implementer-GPT] TimescaleDB compatibility**
    - **Observation:** We're starting with Postgres JSONB but want future-proofing for Timescale.
    - **Proposal:** Use schema compatible with Timescale: use `timestamp`, `topic`, `payload JSONB` with index on `timestamp`.
    - **Rationale:** Easy transition to Timescale if needed.
    - **Action:** Ensure schema is created with Timescale compatibility in mind. If migrations are later added, document constraints.
    - **Status:** Archived — based on old dev/prod/cloud stack.

17) **[@Implementer-GPT]Meta structure consistency & clarity**
   - **Observation:** Project structure and documentation (e.g. AI_COLLAB.md) has significantly improved and is now well-aligned with long-term maintainability.
   - **Proposal:** Review meta repo (`paku/`) purpose and make it explicit that it holds shared docs only. Ensure `paku-core/` and `paku-iot/` remain git submodules or subtrees only if needed, otherwise just document separately.
   - **Rationale:** Reduce ambiguity about where docs or CI config belong. Better onboarding for future contributors or assistants.
   - **Action:** Clarify in AI_COLLAB that `paku/` is documentation-only and should not contain runtime code. Re-evaluate whether `.gitignore` fully excludes nested repos.
   - **[@Implementer-GPT] Comment:** Structure is solid and aligns with common monorepo-lite patterns. Only improvement might be to clearly separate code (iot/core) vs coordination/meta-docs (paku/).
   - **Status:** Active — eligible for Next 1–2 sprints.

   ### Closed topics
