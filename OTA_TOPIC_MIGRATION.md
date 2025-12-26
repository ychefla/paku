# OTA Topic Migration Plan

Migration from `paku/devices/{id}/cmd/ota` to `paku/edge/{id}/cmd/ota`

## Rationale

Align OTA commands with the edge device topic structure for consistency:
- Status: `paku/edge/{deviceId}/status`
- Config: `paku/edge/{deviceId}/config`
- Control: `paku/edge/{deviceId}/control`
- **OTA: `paku/edge/{deviceId}/cmd/ota`** ← New location

## Phase 1: Dual Topic Support ✅ COMPLETED (v1.4.0)

**Status:** Firmware updated to v1.4.0 (commit 85ef9a7)

**Changes:**
- Firmware subscribes to **both** topics:
  - Legacy: `paku/devices/{deviceId}/cmd/ota`
  - New: `paku/edge/{deviceId}/cmd/ota`
- OTA handler accepts commands from either topic
- Version bumped from 1.3.0 → 1.4.0

**Action Required:**
1. Deploy firmware v1.4.0 to all devices using current workflow
2. Wait for all devices to update (check database for `current_firmware_version`)

## Phase 2: Update Workflow ✅ COMPLETED

**Prerequisites:**
- ✅ All devices running firmware v1.4.0 or higher
- All devices must show `current_firmware_version >= 1.4.0` in database

**Status:** Workflow updated (commit previous), all devices using new command topic

**Changes Made:**
In `paku-iot/.github/workflows/ota-update-esp.yaml`:
- Updated MQTT topic from `paku/devices/{id}/cmd/ota` → `paku/edge/{id}/cmd/ota`

## Phase 3: Remove Legacy Subscription ✅ COMPLETED (v1.4.1)

**Status:** Firmware updated to v1.4.1

**Changes:**
- Removed legacy `paku/devices/{id}/cmd/ota` subscription
- Device only subscribes to `paku/edge/{id}/cmd/ota`
- Version bumped from 1.4.0 → 1.4.1

## Phase 4: Update Response Topics ✅ COMPLETED (v1.4.2)

**Status:** Firmware updated to v1.4.2

**Changes:**
- Updated all OTA response topics from `paku/devices` to `paku/edge`:
  - Status: `paku/edge/{id}/ota/status`
  - Progress: `paku/edge/{id}/ota/progress`
  - Result: `paku/edge/{id}/ota/result`
- Updated `ota_update.sh` script to monitor new topics
- Version bumped from 1.4.1 → 1.4.2

**Files Modified:**
- `paku_core/src/main.cpp` - Updated 3 topic paths
- `paku_core/ota_update.sh` - Updated monitoring topics

## Phase 5: Cleanup (Future)

**Timeline:** After 1-2 months of stable operation

**Changes:**
- Remove legacy `paku/devices/{id}/cmd/ota` subscription from firmware
- Update documentation to reference only new topic
- Version bump to 1.5.0

## Rollback Plan

If Phase 2 fails:
- Revert workflow changes
- Devices will continue working with legacy topic
- No firmware changes needed (dual support maintained)

## Monitoring

Check device OTA activity:
```bash
# Verify devices receiving OTA commands
docker logs -f paku_collector | grep "ota"

# Check MQTT command topic
mosquitto_sub -h 37.27.192.107 -p 1883 -v -t 'paku/edge/+/cmd/ota'

# Check MQTT response topics
mosquitto_sub -h 37.27.192.107 -p 1883 -v -t 'paku/edge/+/ota/#'
```

---

**Last Updated:** 2025-12-26  
**Phase 1 Completion:** 2025-12-26  
**Phase 2 Completion:** 2025-12-26  
**Phase 3 Completion:** 2025-12-26  
**Phase 4 Completion:** 2025-12-26
