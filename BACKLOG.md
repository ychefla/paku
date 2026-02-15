# Paku Project Backlog

Consolidated TODO / future-work items across all repositories.
Review periodically and promote items to sprint scope as needed.

---

## 🔒 Security

- [ ] MQTT authentication (username/password) for production
- [ ] TLS encryption for MQTT broker
- [ ] MQTT topic ACLs (per-device access control)
- [ ] OTA firmware digital signature verification
- [ ] Replace `setInsecure()` with CA cert for HTTPS OTA
- [ ] Pre-commit `detect-secrets` hook
- [ ] `.env.example` creation + secrets hygiene

## 📡 OTA

- [ ] Non-blocking OTA (currently blocks main loop 1–5 min)
- [ ] Resumable downloads (HTTP Range requests)
- [ ] Delta/differential updates
- [ ] OTA bundle signing + version metadata
- [ ] OTA alert conditions (failure rate, device offline post-update)

## 🏗️ Infrastructure / DevOps

- [ ] CI: `yamllint`, `jsonlint`, compose config validation
- [ ] Reverse proxy (Caddy/Traefik) + TLS + domain
- [ ] Docker healthchecks + `depends_on: service_healthy`
- [ ] Makefile shim (`make up`, `make logs`, `make psql`)

## 🧪 Testing / QA

- [ ] E2E test: spin stack → publish MQTT → verify Postgres row
- [ ] Unit tests for collector (payload → DB row)
- [ ] Load test (`mosquitto_pub` loop, 1–5 k msgs/min)
- [ ] MQTT payload validation in collector (reject malformed)

## 🗄️ Data / Storage

- [ ] Nightly `pg_dump` backups (named volume + optional S3)
- [ ] Schema migrations (Flyway/Atlas) with CI drift check
- [ ] Retention policy: prune raw measurements after 30–90 days
- [ ] Rollup / aggregation jobs
- [ ] Device registry (device_id metadata table)
- [ ] Dead-letter queue for unparseable messages

## 🔥 Hydronic Heater

- [ ] Motorized zone valves (floor, air, water)
- [ ] PID temperature control per zone
- [ ] Additional DS18B20 sensors (coolant, floor supply/return)
- [ ] Heat exchanger fan control (PWM via MOSFET)
- [ ] Power profiles: Eco / Normal / Boost presets
- [ ] Scheduled heating (day-of-week, 4 slots, NVS persistence)
- [ ] Outside temperature source for smart pre-heating
- [ ] Per-zone safety limits (floor max 50 °C, water max 85 °C)
- [ ] Heater UART protocol: re-enable when full spec available

## 🔌 Sensors / Hardware (EDGE)

- [ ] Flow sensor: re-enable when hardware installed + calibrated
- [ ] Runtime configuration via MQTT or NVS
- [ ] MoKo: motion detection alerts, battery low notifications
- [ ] Deep sleep power management cycle

## 📊 Monitoring / Observability

- [ ] Loki + Promtail centralized logging
- [ ] Mosquitto exporter + `$SYS` metrics
- [ ] Telegram alerts for threshold breaches
- [ ] Grafana alerts: battery low, power output, data loss

## 🖥️ UI / Display

- [ ] Unified Grafana dashboard as default home
- [ ] Mobile-optimized Grafana views
- [ ] EcoFlow dashboard: alerts, weather integration

## 📖 Documentation

- [ ] "First run" quickstart + troubleshooting matrix
- [ ] Architecture diagram (Mermaid) in `/docs`
