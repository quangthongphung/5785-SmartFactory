# Module 1 Assignment — Protocol Comparison Report

**Student Name:** Quang Thong Phung
**Student ID:**   100987892
**Date:**         2026-05-28

---

## 5.1 QoS Comparison Results Table

> Run `pytest tests/mqtt/test_qos_loss.py -v -s` and paste the output table here.

| Protocol / QoS | Sent | Received | Lost (%) | Duplicates | Avg Latency (ms) |
|----------------|------|----------|----------|------------|-----------------|
| MQTT QoS 0 | 100 | 100 | 0.0% | 0 | 0.5 |
| MQTT QoS 1 | 100 | 100 | 0.0% | 0 | 0.3 |
| MQTT QoS 2 | 100 | 100 | 0.0% | 0 | 1.1 |
| CoAP NON | N/A | N/A | N/A | N/A | N/A |
| CoAP CON | N/A | N/A | N/A | N/A | N/A |
| AMQP (confirms off) | N/A | N/A | N/A | N/A | N/A |

**Analysis Questions:**

1. **Why does QoS 0 lose messages while QoS 1 and 2 do not?** *(2–3 sentences)*

   > _Your answer here_
   For QoS 0, it mostly relies on a best effort type delivery and it doesn’t ask for confirmations or any acknowledgements from the receiver. If packets are dropped while they are traveling, they simply aren’t resent. In contrast, QoS 1 and QoS 2 use an acknowledgement mechanism, so the delivery is made dependable, and in turn this helps avoid message loss

2. **QoS 1 may show duplicates. Under what circumstances does this happen, and is it a problem for sensor telemetry?** *(2–3 sentences)*

   > _Your answer here_
   Duplicates can show up when the sender ends up retransmitting a message because it doesn’t get a PUBACK confirmation in the right time window. The receiver might, already have handled the first copy of that same message though. For sensor telemetry, these occasional duplicates are often totally fine, since the newest readings quickly displace the earlier ones

3. **QoS 2 has higher latency than QoS 1. What causes this, and when is the trade-off worth it?** *(2–3 sentences)*

   > _Your answer here_
   QoS 2 requires a four-step handshake consisting of PUBLISH, PUBREC, PUBREL, and PUBCOMP messages. These additional control packets increase protocol overhead and latency. The trade-off is worthwhile for critical operations where duplicate processing must be completely eliminated.

---

## 5.2 CoAP–HTTP Proxy Mapping

> Run `pytest tests/coap/test_proxy.py -v -s` and record the observed HTTP headers.

| HTTP Header | CoAP Option | Your Observed Value |
|-------------|-------------|---------------------|
| Content-Type | Content-Format | application/json |
| Cache-Control: max-age | Max-Age | Not Observed |
| ETag | ETag | Not Observed |
| Location | Location-Path | Not Observed |

---

## 5.3 Protocol Selection Recommendation

*(500–700 words. Justify each recommendation with specific technical evidence from your implementation and packet captures.)*

### Data Path Recommendations

| Data Path | Recommended Protocol | Justification |
|-----------|---------------------|---------------|
| Sensor → Cloud (high frequency, <100 ms latency) | MQTT QoS 1 | Reliable delivery with low overhead and low latency |
| Actuator commands (safety-critical, exactly-once) | MQTT QoS 2 | Guarantees exactly-once processing |
| Backend service-to-service routing | AMQP | Advanced routing and queue management |
| OTA firmware delivery to constrained MCU (Class 2) | CoAP | Lightweight protocol with efficient resource usage |

### Detailed Justification

> *(Write 500–700 words here. Each recommendation must cite specific evidence — e.g. measured latency values from Section 5.1, packet overhead observed in Task 4, or implementation complexity experienced in Tasks 1–3.)*
MQTT kinda feels like the best choice for sensor-to-cloud telemetry, since it supports publish subscribe communication fairly efficiently with very little protocol overhead. In the testing phase, MQTT showed low latency somewhere around 0.3 ms to 1.1 ms while still keeping message delivery dependable. MQTT QoS 1 is sort of a decent middle ground between reliability and performance, because it makes sure messages are delivered at least once , without the extra handshake work that comes with QoS 2.

When we talk about safety-critical actuator commands, MQTT QoS 2 becomes the go-to option. QoS 2 runs with a four step acknowledgement sequence, using PUBLISH, PUBREC, PUBREL, and PUBCOMP packets. Sure, this adds some extra latency, but it also gives exactly once message delivery, and it blocks duplicate command execution. That property matters quite a lot for industrial control contexts where redoing a command multiple times could cause real safety issues.

AMQP is recommended for backend service-to-service communication, mainly because it supports more advanced routing mechanics things like exchanges and queues plus acknowledgements and message persistence. Compared with MQTT, AMQP generally offers more layered enterprise messaging features. Those capabilities are useful for microservice setups and business processing workflows that want adaptable routing and delivery guarantees, without too much hassle.

For OTA firmware updates aimed at constrained IoT devices, CoAP is kind of the preferred protocol. It runs over UDP and in general it keeps a much smaller protocol footprint than HTTP. In the packet captures, you can see successful GET requests, followed by 2.05 Content responses , all while keeping the overhead low. CoAP also has Observe and Block-wise Transfer mechanisms , which makes it a good fit for devices with tight resources. Even so, Block2 transfer wasn’t really needed in the captured payload because the response was only 114 bytes , but the protocol still supports segmented transfers when firmware images get larger.

Packet analysis also made the protocol differences pretty clear. With MQTT you needed the whole sequence—CONNECT, SUBSCRIBE, PUBLISH, and then PUBACK exchanges—so it clearly shows the broker based architecture. CoAP instead followed a simple request response rhythm, basically GET plus ACK 2.05 Content messages. That streamlined style helps with lower resource use, and it’s one reason CoAP stays attractive for constrained devices. From the implementation experience side, MQTT ended up being the easiest solution for real-time telemetry, CoAP gave the most lightweight communication model , and AMQP still looks like the strongest choice for complex backend messaging cases.
---

## 5.4 Reflection

*(300–400 words addressing all three prompts below.)*

### Technical Challenge

> *Describe one technical challenge you encountered in the implementation and how you resolved it.*
One major technical challenge was setting up the Python environment and the protocol libraries the right way, sorta. At the beginning the project ran into compatibility troubles tied to aiocoap and which Python version was being used, it was a bit messy. After that, more troubleshooting was needed so that MQTT and CoAP parts would actually run, and they had to satisfy all those automated tests too. In the end it got fixed by installing the required dependencies from requirements.txt, then checking compatibility with the starter kit that was provided, just to be sure.

### Most Surprising Protocol Difference

> *Describe the most surprising difference you observed between the protocols during the packet capture task.*
The most surprising difference I noticed was the contrast between MQTT and CoAP communication models, honestly it felt kind of back to front at first. MQTT leans on a broker-based publish-subscribe setup that really needs connection setup, subscriptions, and acknowledgements before any actual data can go through. CoAP on the other hand uses a lightweight REST style request-response pattern over UDP so the packet flow stays pretty straightforward . When I looked at packet captures, it was obvious that CoAP tends to need fewer protocol exchanges than MQTT, like a smaller number of little handshakes overall

### Most Complex Protocol to Implement

> *Which protocol was the most complex to implement correctly, and what specifically made it harder?*
MQTT QoS 2 was, honestly the most complex bit to understand and implement it right. Compared with QoS 0 and QoS 1, QoS 2 asks for a four stage acknowledgement dance using PUBREC, PUBREL, and PUBCOMP packets, in sequence. Figuring out what each stage is doing, and then tracing that packet flow inside Wireshark   took more effort than I expected , not gonna lie. Still, even with the extra implementation weight, the protocol delivers exactly once arrival guarantees, which turns out to be super useful in safety critical scenarios where losing or duplicating messages is not acceptable.

---

*Module 1 Assignment — Real-Time Data Analytics for IoT*
