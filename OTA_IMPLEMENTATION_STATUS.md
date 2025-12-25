# OTA Implementation Status

## ✅ What's Implemented

### ESP32 Firmware (paku-core)
- ✅ **OtaClient library** - Full OTA update client with HTTP download, checksum verification, partition management
- ✅ **MQTT OTA command handling** - Listens to `paku/devices/{deviceId}/cmd/ota` topic
- ✅ **OTA progress reporting** - Publishes progress updates during download/installation
- ✅ **OTA result reporting** - Publishes success/failure to `paku/devices/{deviceId}/ota/result`
- ✅ **Firmware validation** - Marks firmware as valid after successful boot
- ✅ **Automatic rollback** - Reverts to previous firmware if new one doesn't validate

### Backend (paku-iot)
- ✅ **Database schema** - All OTA tables created (`devices`, `firmware_releases`, `device_update_status`, `rollout_configurations`, `ota_events`)
- ✅ **Grafana dashboard** - OTA monitoring dashboard with device status, firmware versions, update history
- ✅ **MAC address support** - Devices can be targeted by MAC address or device ID

## ❌ What's Missing

### 1. **Device Registration** 
The collector doesn't register devices into the `devices` table when they connect.

**Current:** ESP32 publishes status to `paku/edge/{deviceId}/status`
**Missing:** Collector needs to extract device_id, MAC address, and firmware version from status messages and upsert into `devices` table

**Solution:** Add device registration handler to collector.py

### 2. **OTA Update Status Tracking**
The collector doesn't capture OTA update status/progress messages.

**Current:** ESP32 publishes to `paku/devices/{deviceId}/ota/result`
**Missing:** Collector needs to subscribe to `paku/devices/+/ota/#` and insert into `device_update_status` table

**Solution:** Add OTA message handler to collector.py

### 3. **Firmware Hosting**
No service to host firmware binaries and generate download URLs.

**Options:**
- Simple HTTP server serving from `/firmware/` directory
- S3/cloud storage with presigned URLs
- Integrated into existing service

### 4. **OTA Orchestration Service**
No service to trigger OTA updates based on rollout configurations.

**Missing:**
- Service that reads `rollout_configurations` table
- Matches devices based on `target_filter` (device_ids, mac_addresses, groups)
- Publishes OTA commands to `paku/devices/{deviceId}/cmd/ota`
- Tracks rollout progress

## 📋 Implementation Plan

### Phase 1: Device Registration (Highest Priority)
Add to collector.py:
```python
def register_device(conn, site_id, device_id, status_payload):
    """Extract device info from status and upsert into devices table"""
    mac_address = status_payload.get("wifi", {}).get("mac")
    firmware_version = status_payload.get("firmware_version", "unknown")
    device_model = status_payload.get("device_model", "esp32-s3")
    
    cur.execute("""
        INSERT INTO devices (device_id, mac_address, device_model, 
                           current_firmware_version, last_seen)
        VALUES (%s, %s, %s, %s, NOW())
        ON CONFLICT (device_id) 
        DO UPDATE SET 
            mac_address = EXCLUDED.mac_address,
            current_firmware_version = EXCLUDED.current_firmware_version,
            last_seen = NOW()
    """, (device_id, mac_address, device_model, firmware_version))
```

### Phase 2: OTA Status Tracking
Add to collector.py:
```python
# Subscribe to: paku/devices/+/ota/result, paku/devices/+/ota/progress

def handle_ota_result(conn, device_id, payload):
    """Insert OTA update result into device_update_status"""
    status = "success" if payload.get("success") else "failed"
    cur.execute("""
        INSERT INTO device_update_status 
            (device_id, firmware_version, status, error_message, 
             started_at, completed_at, reported_at)
        VALUES (%s, %s, %s, %s, %s, NOW(), NOW())
    """, (device_id, payload.get("version"), status, 
          payload.get("message"), payload.get("timestamp")))
```

### Phase 3: Firmware Hosting
Create simple firmware server:
```python
# Simple Flask/FastAPI service
@app.get("/firmware/{version}/esp32-s3.bin")
def download_firmware(version: str):
    return FileResponse(f"/firmware/{version}/esp32-s3.bin")
```

### Phase 4: OTA Orchestrator
Create orchestrator service that:
1. Polls `rollout_configurations` for active rollouts
2. Queries `devices` table filtered by `target_filter`
3. Publishes OTA commands to selected devices
4. Monitors progress in `device_update_status`

## 🎯 Quick Start: Get Data Flowing

To see devices in the OTA dashboard immediately:

1. **Add firmware version to status messages** (ESP32):
   ```cpp
   // In publishDeviceStatus()
   statusDoc["firmware_version"] = "1.0.0";  // Add this line
   statusDoc["device_model"] = "esp32-s3";
   ```

2. **Add device registration to collector** (see Phase 1 above)

3. Devices will automatically appear in dashboard when they publish status!

## Current State
- Infrastructure: ✅ 100% complete (database, dashboard)
- ESP32 OTA client: ✅ 100% complete (can receive and process updates)
- Backend integration: ❌ 0% complete (no device tracking or orchestration)

The system is **ready to receive OTA commands** but has **no automated way to trigger them**.
