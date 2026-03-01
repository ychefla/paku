# Paku Project Backlog

Consolidated TODO / future-work items across all repositories.
Review periodically and promote items to sprint scope as needed.

---

## 🔒 Security

- [x] MQTT authentication (username/password) for production
- [x] TLS encryption for MQTT broker (port 8883, self-signed CA)
- [ ] MQTT topic ACLs (per-device access control)
- [ ] OTA firmware digital signature verification
- [ ] Replace `setInsecure()` with CA cert for HTTPS OTA
- [ ] Pre-commit `detect-secrets` hook
- [x] `.env.example` creation + secrets hygiene
- [x] Grafana multi-user (admin + viewer accounts)
- [ ] Caddy security headers + rate limiting

## 📡 OTA

- [ ] Non-blocking OTA (currently blocks main loop 1–5 min)
- [ ] Resumable downloads (HTTP Range requests)
- [ ] Delta/differential updates
- [ ] OTA bundle signing + version metadata
- [ ] OTA alert conditions (failure rate, device offline post-update)

## 🏗️ Infrastructure / DevOps

- [ ] CI: `yamllint`, `jsonlint`, compose config validation
- [x] Reverse proxy (Caddy) + automatic TLS + domain
- [x] Docker healthchecks + `depends_on: service_healthy`
- [ ] Makefile shim (`make up`, `make logs`, `make psql`)

## 🧪 Testing / QA

- [ ] E2E test: spin stack → publish MQTT → verify Postgres row
- [x] Unit tests for collector (topic parsing + payload validation)
- [ ] Load test (`mosquitto_pub` loop, 1–5 k msgs/min)
- [x] MQTT payload validation in collector (reject malformed)

## 🗄️ Data / Storage

- [x] Daily `pg_dump` backups (7-day retention, named volume)
- [ ] Schema migrations (Flyway/Atlas) with CI drift check
- [x] Grafana dashboard queries: use `$__interval` aggregation (not raw rows)
- [x] EcoFlow temp fields extracted to dedicated columns (`inv_out_temp`, `bms_temp`, etc.)
- [x] Expression indexes on `ecoflow_measurements` for JSONB temp fields (interim)

### EcoFlow schema cleanup (Vaihe 2)
- [ ] **Poista `raw_data` JSONB-sarake** `ecoflow_measurements`-taulusta
  - Collector kirjoittaa enää vain purettuihin sarakkeisiin
  - Grafana-kyselyt eivät tarvitse enää `raw_data->>'...'` -syntaksia
  - Taulun koko putoaa ~783 MB → ~20 MB
  - Muista: kaikki EcoFlow Grafana-dashboardit päivitettävä samalla

### EcoFlow config-taulu eriyttäminen (Vaihe 2b, optional)
- [ ] Jaa `ecoflow_measurements` kahteen tauluun:
  - `ecoflow_telemetry` – mittaustiedot (soc, watts, lämpötilat) – kirjoitetaan 30s välein
  - `ecoflow_config` – asetukset (ems_min_soc, lcd_brightness, sys_ver jne.) – upsert vain muuttuessa
  - Malli: `INSERT ... ON CONFLICT (device_sn) DO UPDATE SET ... WHERE changed`

### TimescaleDB (Vaihe 3)
- [ ] **Vaihda Postgres-image** `timescale/timescaledb:latest-pg16`:ksi `stack/postgres/Dockerfile`-ssa
- [ ] **Luo hypertablet** tyhjille/uusille tauluille:
  ```sql
  CREATE EXTENSION IF NOT EXISTS timescaledb;
  SELECT create_hypertable('ecoflow_telemetry', 'ts');
  SELECT create_hypertable('measurements', 'ts');   -- vaatii migraation
  ```
- [ ] **`measurements`-migraatio** (olemassa olevalla datalla):
  ```sql
  -- ~1 min downtime:
  ALTER TABLE measurements RENAME TO measurements_old;
  CREATE TABLE measurements ( ...same schema... );
  SELECT create_hypertable('measurements', 'ts', chunk_time_interval => INTERVAL '7 days');
  INSERT INTO measurements SELECT * FROM measurements_old;
  DROP TABLE measurements_old;
  ```
- [ ] **Continuous Aggregates** temperature + humidity paneeleille:
  ```sql
  CREATE MATERIALIZED VIEW measurements_5min
  WITH (timescaledb.continuous) AS
  SELECT time_bucket('5 min', ts) AS bucket,
         site_id, device_id, location,
         AVG((metrics->>'temperature_c')::numeric) AS avg_temp,
         AVG((metrics->>'humidity_percent')::numeric) AS avg_humidity
  FROM measurements
  GROUP BY 1, site_id, device_id, location;
  ```
- [ ] **Data retention policies**:
  ```sql
  -- Raakadata: säilytä 30 pv
  SELECT add_retention_policy('measurements', INTERVAL '30 days');
  -- Continuous aggregate: säilytä 2 v
  SELECT add_retention_policy('measurements_5min', INTERVAL '2 years');
  ```
- [ ] Grafana-kyselyt osoittamaan `measurements_5min`-näkymään pitkillä aikaikkunoilla
  (tai Grafana + TimescaleDB datasource-plugin automaattinen continuous aggregate -reititys)
- [ ] **Huomio**: TimescaleDB:n asennus olemassa olevaan Postgres-konttiin vaatii datadumpin ennen image-vaihtoa

### Downsampling (Vaihe 2 väliaikainen, jos TimescaleDB siirtyy)
- [ ] Retention policy scripti: harvennetaan yli 30 pv vanha `measurements`-data 5 min aggregaateiksi
  - pg_cron tai erillinen cron-kontti
  - Säilytä alkuperäinen data 7 pv, aggregoitu ikuisesti

- [ ] Retention policy: prune raw measurements after 30–90 days
- [ ] Rollup / aggregation jobs
- [x] Device registry (devices table, auto-populated from edge status)
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
