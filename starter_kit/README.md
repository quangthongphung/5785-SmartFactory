# Module 1 Assignment — SmartFactory IoT Protocol Integration

**Real-Time Data Analytics for IoT** · Graduate Course · Module 1

**Student Name:** Quang Thong Phung  
**Student ID:** 100987892

---

## Quick Start

```bash
# Install dependencies
pip install -r requirements.txt

# Start Docker services
docker compose up -d

# Run MQTT components
python -m src.mqtt.publisher
python -m src.mqtt.subscriber

# Run CoAP components
python -m src.coap.server
python -m src.coap.observer

# Run all tests
pytest tests/ -v
```

---

## Repository Structure

```text
starter_kit/
├── src/
│   ├── mqtt/
│   │   ├── publisher.py
│   │   └── subscriber.py
│   └── coap/
│       ├── server.py
│       └── observer.py
│
├── tests/
│   ├── mqtt/
│   └── coap/
│
├── report/
│   ├── packet_analysis.md
│   └── comparison_report.md
│
├── captures/
│   ├── mqtt.pcapng
│   └── coap.pcapng
│
├── docker-compose.yml
├── requirements.txt
├── pytest.ini
└── README.md
```

---

## Running Individual Components

### MQTT

Terminal 1:

```bash
python -m src.mqtt.publisher
```

Terminal 2:

```bash
python -m src.mqtt.subscriber
```

### CoAP

Terminal 1:

```bash
python -m src.coap.server
```

Terminal 2:

```bash
python -m src.coap.observer
```

---

## Running Tests

Run all tests:

```bash
pytest tests/ -v
```

Run MQTT tests:

```bash
pytest tests/mqtt/ -v
```

Run MQTT QoS experiment:

```bash
pytest tests/mqtt/test_qos_loss.py -v -s
```

Run CoAP tests:

```bash
pytest tests/coap/ -v
```

---

## Packet Capture

### MQTT Capture

```powershell
& "C:\Program Files\Wireshark\tshark.exe" -i 6 -f "tcp port 1883" -a duration:30 -w captures\mqtt.pcapng
```

Verified packets:

- CONNECT
- SUBSCRIBE
- SUBACK
- PUBLISH
- PUBACK

### CoAP Capture

```powershell
& "C:\Program Files\Wireshark\tshark.exe" -i 6 -f "udp port 5683" -a duration:30 -w captures\coap.pcapng
```

Verified packets:

- CON GET
- ACK
- 2.05 Content
- Observe notifications

---

## Results Summary

### MQTT QoS Comparison

| QoS | Sent | Received | Lost | Avg Latency (ms) |
|------|------|------|------|------|
| QoS 0 | 100 | 100 | 0 | 0.5 |
| QoS 1 | 100 | 100 | 0 | 0.3 |
| QoS 2 | 100 | 100 | 0 | 1.1 |

### CoAP

Successfully verified:

- Resource discovery
- GET requests
- Observe notifications
- JSON payload delivery
- Actuator control

---

## Submission Notes

Completed:

- MQTT Publisher
- MQTT Subscriber
- MQTT QoS Comparison
- CoAP Server
- CoAP Observer
- Packet Analysis
- Protocol Comparison Report

All provided MQTT and CoAP tests pass successfully.

---

*Graduate Course: Real-Time Data Analytics for IoT · Module 1*