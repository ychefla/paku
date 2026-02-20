# Copilot Instructions for Paku (monorepo root)

These instructions apply to all Paku sub-projects (paku-iot, paku-core, paku-ha, hydronic-heater).

## Communication Style

- **Never present assumptions, guesses, or generated logic as verified facts.**
- If something is uncertain, say so explicitly using phrases like "I assume", "I'm not sure", "this is my best guess", or "I don't have a verified source for this".
- When generating protocol implementations, hardware interfaces, or integration code, always state whether the implementation is based on verified documentation or inference.
- Prefer "I don't know" over a confident-sounding but unverified answer.

## Project Structure

| Repository | Purpose |
|---|---|
| `paku-core` | ESP32 edge firmware (LilyGo T-Display S3) |
| `paku-iot` | Backend services (collector, Grafana, Postgres, MQTT) |
| `paku-ha` | Home Assistant integration (dashboards, automations, MQTT sensors) |
| `hydronic-heater` | Hydronic heater controller firmware (ESP32) |
