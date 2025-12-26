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

## Phase 2: Update Workflow (PENDING)

**Prerequisites:**
- ✅ All devices running firmware v1.4.0 or higher
- All devices must show `current_firmware_version >= 1.4.0` in database

**Changes to Make:**
In `paku-iot/.github/workflows/ota-update-esp.yaml`:

### Line ~431: Update MQTT topic in send_ota_command function

**Before:**
```bash
mosquitto_pub -h "$MQTT_BROKER" -p 1883 \
  -t "paku/devices/${device_id}/cmd/ota" \
  -m "${ota_payload}"
```

**After:**
```bash
mosquitto_pub -h "$MQTT_BROKER" -p 1883 \
  -t "paku/edge/${device_id}/cmd/ota" \
  -m "${ota_payload}"
```

### Test Plan

1. **Verify all devices are updated:**
   ```bash
   curl -s -H "X-API-Key: ${OTA_API_KEY}" \
     "http://37.27.192.107:8080/api/admin/devices?limit=500" \
     | jq -r '.devices[] | select(.current_firmware_version < "1.4.0") | "\(.device_id) - \(.current_firmware_version)"'
   ```
   Should return empty (no devices below v1.4.0)

2. **Test new topic manually:**
   ```bash
   # Pick a test device
   DEVICE_ID="ESP8266-9608CF16"
   
   # Send OTA command to NEW topic
   mosquitto_pub -h 37.27.192.107 -p 1883 \
     -t "paku/edge/${DEVICE_ID}/cmd/ota" \
     -m '{"url":"http://example.com/test.bin","version":"test"}'
   
   # Monitor device response
   mosquitto_sub -h 37.27.192.107 -p 1883 -v \
     -t "paku/devices/${DEVICE_ID}/ota/#"
   ```

3. **Update workflow and test:**
   - Make the topic change in workflow file
   - Run workflow with single device
   - Verify OTA command received and processed

## Phase 3: Cleanup (Future)

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

Check device registration after Phase 2:
```bash
# Verify devices receiving OTA commands
docker logs -f paku_collector | grep "ota"

# Check MQTT traffic
mosquitto_sub -h 37.27.192.107 -p 1883 -v -t 'paku/edge/+/cmd/ota'
```

---

**Last Updated:** 2025-12-26  
**Phase 1 Completion:** 2025-12-26  
**Phase 2 Target:** After all devices running v1.4.0
