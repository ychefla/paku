# AI_COLLAB – Shared context for assistants

## Prerequisites
To ensure the project works correctly, all repositories must be cloned into the same parent directory. The expected structure is as follows:


## MQTT Broker Configuration

## TODO: Secure the MQTT Broker for Production
- **Task**: Add authentication and TLS encryption to the Mosquitto broker in production.
- **Steps**:
  1. Generate or obtain the following files:
     - `passwords`: Use the `mosquitto_passwd` command to create a password file.
     - `ca.crt`, `server.crt`, `server.key`: Obtain or generate TLS certificates for encrypting communication.
  2. Update the `prod.yaml` file to include the following volumes:
     ```yaml
     volumes:
       - ./mosquitto/config/passwords:/mosquitto/config/passwords
       - ./mosquitto/config/ca.crt:/mosquitto/config/ca.crt
       - ./mosquitto/config/server.crt:/mosquitto/config/server.crt
       - ./mosquitto/config/server.key:/mosquitto/config/server.key
     ```
  3. Update the `mosquitto.prod.conf` file to enable authentication and TLS:
     ```conf
     allow_anonymous false
     password_file /mosquitto/config/passwords
     listener 8883
     cafile /mosquitto/config/ca.crt
     certfile /mosquitto/config/server.crt
     keyfile /mosquitto/config/server.key
     ```
  4. Test the configuration in a staging environment before deploying to production.

### Development Configuration
- **File**: `$PROJECT_ROOT/paku-iot/compose/mosquitto/mosquitto.conf`
- **Purpose**: Configures the Mosquitto MQTT broker for development purposes.
- **Example Configuration**:
  ```conf
  # Listen on port 1883 for MQTT connections
  listener 1883
  allow_anonymous true

  # Persistence settings
  persistence true
  persistence_location /mosquitto/data/

  # Log settings
  log_dest stdout
  ```
- **Notes**:
  - The above configuration allows anonymous connections, which is acceptable for local development but **must not** be used in production.
  - The persistence settings ensure that messages are retained across broker restarts.

### Production Configuration
- **File**: `$PROJECT_ROOT/paku-iot/compose/mosquitto/mosquitto.prod.conf` (to be created if not available).
- **Purpose**: Configures the Mosquitto MQTT broker for production use with enhanced security.
- **How to create**:
  1. Copy the development configuration as a starting point:
     ```bash
     cp $PROJECT_ROOT/paku-iot/compose/mosquitto/mosquitto.conf $PROJECT_ROOT/paku-iot/compose/mosquitto/mosquitto.prod.conf
     ```
  2. Modify the `mosquitto.prod.conf` file to include production-specific settings:
     - **Disable anonymous access**:
       ```conf
       allow_anonymous false
       ```
     - **Enable authentication**:
       ```conf
       password_file /mosquitto/config/passwords
       ```
       Create the password file:
       ```bash
       mosquitto_passwd -c $PROJECT_ROOT/paku-iot/compose/mosquitto/config/passwords <username>
       ```
     - **Restrict listener access**:
       ```conf
       listener 1883 127.0.0.1
       ```
     - **Enable TLS encryption** (optional but recommended):
       ```conf
       listener 8883
       cafile /mosquitto/config/ca.crt
       certfile /mosquitto/config/server.crt
       keyfile /mosquitto/config/server.key
       ```
  3. Update the Docker Compose configuration to use the production configuration file:
     ```yaml
     services:
       mosquitto:
         volumes:
           - ./mosquitto/mosquitto.prod.conf:/mosquitto/config/mosquitto.conf
     ```

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
- Bring up the dev stack:
  ```bash
  cd $PROJECT_ROOT/paku-iot/compose
  docker compose -f dev.yaml up -d
  ```

- Mosquitto config used by dev:
  - `$PROJECT_ROOT/paku-iot/compose/mosquitto/mosquitto.conf`

> **Note:** Paths inside `dev.yaml` are **relative to the compose dir** and still correct (`../data`, `../services`). No change needed there.

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

## Docker Compose Configuration

### Development Configuration
- **File**: `$PROJECT_ROOT/paku-iot/compose/dev.yaml`
- **Purpose**: Used to bring up the development stack with services like Mosquitto, Postgres, Adminer, and the OTA static server.
- **How to use**:
  ```bash
  cd $PROJECT_ROOT/paku-iot/compose
  docker compose -f dev.yaml up -d
  ```
- **Notes**:
  - The `dev.yaml` file uses relative paths (e.g., `../data`, `../services`) to ensure portability.
  - This configuration is optimized for local development and testing.

### Production Configuration
- **File**: `$PROJECT_ROOT/paku-iot/compose/prod.yaml` (to be created if not available).
- **Purpose**: Used to deploy the stack in a production environment with stricter security and performance optimizations.
- **How to create**:
  1. Copy the `dev.yaml` file as a starting point:
     ```bash
     cp $PROJECT_ROOT/paku-iot/compose/dev.yaml $PROJECT_ROOT/paku-iot/compose/prod.yaml
     ```
  2. Modify the `prod.yaml` file to include production-specific settings:
     - Use production-grade images (e.g., remove debugging tools).
     - Configure persistent storage for databases.
     - Use secrets management tools (e.g., Docker Secrets) for sensitive data.
     - Restrict service access to specific IPs or networks.
  3. Bring up the production stack:
     ```bash
     cd $PROJECT_ROOT/paku-iot/compose
     docker compose -f prod.yaml up -d
     ```

### Best Practices for Docker Compose
1. **Separate Configurations**:
   - Always maintain separate `dev.yaml` and `prod.yaml` files to avoid accidental misuse.
2. **Environment Variables**:
   - Use `.env` files to manage environment-specific variables for both development and production.
3. **Health Checks**:
   - Add health checks to critical services (e.g., Mosquitto, Postgres) to ensure they are running properly.
4. **Networking**:
   - In production, restrict service access to trusted networks and use firewalls to block unauthorized access.
5. **Monitoring**:
   - Integrate monitoring tools (e.g., Prometheus, Grafana) to track the health and performance of services.

> **Important**: Never use the `dev.yaml` file in production, as it may lack the necessary security and performance optimizations.

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
6) Create Grafana dashboard file: `/Users/jossu/GIT/paku/paku-iot/grafana/dashboards/sprint1_measurements.json` (single panel querying `measurements`).
7) Add `.env.example` to `/Users/jossu/GIT/paku/paku-iot/compose` with `PGUSER`, `PGPASSWORD`, `PGDATABASE` placeholders.

## How assistants should use this file
- **Always read/summarize this file first** before proposing changes.
- Propose commands with **full absolute paths** as above.
- Keep changes minimal and aligned with decisions here.
- Topics that have not yet been agreed on, but require discussion should be categorized and discussed under "Discussion - open topics". This is where improvement suggestions, found problems, etc. are added and discussed befor making decision. Once the decision is made and approved by human the topic is moved to decisions and removed from discussion. Everyone should tag themselves so that it is clear who made the comment or raised the topic. Human is called Jossu and uses @Jossu.

### Sprint 1 — Hello Data Path (source of truth)

**Goal:** Prove end-to-end telemetry: Edge → MQTT → Collector → Postgres (+ OTA alive).

**Scope**
- Local stack up via `compose/dev.yaml`: Mosquitto :1883, Postgres+Adminer :8080, OTA :8081 (healthchecks green).
- Edge stub (paku-core) publishes heartbeat + one sample metric to `paku/<device>/tele/...`.
- Minimal collector writes to Postgres table `measurements (ts timestamptz default now(), topic text, payload jsonb)`.
- Secrets hygiene: `.env.example` (placeholders); `.env` & `secrets.h` ignored.
- CI lint: `docker compose -f compose/dev.yaml config -q` + `yamllint`.
- Grafana panel (**mandatory**): **dashboard JSON lives at** `/Users/jossu/GIT/paku/paku-iot/grafana/dashboards/sprint1_measurements.json` (single panel visualizing data from `measurements` in Postgres).
- OTA healthcheck file present at `/Users/jossu/GIT/paku/paku-iot/services/ota/health.txt` and referenced by container healthcheck.

**Done criteria**
- `mosquitto_pub` test appears in DB.
- Edge heartbeat appears in DB.
- `http://localhost:8081/health.txt` returns 200.
- Adminer reachable at `http://localhost:8080`.
- Grafana dashboard JSON exists at `/Users/jossu/GIT/paku/paku-iot/grafana/dashboards/sprint1_measurements.json` and, when imported into Grafana, shows data from table `measurements`.

**Ownership**
- Owner: @Jossu  
- Assistants: @Architect-GPT, @Review-GPT, @Implementer-GPT  
- Tasks mirrored under “Open tasks”.

**Sprint 1 picks from “Discussion – open topics”**
- #1 Healthchecks & startup ordering → Ensure OTA has a healthcheck and Adminer waits on Postgres (`depends_on: condition: service_healthy`).
- #2 Dashboards under version control → Create the single dashboard JSON at the path above; alerts deferred.
- #3 Secrets hygiene → Add `.env.example` in `/Users/jossu/GIT/paku/paku-iot/compose` with placeholders; keep `.env` in `.gitignore`.
- #5 CI (minimal) → Add GitHub Actions to run `docker compose -f compose/dev.yaml config -q` and `yamllint`.
- #9 VS Code ergonomics → Add minimal `.vscode/tasks.json` to start/stop stack and check health.
- #12 OTA static server → Include `/health.txt` and container healthcheck in `dev.yaml`.

## Feature Backlog (Parking Lot) — [Architect‑GPT]

Organized by category. Pick from here for future sprints; not required for Sprint 1.

### Observability & Ops
- Alerting rules in Grafana for sensor silence and thresholds (email/Telegram webhook).
- Centralized logs with Loki + Promtail (collector, edge-sim, mosquitto).
- Grafana annotations emitted by collector on deploy/edge restart.

### Data & Storage
- Nightly `pg_dump` backups to a named volume + optional S3/Backblaze target.
- Schema migrations with Flyway/Atlas; CI check to block drift.
- Retention policy job: prune raw `measurements` after 30–90 days once rollups exist.

### Security & Secrets
- `age` + `sops` for secrets at rest; add `.sops.yaml` policy (optionally use KMS).
- Least‑privileged DB user for Grafana (read‑only).
- Mosquitto auth: per‑device user, ACL templates.

### Edge / Device
- Device registry (device_id, label, owner, location, tags).
- OTA bundle signing; version check endpoint; progressive rollout rings.

### Collector & Pipelines
- JSON schema validation; reject malformed payloads with reason.
- Enrich messages with device metadata; write to `measurements_enriched`.
- Dead‑letter queue (DLQ) MQTT topic for parse failures.

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

### Performance & Reliability
- Index tuning on `measurements(ts, topic)`.
- Monthly partitioning and maintenance jobs (VACUUM/ANALYZE).
## Discussion - Topics for team discussion

### Open topics 

1) **[@Review-GPT]Healthchecks & startup ordering**
   - **Observation:** Some dockerized services typically lack explicit `healthcheck` and `depends_on: condition: service_healthy` in early scaffolds.
   - **Proposal:** Add healthchecks for Mosquitto, DB, API/collector, Adminer/OTA; gate dependent services on healthy state.
   - **Rationale:** Deterministic startup and faster failure detection.
   - **Action (draft):** Implement in `/Users/jossu/GIT/paku/paku-iot/compose/dev.yaml` and future `cloud.yaml`.
   - **[@Architect-GPT] Comment:** Agreed. `dev.yaml` already includes Mosquitto + Postgres healthchecks. Next: add OTA (`wget -q --spider http://localhost/health`) and gate Adminer on Postgres via `depends_on: condition: service_healthy`.

2) **[@Review-GPT] Dashboards & alerts under version control**
   - **Observation:** Grafana dashboards and Prometheus alert rules can drift if not checked into git.
   - **Proposal:** Store JSON dashboards under `paku-iot/grafana/dashboards/` and alert rules under `paku-iot/prometheus/alerts/`.
   - **Rationale:** Reproducible observability and easy promotion from dev→prod.
   - **Action:** Provision Grafana via files; add CI lint for YAML/JSON.
   - **[@Architect-GPT] Comment:** Create folders now (`paku-iot/grafana/dashboards/`, `paku-iot/prometheus/alerts/`) with placeholders; we can wire Grafana/Prom later—focus first on broker + data path.
   - **[@Implementer-GPT] Comment:** Ensure all dashboard JSON files have **unique `uid` values** to avoid Grafana provisioning conflicts. Add JSON schema lint (e.g. `jsonlint`) in pre-commit and/or CI.

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
   - **Action:** Add `compose/cloud.yaml` skeleton with reverse proxy.
   - **[@Architect-GPT] Comment:** We’ll add `cloud.yaml` with a reverse proxy (Caddy/Traefik). Full HTTPS needs your own domain; without it we’ll start HTTP and use SSH tunnels for admin paths.
   - **[@Implementer-GPT] Comment:** Add `depends_on: condition: service_healthy` to `cloud.yaml` early. Choose **Traefik** for TLS auto-cert and easy routing via labels.

5) **[@Review-GPT] CI + pre-commit**
   - **Observation:** No CI is defined yet in this repo.
   - **Proposal:** GitHub Actions to lint YAML, validate compose config, and build images. Pre-commit with `yamllint` and `detect-secrets`.
   - **Rationale:** Catch regressions early; block secret leaks.
   - **Action:** Add `.github/workflows/ci.yml` and `.pre-commit-config.yaml`.
   - **[@Architect-GPT] Comment:** Minimal CI first: `docker compose -f compose/dev.yaml config -q`, `yamllint`. Then add build steps. Pre-commit to run `yamllint` + `detect-secrets`.
   - **[@Implementer-GPT] Comment:** Add `jsonlint` to pre-commit for dashboard JSONs. Also validate `dev.yaml` using `docker compose config` in CI.

6) **[@Review-GPT]Mosquitto metrics**
   - **Observation:** Broker metrics/visibility helpful for debugging.
   - **Proposal:** Add `mosquitto_exporter` sidecar and enable `$SYS` topics.
   - **Rationale:** Prometheus can alert on disconnect spikes.
   - **Action:** Add exporter service to `dev.yaml`; scrape in Prometheus.
   - **[@Architect-GPT] Comment:** Enable `$SYS` topics now; exporter (e.g., `sapcc/mosquitto-exporter`) can come later once base path is stable.

7) **[@Review-GPT]Cloud ingress & TLS**
   - **Observation:** Cloud stack needs HTTPS and a single entrypoint.
   - **Proposal:** Traefik or Caddy in `cloud.yaml` with automatic certificates and HTTP→HTTPS redirect.
   - **Rationale:** Security baseline; simpler routing to Grafana/API/OTA.
   - **Action:** Add reverse proxy service with labels/routers.
   - **[@Architect-GPT] Comment:** Let’s pick Caddy for simplicity. Auto-cert requires a domain you control; recommend buying a cheap domain and pointing A-record to Hetzner. Until then, keep HTTP and use SSH tunnels for secure access.

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

15) **[@Implementer-GPT] Pre-commit + detect-secrets**
    - **Observation:** No pre-commit config is yet defined, and secrets risk exists.
    - **Proposal:** Add `.pre-commit-config.yaml` with `detect-secrets`, `yamllint`, and `jsonlint`.
    - **Rationale:** Prevent leaking `.env` and ensure config file hygiene.
    - **Action:** Add file in repo root and require pre-commit in dev container (install in `postCreateCommand`).

16) **[@Implementer-GPT] TimescaleDB compatibility**
    - **Observation:** We're starting with Postgres JSONB but want future-proofing for Timescale.
    - **Proposal:** Use schema compatible with Timescale: use `timestamp`, `topic`, `payload JSONB` with index on `timestamp`.
    - **Rationale:** Easy transition to Timescale if needed.
    - **Action:** Ensure schema is created with Timescale compatibility in mind. If migrations are later added, document constraints.

17) **[@Implementer-GPT]Meta structure consistency & clarity**
   - **Observation:** Project structure and documentation (e.g. AI_COLLAB.md) has significantly improved and is now well-aligned with long-term maintainability.
   - **Proposal:** Review meta repo (`paku/`) purpose and make it explicit that it holds shared docs only. Ensure `paku-core/` and `paku-iot/` remain git submodules or subtrees only if needed, otherwise just document separately.
   - **Rationale:** Reduce ambiguity about where docs or CI config belong. Better onboarding for future contributors or assistants.
   - **Action:** Clarify in AI_COLLAB that `paku/` is documentation-only and should not contain runtime code. Re-evaluate whether `.gitignore` fully excludes nested repos.
   - **[@Implementer-GPT] Comment:** Structure is solid and aligns with common monorepo-lite patterns. Only improvement might be to clearly separate code (iot/core) vs coordination/meta-docs (paku/).

   ### Closed topics
