# Hardware Integration Specification: Biometric TCP/IP Bridge

## 1. Networking Topography (Local Bridge Proxy)
Since physical biometric readers (ZKTeco/Hikvision) sit inside the local area network (LAN) in Surquillo behind a dynamic IP address provider, they stream real-time attendance sequences using the **ADMS (Automatic Data Master Server)** protocol via HTTP/POST reversing channels targeting our cloud proxy.

[ZKTeco Hardware Terminal] ──(HTTP POST ADMS Event)──> [arellan-api-gateway (Cloud)]
│
[arellan-hardware-iot Microservice]
│
[NestJS Event Dispatcher]

## 2. Event Payload Data Contract
When a staff member (mechanic, practitioner) presses their finger or passes facial recognition at the terminal gate, the hardware sends the following explicit payload to the webhook endpoint `/api/v1/iot/biometric-punch`:

```json
{
  "device_metadata": {
    "serial_number": "ZK-TERMINAL-SURQUILLO-01",
    "firmware_version": "8.0.4-ADMS",
    "mac_address": "00:1B:44:11:3A:B7"
  },
  "punch_payload": {
    "employee_hardware_id": "MEC-007",
    "timestamp": "2026-05-23T18:05:22.000Z",
    "punch_type": "CHECK_OUT",
    "verification_mode": "FINGERPRINT"
  }
}

## 3. High-Complexity Edge Case Handlers
Local Network Blackout (Offline Buffer Mode): If the internet connection drops in Surquillo, the ZKTeco hardware caches up to 50,000 punch events locally.

Once connectivity is restored via the arellan-infrastructure router failover, the terminal pushes the entire bulk buffer array.

The Backend Rule Engine: The arellan-hardware-iot service must process logs sequentially sorted by the hardware timestamp rather than the cloud database insertion time. If an employee logs a CHECK_OUT timestamp that conflicts with an open, active Work Order on an uncompleted vehicle, the system triggers a CRITICAL_OPERATIONAL_ANOMALY flag inside the logging console.