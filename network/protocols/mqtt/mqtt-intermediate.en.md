# 📡 MQTT — Part 2: Intermediate | Packet Structure and Mechanisms

# 🟡 Level Two: Intermediate — Packet Dissection and Control Mechanisms

At the intermediate level, we explore protocol engineering details: count packet bytes, examine MQTT over UDP and WebSocket, and learn about the `Retain` and `LWT` mechanisms.

## 1. Exact MQTT Packet Structure (Byte-by-Byte)

Why is MQTT described as extremely lightweight? HTTP text headers can easily be $200$ to $800\text{ bytes}$ in size, whereas MQTT is a **binary-based** protocol whose smallest control packet is only $2\text{ bytes}$!

### 📦 MQTT Packet Anatomy Diagram

```
╭─────────────────────────────────────────────────────────────────────────────╮
│ 🟦 Part One: Fixed Header                                                   │
│    • Presence: Included in every packet                                    │
│    • Size: 2 to 5 bytes (packet type, flags, and Remaining Length)          │
├─────────────────────────────────────────────────────────────────────────────┤
│ 🟨 Part Two: Variable Header                                                │
│    • Presence: Depends on packet type                                      │
│    • Size: Variable (may include topic name and 2-byte packet ID when QoS>0)│
├─────────────────────────────────────────────────────────────────────────────┤
│ 🟩 Part Three: Payload (message body / content)                             │
│    • Presence: Included in packets such as PUBLISH and CONNECT              │
│    • Size: 0 bytes up to 256 MB (JSON, text, binary, Protobuf)             │
╰─────────────────────────────────────────────────────────────────────────────╯
```

### A) 🟦 Fixed Header Dissection (2 to 5 bytes)

The fixed header is present in **every** MQTT packet. Its bits are arranged as follows:

| Byte number | Bit 7 | Bit 6 | Bit 5 | Bit 4 | Bit 3 | Bit 2 | Bit 1 | Bit 0 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Byte 1** | 🟦 **Packet Type** | 🟦 **Packet Type** | 🟦 **Packet Type** | 🟦 **Packet Type** | 🟧 **DUP** | 🟩 **QoS MSB** | 🟩 **QoS LSB** | 🟪 **RETAIN** |
| **Byte 2..5** | 🟨 **Continuation Bit** | 🟨 **Length** | 🟨 **Length** | 🟨 **Length** | 🟨 **Length** | 🟨 **Length** | 🟨 **Length** | 🟨 **Length** |

#### 1. The First Byte ($1\text{ byte} = 8\text{ bits}$):
* **🟦 First four bits (Bits 7 through 4) — Packet Type:** Identify what the packet does (four bits can represent $2^4 = 16$ packet types):
  * `1` (`0001`): `CONNECT` (client connection request to the broker)
  * `2` (`0010`): `CONNACK` (connection acknowledgement)
  * `3` (`0011`): `PUBLISH` (message publication)
  * `4` through `7`: `PUBACK`, `PUBREC`, `PUBREL`, `PUBCOMP` (QoS 1 and 2 handling)
  * `8` (`1000`): `SUBSCRIBE` (topic subscription request)
  * `12` (`1100`): `PINGREQ` (connection liveness check)
  * `13` (`1101`): `PINGRESP` (liveness-check response)
  * `14` (`1110`): `DISCONNECT` (graceful disconnection)
* **Lower four bits (Bits 3 through 0) — Flags:** (for the `PUBLISH` packet)
  * **🟧 Bit 3 (`DUP` — Duplicate):** If it is `1`, this packet was sent before but did not receive an acknowledgement and is being retransmitted.
  * **🟩 Bits 2 & 1 (`QoS`):** Two bits represent the service level (`00` for QoS 0, `01` for QoS 1, and `10` for QoS 2).
  * **🟪 Bit 0 (`RETAIN`):** If it is `1`, it tells the broker to store the latest version of this message.

#### 2. Bytes Two through Five (🟨 Remaining Length):
This field indicates the combined size, in bytes, of the `Variable Header` and `Payload`. MQTT uses a clever algorithm called **Variable Byte Integer**:
* Each byte contributes $7\text{ bits}$ to the value and uses $1\text{ bit}$ **(the eighth bit or MSB)** as a Continuation Bit (if the eighth bit is `1`, the next byte is also part of the packet length).
* Therefore, for small messages (up to $127\text{ bytes}$), this field occupies only $1\text{ byte}$!
* The maximum addressable size using $4\text{ bytes}$ is:

$$\text{Max Payload Size} = 128^4 - 1 = 268,435,455\text{ bytes} \approx 256\text{ MB}$$

| Number of Remaining Length bytes | Minimum representable length | Maximum representable length |
| :---: | :---: | :---: |
| 🟢 $1\text{ byte}$ | $0\text{ bytes}$ | $127\text{ bytes}$ |
| 🟡 $2\text{ bytes}$ | $128\text{ bytes}$ ($128^1$) | $16,383\text{ bytes}$ ($16\text{ KB}$) |
| 🟠 $3\text{ bytes}$ | $16,384\text{ bytes}$ ($128^2$) | $2,097,151\text{ bytes}$ ($2\text{ MB}$) |
| 🔴 $4\text{ bytes}$ | $2,097,152\text{ bytes}$ ($128^3$) | $268,435,455\text{ bytes}$ ($256\text{ MB}$) |

### B) 🟨 Variable Header Dissection
This section sits between the fixed header and the payload, and its contents vary by packet type. For example, a `PUBLISH` packet’s variable header has two parts:
1. **Topic length and Topic Name:** First, $2\text{ bytes}$ specify the length of the topic string, followed by the topic’s UTF-8 characters.
2. **Packet Identifier:** A $2\text{ byte}$ ($16\text{ bit}$) number included **only when** $\text{QoS} > 0$ (that is, QoS 1 or QoS 2), allowing the broker and client to track messages.

### C) 🟩 Payload (Message Content)
This is the final part of the packet. MQTT is entirely **data-agnostic**: you can put plain text, JSON, XML, Protocol Buffers (Protobuf), an image, or binary audio in the payload. Some packets, such as `PINGREQ` or `DISCONNECT`, have no payload at all.

#### 🧮 Exact PUBLISH Packet Size Example:
Suppose we want to publish the value `25` on topic `a/b` with `QoS 1`:
* **🟦 Fixed Header:** $1\text{ byte}$ (Type/Flags) + $1\text{ byte}$ (Remaining Length) = $2\text{ bytes}$
* **🟨 Variable Header:** $2\text{ bytes}$ (Topic Length) + $3\text{ bytes}$ (`a/b`) + $2\text{ bytes}$ (Packet ID) = $7\text{ bytes}$
* **🟩 Payload:** $2\text{ bytes}$ (`25`) = $2\text{ bytes}$
* **✅ Total packet size:** $2 + 7 + 2 = \mathbf{11\text{ bytes}}$!

---

## 2. MQTT over UDP and the Key Role of QoS 2

As discussed in the Basic section, classic MQTT runs over **TCP**. In many IoT scenarios (such as battery-powered sensors, satellite networks, Zigbee, or low-power radio links), establishing TCP’s 3-Way Handshake and keeping a socket open can greatly increase battery consumption and network overhead.

This is where implementations of **MQTT over UDP** (either custom/raw or using the official **MQTT-SN** specification, or *MQTT for Sensor Networks*) come into discussion.

### What Is UDP’s Big Challenge, and How Is It Addressed with QoS 2?

**UDP (User Datagram Protocol)** is a **connectionless** and **unreliable** Layer 4 protocol. This means it:
* Sends packets without prior coordination.
* Provides no guarantee that a packet will reach its destination.
* May duplicate a packet in the network or deliver packets out of order.

**💡 Engineering approach (UDP combined with QoS 2):**
When we switch the Transport Layer from TCP to **UDP**, we move the burden of “delivery assurance and duplicate prevention” off Layer 4 (the network) and directly to **Layer 7 (the application—that is, MQTT packet logic itself)!**

### 📊 Delivery Assurance over UDP with QoS 2

```
 📡 Sensor over UDP                                      🟠 Gateway / Broker (UDP Listener)
 ════════════════════                                      ═══════════════════════════════
  (No initial TCP handshake required!)
           │                                                              │
           │  🟦 Step 1: Send UDP datagram containing [PUBLISH | ID=101 | QoS=2] │
           │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │
           │                                                              │ 🟢 Receive packet and lock
           │  🟩 Step 2: UDP datagram response containing [PUBREC | ID=101]           │    identifier 101 (prevent duplicates)
           │ ◄╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌ │
           │                                                              │
 ╭─────────┴──────────────────────────────────────────────────────────────┴─────────╮
 │ 🔴 UDP network failure: suppose the next packet is lost over radio!           │
 │                                                                                  │
 │         │  🟥 Send [PUBREL | ID=101]                                  │         │
 │         │ ┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄ ❌ (packet lost on UDP network)         │         │
 │         │                                                              │         │
 │  ⏱️ [Client retry timer expires]                                      │         │
 │         │                                                              │         │
 │         │  🟧 Step 3 (retry): resend [PUBREL | ID=101]          │         │
 │         │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │         │
 ╰─────────┬──────────────────────────────────────────────────────────────┬─────────╯
           │                                                              │ 🟢 Release identifier 101
           │  🟪 Step 4: final acknowledgement datagram [PUBCOMP | ID=101] │
           │ ◄╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌ │
           │                                                              │
  ✅ Exactly-once delivery over unreliable UDP!
```

### Benefits of Combining UDP with Application-Layer QoS 2:

1. **🔋 Eliminates persistent TCP connection overhead:** A device can stay in Deep Sleep, wake up, exchange the UDP datagrams in the `QoS 2` sequence (`PUBLISH` $\rightarrow$ `PUBREC` $\rightarrow$ `PUBREL` $\rightarrow$ `PUBCOMP`), and go back to sleep.
2. **🛡️ Prevents all UDP loss or duplication:** If a packet is lost because of UDP’s nature, the MQTT client’s internal timer triggers retransmission; because the broker locks the `Packet ID` until it receives `PUBREL`, a duplicate message is never delivered to the system.
3. **📉 Reduces Layer 4 header size:** A TCP header is at least $20\text{ bytes}$, while a UDP header is only $8\text{ bytes}$.

> [!NOTE]
> In **MQTT-SN** (designed for UDP-based networks), in addition to `QoS 0, 1, 2`, there is a special **`QoS -1`** level. It lets a client fire a raw UDP packet at the broker without registering or even sending `CONNECT`. Instead of sending a long topic name, it also uses a two-byte **Topic ID** to minimize UDP packet size.

---

## 3. Encapsulating MQTT in WebSockets (MQTT over WebSockets)

Suppose you want to build an admin dashboard with **React, Vue, or JavaScript** that runs directly in the user’s browser and displays sensor data in real time.

### What Is the Problem?
For security reasons (sandboxing), web browsers do not allow opening a raw TCP socket to the broker’s port `1883`! Browsers understand `HTTP/HTTPS` and **`WebSocket (ws/wss)`**.

### Solution: Encapsulation
To solve this problem, we take MQTT binary packets unchanged and place (encapsulate) them inside the **payloads of WebSocket frames**.

### 📊 Encapsulation Layers Diagram (MQTT over WebSockets)

```
╭───────────────────────────────────────────────────────────────────────────╮
│ 🟪 Layer 2: Network frame (Ethernet / Wi-Fi Frame)                       │
│  ╭─────────────────────────────────────────────────────────────────────╮  │
│  │ 🟨 Layer 3: IP packet (IPv4 / IPv6)                                 │  │
│  │  ╭───────────────────────────────────────────────────────────────╮  │  │
│  │  │ 🟦 Layer 4: TCP segment (Ports 8083 / 8084)                   │  │  │
│  │  │  ╭─────────────────────────────────────────────────────────╮  │  │  │
│  │  │  │ 🔒 Security layer: TLS / SSL (when using wss://)         │  │  │  │
│  │  │  │  ╭───────────────────────────────────────────────────╮  │  │  │  │
│  │  │  │  │ 🟧 WebSocket Binary Frame (0x02)                  │  │  │  │  │
│  │  │  │  │  ╭─────────────────────────────────────────────╮  │  │  │  │  │
│  │  │  │  │  │ 🟩 Complete MQTT binary packet              │  │  │  │  │  │
│  │  │  │  │  │ [Fixed Header] [Variable Header] [Payload]  │  │  │  │  │  │
│  │  │  │  │  ╰─────────────────────────────────────────────╯  │  │  │  │  │
│  │  │  │  ╰───────────────────────────────────────────────────╯  │  │  │  │
│  │  │  ╰─────────────────────────────────────────────────────────╯  │  │  │
│  │  ╰───────────────────────────────────────────────────────────────╯  │  │
│  ╰─────────────────────────────────────────────────────────────────────╯  │
╰───────────────────────────────────────────────────────────────────────────╯
```

### Exact MQTT over WebSocket Connection Steps:

1. **🤝 Initial negotiation (HTTP Upgrade Handshake):**
   The browser sends an HTTP request to the broker (usually on port `8083` for `ws` or `8084` for `wss`) and requests an upgrade to WebSocket using the **`mqtt`** subprotocol:
   ```http
   GET /mqtt HTTP/1.1
   Host: broker.example.com:8083
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
   Sec-WebSocket-Protocol: mqtt
   Sec-WebSocket-Version: 13
   ```

2. **🔄 Broker acknowledgement (Switching Protocols):**
   The broker responds with status code `101 Switching Protocols`, opening the bidirectional WebSocket channel.

3. **📦 Sending binary frames:**
   From this point on, a JavaScript library (such as `MQTT.js`) creates a binary `CONNECT` or `PUBLISH` packet, places it inside a **WebSocket Binary Frame (`Opcode = 0x02`)**, and sends it. The broker opens (decapsulates) the WebSocket frame and processes the MQTT packet inside just like a regular packet!

> [!TIP]
> **Additional benefit of WebSocket:** Since WebSocket traffic can use standard web ports (`80` and `443`), it can readily pass through strict corporate firewalls and proxies (such as Cloudflare or Nginx Reverse Proxy).

---

## 4. Retained Messages with an Example

Normally, when a Publisher sends a message on a topic, the broker sends it to clients subscribed to that topic **at that moment**, then discards it. If a new client connects 5 seconds later and subscribes to that topic, it receives no message until the sender publishes again!

### What Does the Retain Flag Do?
If the **`RETAIN = 1`** bit is set when sending a `PUBLISH` packet, the broker is instructed:
> “Keep this message in memory as the **Last Known Good Value** for this topic. Whenever a new client subscribes to this topic in the future, send this message to it **immediately**!”

* For each topic, the broker stores **only one retained message** (the latest message published with the Retain flag).

### 💡 Practical Example: Garage Door Status Sensor

```
 🚪 Garage Door Sensor                    🟠 Broker                            📱 New Mobile App
 ════════════════════                     ═════════════════                     ═════════════════════════
           │                                      │                                         │
 [08:00 AM]                                 │                                         │
           │ 1. PUBLISH (Retain = True)           │                                         │
           │    Topic: garage/door                │                                         │
           │    Payload: {"state": "OPEN"}        │                                         │
           │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │ 💾 Store retained topic value           │
           │                                      │                                         │
           ⋮         (6 hours pass...)             ⋮                                 [02:00 PM]
           │                                      │         2. SUBSCRIBE (garage/door)      │
           │                                      │ ◄━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
           │                                      │                                         │
           │                                      │ 3. Immediately send latest stored value ⚡│
           │                                      │    Payload: {"state": "OPEN"} [Retained]│
           │                                      │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │
```

> [!IMPORTANT]
> **How do we remove a retained message from the broker’s memory?**
> Publish a `PUBLISH` packet with an **empty payload (zero-byte payload)** and **`Retain = true`** to the same topic. The broker will delete the previous message.

---

## 5. Client Last Will and Testament (LWT), with an Example

In the real world, devices do not always leave the network gracefully (by sending a `DISCONNECT` packet): power may fail, a battery may run out, or a network cable may be pulled. How can other clients know that this device has suddenly gone offline?

This is where **Last Will and Testament (LWT)** comes into play.

### How LWT Works:
When the client first connects to the broker, it provides its Will inside the `CONNECT` packet and says:
> “Broker! Here are my Will Topic, message text, QoS, and Retain setting. Do not use it while I am alive and sending Keep-Alive packets; but if my connection is lost **unexpectedly (ungracefully)**, publish this Will on my behalf to everyone!”

### 📊 Will Lifecycle Diagram (Birth + Last Will)

```
 🌡️ Industrial Sensor (Client A)             🟠 Broker                         🖥️ Monitoring Dashboard (Client B)
 ═════════════════════════                 ═════════════════                 ════════════════════════════════
           │                                       │                                        │
           │                                       │      1. SUBSCRIBE (sensor1/status)     │
           │                                       │ ◄━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
           │                                       │                                        │
 ╭─────────┴───────────────────────────────────────┴────────────────────────────────────────┴─────────╮
 │ 🟢 Phase One: Initial connection and registering the Will (LWT) with the broker                                           │
 │                                                                                                    │
 │         │ 2. CONNECT                            │                                        │         │
 │         │    • Will Topic: "sensor1/status"     │                                        │         │
 │         │    • Will Payload: "OFFLINE"          │                                        │         │
 │         │    • Will Retain: True                │                                        │         │
 │         │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │ 📝 [Store Will in broker memory]      │         │
 │         │ ◄╌╌╌╌╌╌╌╌╌╌╌╌ CONNACK ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌ │                                        │         │
 ╰─────────┬───────────────────────────────────────┬────────────────────────────────────────┬─────────╯
           │                                       │                                        │
 ╭─────────┴───────────────────────────────────────┴────────────────────────────────────────┴─────────╮
 │ 🔵 Phase Two: The sensor announces that it is online (Birth Message)                                        │
 │                                                                                                    │
 │         │ 3. PUBLISH ("sensor1/status" = ONLINE)│                                        │         │
 │         │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │ ─── PUBLISH ("ONLINE") ──────────────► │ 🟢      │
 ╰─────────┬───────────────────────────────────────┬────────────────────────────────────────┬─────────╯
           │                                       │                                        │
 ╭─────────┴───────────────────────────────────────┴────────────────────────────────────────┴─────────╮
 │ 🔴 Phase Three: Sudden sensor power failure and automatic Will publication                                          │
 │                                                                                                    │
 │    ⚡ [Power failure!]                                │                                        │         │
 │         │ ✕ ┄┄┄┄ (connection lost; no ping sent)  │                                        │         │
 │                                                 │ ⏱️ [Keep-Alive timer expires]       │         │
 │                                                 │ ⚠️ [Client detected offline; publish LWT]     │         │
 │                                                 │ ─── PUBLISH ("OFFLINE") ─────────────► │ 🔴      │
 ╰─────────────────────────────────────────────────┴────────────────────────────────────────┴─────────╯
```

> [!TIP]
> If a client sends a normal, graceful `DISCONNECT` packet, the broker **removes the Will and does not publish it** (except in MQTT v5, where a specific reason code can request publication).

---
