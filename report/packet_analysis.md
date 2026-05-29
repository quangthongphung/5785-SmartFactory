## 4.2 MQTT Packet Annotations

### CONNECT Packet

| Field | Offset (bytes) | Raw Hex | Decoded Value |
|-------|---------------|---------|---------------|
| Frame type + flags (byte 1) | 0 | `10` | Type=CONNECT (0001), flags=0000 |
| Remaining length (byte 2) | 1 | `27` | 39 bytes |
| Protocol name length | 2–3 | `00 04` | 4 |
| Protocol name | 4–7 | `4D 51 54 54` | "MQTT" |
| Protocol version | 8 | `04` | MQTT v3.1.1 |
| Connect flags | 9 | `02` | Clean Session enabled |
| Keep-alive | 10–11 | `00 3C` | 60 seconds |
| Client ID length | 12–13 | `00 1B` | 27 |
| Client ID | 14–40 | smartfactory-subscriber-001 | Subscriber identifier |

### Connect Flags byte breakdown

| Bit | Name | Value | Meaning |
|-----|------|-------|---------|
| 7 | Username flag | 0 | Username not present |
| 6 | Password flag | 0 | Password not present |
| 5 | Will retain | 0 | Disabled |
| 4–3 | Will QoS | 00 | QoS 0 |
| 2 | Will flag | 0 | Disabled |
| 1 | Clean session | 1 | Enabled |
| 0 | Reserved | 0 | Valid |

---

### QoS 1 PUBLISH Packet

| Field | Offset (bytes) | Raw Hex | Decoded Value |
|-------|---------------|---------|---------------|
| Fixed header byte 1 | 0 | `32` | Type=PUBLISH(0011), DUP=0, QoS=01, RETAIN=0 |
| Remaining length | 1 | `A3` | 163 bytes |
| Topic length | 2–3 | `00 19` | 25 |
| Topic string | 4–28 | factory/line1/temperature | Temperature topic |
| Packet Identifier | 29–30 | `03 A1` | 929 |
| Payload | 31–… | JSON payload | Sensor telemetry data |

### Fixed header byte 1 bit expansion

| Bits 7–4 (packet type) | Bit 3 (DUP) | Bits 2–1 (QoS) | Bit 0 (RETAIN) |
|------------------------|-------------|----------------|----------------|
| `0011` = PUBLISH (3) | `0` = No duplicate | `01` = QoS 1 | `0` = Not retained |

---

### PUBACK Packet

| Field | Offset | Raw Hex | Decoded Value |
|-------|--------|---------|---------------|
| Fixed header | 0 | `40` | Type=PUBACK (0100) |
| Remaining length | 1 | `02` | 2 bytes |
| Packet Identifier | 2–3 | `03 A1` | 929 |

**Packet Identifier match:** PUBLISH PKT ID = 929 ; PUBACK PKT ID = 929 ; **Match? YES**

---

## 4.3 CoAP Packet Annotations

### CON GET Request

| Field | Value | Meaning |
|---------|---------|---------|
| Version | 1 | CoAP Version 1 |
| Type | Confirmable (CON) | Reliable request |
| Token Length | 2 | 2-byte token |
| Code | GET (1) | Resource retrieval |
| Message ID | 40079 | Request identifier |
| Token | 5079 | Correlation token |
| Observe | 0 | Register observation |
| Uri-Host | localhost | Destination host |
| Uri-Path | factory | Root resource |
| Uri-Path | line2 | Production line |
| Uri-Path | temperature | Temperature sensor |

URI:

coap://localhost/factory/line2/temperature

---

### ACK 2.05 Content Response

| Field | Value | Meaning |
|---------|---------|---------|
| Version | 1 | CoAP Version 1 |
| Type | ACK | Acknowledgement |
| Code | 2.05 Content | Successful response |
| Message ID | 40079 | Matches request |
| Token | 5079 | Matches request |
| Observe | 0 | Initial observe sequence |
| Content Format | application/json | JSON payload |
| Payload Length | 114 bytes | Sensor data |

---

### Observe Notification

| Field | Value |
|-------|-------|
| Observe option number | 6 |
| Observe sequence value | 0 |
| Message type | ACK |
| Response code | 2.05 Content |

The Observe option establishes a subscription relationship between the client and the server. After registration, the server periodically sends updated sensor readings without requiring additional GET requests.

---

## Block2 Transfer Analysis

No Block2 option was observed in the captured temperature response.

The temperature payload size was only 114 bytes and therefore fit into a single CoAP message without fragmentation.

| Item | Value |
|---------|---------|
| Block2 Used | No |
| Fragmentation Required | No |
| Payload Size | 114 bytes |

---

## Protocol Transfer Analysis

### MQTT Transfer Flow

CONNECT → CONNACK → SUBSCRIBE → SUBACK → PUBLISH → PUBACK

MQTT uses a broker-based publish/subscribe model. QoS 1 delivery guarantees at-least-once delivery through acknowledgement packets.

### CoAP Transfer Flow

GET → ACK 2.05 Content

CoAP uses a lightweight request-response model over UDP. The Observe option allows the server to push updates to subscribed clients.

---

## Conclusion

The packet captures successfully demonstrated both MQTT and CoAP communication mechanisms. MQTT provided reliable publish-subscribe telemetry transport through QoS acknowledgements, while CoAP provided lightweight REST-style resource access and observation functionality. Both protocols successfully transmitted SmartFactory sensor data and fulfilled the communication requirements of the assignment.