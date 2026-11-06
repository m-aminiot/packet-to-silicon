# 📡 MQTT — Part 1: Basic | Core Concepts and Hands-on Workshop

# 🟢 Level One: Basic — Core Concepts and Architecture

In this section, we learn about the design philosophy of MQTT, its place in the network model, and how messages are sent and received.

## 1. What Is MQTT, and Which Network Layer Does It Use?

The MQTT protocol was designed in 1999 by IBM engineers to monitor oil pipelines over satellite communications, which had low bandwidth and high latency.

In the **OSI** reference model, MQTT is in **Layer 7 (the Application Layer)**, alongside protocols such as HTTP, FTP, and WebSocket.

### 📊 MQTT’s Place in the OSI and TCP/IP Network Layers

```
╭──────────────────────────────────────────────────────────────────────────╮
│ 🟩 Layer 7 (Application Layer) ──► [ 📡 MQTT Protocol ]                   │
│    • Message structure, Topic management, QoS levels, Retain, and Last Will│
├──────────────────────────────────────────────────────────────────────────┤
│ 🟦 Layer 4 (Transport Layer) ──► [ 🔒 TCP / TLS (ports 1883 / 8883) ]    │
│    • Ordered delivery, reliable connections, or UDP / QUIC in certain cases│
├──────────────────────────────────────────────────────────────────────────┤
│ 🟨 Layer 3 (Network Layer) ──► [ 🌐 IP (IPv4 / IPv6) ]                   │
│    • Routing and logical network packet addressing                        │
├──────────────────────────────────────────────────────────────────────────┤
│ 🟪 Layers 1 and 2 (Physical/Link) ──► [ 📶 Wi-Fi | Ethernet | 4G/5G ]   │
│    • Physical transmission of bits over wired or radio media              │
╰──────────────────────────────────────────────────────────────────────────╯
```

### Why Does It Run over TCP by Default?

For its standard operation, MQTT runs over **TCP/IP** (Layer 4, or the Transport layer). The main reasons for using TCP are:

1. **Connection-oriented:** A stable connection is established between the client and server before data is exchanged.
2. **Ordered delivery:** Packets are received in the same order in which they were sent.
3. **Error checking:** The TCP layer handles packet corruption during transmission across the network.

> [!NOTE]
> **Standard MQTT ports:**
> * 🔓 Port `1883`: Standard, unencrypted MQTT over TCP
> * 🔐 Port `8883`: Secure MQTT over TCP (using TLS/SSL)

---

## 2. Client–Broker Architecture and the Publish/Subscribe Pattern

Unlike HTTP, which uses a direct **Request/Response** architecture between a client and server, MQTT is based on the **Publish/Subscribe** pattern and **Client–Broker** architecture.

In this architecture, the sender and receiver do not know each other directly (for example, they do not need each other’s IP address or port). All communication is managed through a central component called the **Broker**.

### 📊 Publish/Subscribe Architecture Diagram

```
 🟢 Publishers                         🟠 Central Broker                  🔵 Subscribers
 ═══════════════════════             ═══════════════════════            ════════════════════════

╭────────────────────────╮                                             ╭────────────────────────╮
│ 🌡️ Temperature sensor (Client A) │ ── Publish ─╮   ╭── Forward ─► │ 📱 Mobile app (Client C) │
╰────────────────────────╯              │               │              ╰────────────────────────╯
       (Topic: home/temp)               ▼               │                  (Sub: home/#)
                               ╭─────────────────╮      │
                               │  🔄 MQTT Broker │ ─────┤
                               │ (EMQX/Mosquitto)│      │
       (Topic: home/door)               ▲               │                  (Sub: home/#)
╭────────────────────────╮              │               │              ╭────────────────────────╮
│ 🚪 Door sensor (Client B) │ ── Publish ─╯     ╰── Forward ─► │ 🗄️ Database (Client D) │
╰────────────────────────╯                                             ╰────────────────────────╯
```

### Main Components of This Architecture:

1. **🟢 MQTT Client:** Any device—from a small microcontroller such as an ESP32 to a powerful Linux server or mobile app—that runs an MQTT library and connects to a broker is a client. Every client can be both a **Publisher** and a **Subscriber** at the same time.
2. **🟠 MQTT Broker:** The heart of the network. The broker’s responsibilities include:
   * Accepting and managing thousands or millions of simultaneous client connections.
   * Receiving messages from Publishers.
   * Filtering messages by topic and forwarding them **only** to Subscribers interested in that topic.
   * Managing security, authentication, and client state.
3. **📤 Publish operation:** A client sends a message to the broker on a specified topic.
4. **📥 Subscribe operation:** A client tells the broker: “Whenever a message arrives on topic $X$, send me a copy as well.”

### Benefits of Decoupling in the Pub/Sub Pattern:

* **🌍 Space decoupling:** The sender and receiver do not need to know each other’s IP address.
* **⏱️ Time decoupling:** The sender and receiver do not have to be online at exactly the same time (the broker can keep a message).
* **⚡ Synchronization decoupling:** Sending and receiving are asynchronous, so the client application is not blocked.

---

## 3. Topics and Their Hierarchical Structure

In MQTT, messages are not sent directly to client addresses. Instead, they are published to text addresses called **Topics**. Topics are the broker’s message-routing channels.

### Topic Characteristics:

* A UTF-8 encoded text string.
* Topics have a hierarchical (tree-like) structure, with levels separated by a slash (`/`).
* Topics are **case-sensitive**: `home/temp` and `Home/Temp` are two completely different topics.
* Topics do not need to be created or defined in advance! A broker handles them dynamically when a client publishes or subscribes to one.

### 🌳 Topic Tree Diagram for a Smart Home

```
🏠 home
 ├── 🛋️ livingroom
 │    ├── 🌡️ temperature   ──►  Full topic: home/livingroom/temperature
 │    └── 💡 light         ──►  Full topic: home/livingroom/light
 │
 └── 🍳 kitchen
      ├── 🌡️ temperature   ──►  Full topic: home/kitchen/temperature
      └── 🔥 smoke         ──►  Full topic: home/kitchen/smoke
```

### Wildcards in Subscriptions:

If a client wants to listen to a set of topics instead of subscribing to each one individually, it uses **wildcards**:

> [!IMPORTANT]
> Wildcards are allowed **only when subscribing**. When publishing, the message must be sent to one exact topic.

#### 1. Single-Level Wildcard: the `+` symbol
The `+` symbol replaces **exactly one level** in the topic structure.
* **Example:** Subscribe to `home/+/temperature`
* 🟢 **Matches:**
  * `home/livingroom/temperature`
  * `home/kitchen/temperature`
* 🔴 **Does not match:**
  * `home/livingroom/light`
  * `home/floor1/kitchen/temperature` (because there are two levels between `home` and `temperature`).

#### 2. Multi-Level Wildcard: the `#` symbol
The `#` symbol matches **zero, one, or all subsequent levels** and must always appear at the **end of the topic filter**.
* **Example:** Subscribe to `home/kitchen/#`
* 🟢 **Matches:**
  * `home/kitchen/temperature`
  * `home/kitchen/smoke`
  * `home/kitchen/fridge/freezer/temp`

---

## 4. Quality of Service Levels (QoS — Quality of Service)

One of MQTT’s most powerful features is its ability to specify a **message delivery guarantee**, called **QoS**. Since IoT networks can be unstable, MQTT provides three delivery levels.

| QoS level | Standard name | Delivery guarantee | Packet exchange count | Main use case |
| :---: | :---: | :---: | :---: | :--- |
| **🟢 QoS 0** | At Most Once | At most once (no guarantee) | 1 packet | Frequently updated data such as live temperature |
| **🟡 QoS 1** | At Least Once | At least once (duplicates possible) | 2 packets | Logs, alerts, and sensor status |
| **🔵 QoS 2** | Exactly Once | Exactly once (no duplicates) | 4 packets | Financial transactions and sensitive industrial commands |

> [!TIP]
> QoS is always negotiated **between a client and the broker** (not directly end-to-end from the sender to the final receiver). If the sender publishes with `QoS 2` but the receiver subscribes to that topic with `QoS 0`, the message is delivered to that receiver at `QoS 0` (the lowest QoS in the chain applies to the receiver).

---

### 1. 🟢 QoS 0: At Most Once — commonly called *Fire and Forget*

In this mode, the sender transmits the message, does not wait for an acknowledgement from the receiver, and does not store the message in memory.

```
 🟢 Sender (Publisher)                                     🟠 Broker
 ══════════════════════                                    ═════════════════
           │                                                       │
           │  1. 📤 PUBLISH (QoS = 0)                              │
           │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │
           │                                                       │ ──► Forward to subscribers
   [Immediately discarded from memory]                                             │     (no response sent)
           │                                                       │
```

---

### 2. 🟡 QoS 1: At Least Once

At this level, the message is guaranteed to reach the receiver, but it may be received **more than once (duplicated)** if the network is slow. The sender keeps the message in its buffer until it receives the **`PUBACK`** acknowledgement.

```
 🟡 Sender (Publisher)                                    🟠 Broker
 ══════════════════════                                    ═════════════════
   [Store message in buffer]                                            │
           │                                                       │
           │  1. 📤 PUBLISH (Packet ID = 10, QoS = 1)              │
           │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │
           │                                                       │ ──► Process and forward to subscribers
           │  2. 📥 PUBACK (Packet ID = 10)                        │
           │ ◄╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌ │
           │                                                       │
   [Remove message from buffer]                                              │
```

---

### 3. 🔵 QoS 2: Exactly Once

This is the highest and most reliable QoS level. It is described as guaranteeing delivery **without loss** and **without any duplication**, exactly once. This is performed using a four-step negotiation (**4-Way Handshake**):

```
 🔵 Sender (Publisher)                                    🟠 Broker
 ══════════════════════                                    ═════════════════
   [Store message + lock ID]                                           │
           │                                                       │
           │  1. 📤 PUBLISH (Packet ID = 42, QoS = 2)              │
           │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │
           │                                                       │ [Register Packet ID 42]
           │  2. 📥 PUBREC (Publish Received | ID = 42)            │ [Forward message to subscribers]
           │ ◄╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌ │
           │                                                       │
   [Delete message content; retain ID]                                          │
           │                                                       │
           │  3. 📤 PUBREL (Publish Release | ID = 42)             │
           │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━► │
           │                                                       │ [Release Packet ID 42]
           │  4. 📥 PUBCOMP (Publish Complete | ID = 42)           │
           │ ◄╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌ │
           │                                                       │
   [Transaction fully complete ✅]                                          │
```

---


---

# 🛠️ From Theory to Practice: Mosquitto Setup and Testing Workshop

Now it is time to test all the theory covered so far (QoS, topics, Retain, LWT, and more) in a real operating-system environment. The **Eclipse Mosquitto** broker is an excellent tool for local setup and engineering experiments because it is very small (less than a few megabytes), fast, and supports MQTT v3.1.1 and MQTT v5.0.

---

## 1. Installing the Mosquitto Broker on Different Operating Systems

### 🐧 1. Ubuntu and Debian (Ubuntu / Debian / Raspberry Pi OS)

```bash
# 1. Update package repositories
sudo apt update

# 2. Install the broker service
sudo apt install -y mosquitto

# 3. Enable the service to start automatically with systemd
sudo systemctl enable mosquitto
sudo systemctl start mosquitto

# 4. Check the service status
sudo systemctl status mosquitto
```

> [!NOTE]
> In newer Mosquitto versions (version 2 and later), for security the broker listens only on `localhost` (127.0.0.1) by default, and anonymous (password-free) access from external networks is disabled. To connect from other systems on the network, open the configuration file:
> ```bash
> sudo nano /etc/mosquitto/conf.d/default.conf
> ```
> Add these two lines and save the file:
> ```text
> listener 1883
> allow_anonymous true
> ```
> Then restart the service: `sudo systemctl restart mosquitto`

---

### 🔴 2. Red Hat, CentOS, and Fedora (RHEL / CentOS / Fedora)

```bash
# On Fedora:
sudo dnf install -y mosquitto

# On CentOS / RHEL (EPEL repository required):
sudo dnf install -y epel-release
sudo dnf install -y mosquitto

# Enable the service
sudo systemctl enable --now mosquitto
```

---

### 🍏 3. macOS (with Homebrew)

If you use the Homebrew package manager:

```bash
# Install the broker
brew install mosquitto

# Run the broker as a background service
brew services start mosquitto

# Or run the broker manually with the default configuration file
/usr/local/sbin/mosquitto -c /usr/local/etc/mosquitto/mosquitto.conf
# (On Apple Silicon Macs, the path is /opt/homebrew/sbin/mosquitto)
```

---

### 🪟 4. Windows operating system

There are three simple options for Windows:

1. **Use the installer (MSI Installer):**
   * Go to the official download page at [mosquitto.org/download](https://mosquitto.org/download/).
   * Download and install the 64-bit Windows installer (for example, `mosquitto-x.x.x-install-windows-x64.exe`).
   * It is registered and runs as a Windows Service, visible in `Services.msc`.
2. **Use the Winget or Chocolatey package manager:**
   ```powershell
   # Using Winget:
   winget install EclipseFoundation.Mosquitto
   
   # Or using Chocolatey:
   choco install mosquitto
   ```
3. The default executable path is usually `C:\Program Files\mosquitto`; adding it to the Windows `PATH` environment variable is recommended.

---

### 🐳 5. Run Instantly with Docker (Docker & Docker Compose)

The fastest way to bring up a ready-to-use isolated environment:

**Run with a single command:**
```bash
docker run -d \
  --name mosquitto-broker \
  -p 1883:1883 \
  -p 9001:9001 \
  eclipse-mosquitto:latest
```

**Or use `docker-compose.yml`:**
```yaml
version: '3.8'
services:
  mqtt-broker:
    image: eclipse-mosquitto:latest
    container_name: mosquitto_server
    restart: always
    ports:
      - "1883:1883"      # Standard MQTT port
      - "9001:9001"      # WebSocket port
    volumes:
      - ./mosquitto.conf:/mosquitto/config/mosquitto.conf
      - mosquitto_data:/mosquitto/data
      - mosquitto_log:/mosquitto/log

volumes:
  mosquitto_data:
  mosquitto_log:
```

---

## 2. Installing the Command-Line Tools (mosquitto_sub and mosquitto_pub)

Two CLI tools let you simulate MQTT clients without writing code:
* **`mosquitto_sub`:** A client for subscribing to topics and listening for messages.
* **`mosquitto_pub`:** A client for publishing a message to a specified topic.

```
 ┌──────────────────────┐                    ┌──────────────────────┐
 │   mosquitto_sub      │                    │   mosquitto_pub      │
 │ (Listener / Subscriber)│                   │  (Message publisher) │
 └──────────┬───────────┘                    └──────────┬───────────┘
            │                                           │
            │ 1. SUBSCRIBE -t "sensors/#"               │ 2. PUBLISH -t "sensors/temp" -m "24.5"
            ▼                                           ▼
      ┌───────────────────────────────────────────────────────┐
      │               🔄 Mosquitto MQTT Broker                │
      │                  (localhost:1883)                     │
      └───────────────────────────────────────────────────────┘
            │
            │ 3. Forward Message -> "24.5"
            ▼
 ┌──────────────────────┐
 │ mosquitto_sub terminal│
 └──────────────────────┘
```

### Installing the Client Tools:

* **🐧 On Ubuntu / Debian:**
  ```bash
  sudo apt install -y mosquitto-clients
  ```
* **🔴 On CentOS / Fedora:**
  ```bash
  sudo dnf install -y mosquitto
  # Client tools are installed with the main package
  ```
* **🍏 On macOS:**
  ```bash
  # These tools are also installed when you run brew install mosquitto.
  ```
* **🪟 On Windows:**
  * The `mosquitto_sub.exe` and `mosquitto_pub.exe` files are available in the installation directory, `C:\Program Files\mosquitto`.

---

## 3. Key CLI Options and Parameters Explained

These options are used by both `mosquitto_sub` and `mosquitto_pub`:

| Short option | Equivalent long option | Description |
| :---: | :--- | :--- |
| **`-h`** | `--host` | Broker IP address or hostname (default: `localhost`) |
| **`-p`** | `--port` | Broker connection port (default: `1883`) |
| **`-t`** | `--topic` | Topic to publish or subscribe to (required) |
| **`-m`** | `--message` | Message text or payload (for `mosquitto_pub`) |
| **`-q`** | `--qos` | QoS level: `0`, `1`, or `2` (default: `0`) |
| **`-r`** | `--retain` | Publish with the Retain flag (for `mosquitto_pub`) |
| **`-v`** | `--verbose` | Show the topic name alongside each received message (for `mosquitto_sub`) |
| **`-i`** | `--id` | Set a unique client identifier (`client-id`) |
| **`-u` / `-P`**| `--username` / `--pw` | Username and password for authentication |
| **`-c`** | `--disable-clean-session` | Disable Clean Session (keep a persistent session) |
| **`-C`** | `--count` | Receive a specified number of messages, then exit (for `mosquitto_sub`) |
| **`-V`** | `--protocol-version` | Choose the protocol version: `mqttv311` (default) or `mqttv5` |
| **`--will-topic`**| `--will-topic` | Topic for the Last Will message (LWT) on an unexpected disconnect |
| **`--will-payload`**| `--will-payload`| Client Last Will message payload |

---

## 4. Step-by-Step Hands-on Labs

For the complete exercises, open two terminal windows side by side.

### Lab 1: First Simple Test (Local Pub/Sub)

**Terminal 1 (listener):**
```bash
# Listen to the smart-home topic on the local broker
mosquitto_sub -h localhost -p 1883 -t "home/livingroom/temp" -v
```

**Terminal 2 (publisher):**
```bash
# Publish a temperature of 26 degrees
mosquitto_pub -h localhost -p 1883 -t "home/livingroom/temp" -m "26.0 C"
```
*Result:* The text `home/livingroom/temp 26.0 C` is printed immediately in Terminal 1.

---

### Lab 2: Test Delivery Assurance with QoS 2

In this lab, a message is sent using the four-step delivery handshake:

**Terminal 1 (Subscriber with QoS 2):**
```bash
mosquitto_sub -h localhost -t "factory/alarm" -q 2 -v
```

**Terminal 2 (Publisher with QoS 2 and protocol logging enabled with -d):**
```bash
mosquitto_pub -h localhost -t "factory/alarm" -m "OVERHEAT_EMERGENCY" -q 2 -d
```

> [!TIP]
> With the `-d` switch (Debug Mode), the packet-exchange log appears in the output:
> `Client ... sending PUBLISH (d0, q2, r0, m1, ...)`  
> `Client ... received PUBREC`  
> `Client ... sending PUBREL`  
> `Client ... received PUBCOMP`  
> This is exactly the four-step QoS 2 exchange covered in the theory section!

---

### Lab 3: Test a Retained Message

We will test Retained messages: first publish the message, and **then** connect the client!

**Step 1: Publish a retained message (before anyone is listening!):**
```bash
mosquitto_pub -h localhost -t "device/status" -m "MACHINE_READY" -r
```

**Step 2: Open the subscriber terminal (a few seconds or minutes later):**
```bash
mosquitto_sub -h localhost -t "device/status" -v
```
*Surprising result:* As soon as the command runs, the client immediately receives the latest state:
`device/status MACHINE_READY`

**Step 3: Clear the retained message from the broker’s memory:**
```bash
# Publishing an empty message with the -r flag clears it from the broker
mosquitto_pub -h localhost -t "device/status" -r -m ""
```

---

### Lab 4: Simulate a Sudden Failure and Last Will (LWT)

In this lab, a client configures a Last Will, then its connection is terminated abruptly (killed) so the broker publishes the Will.

**Terminal 1 (sensor-status monitoring dashboard):**
```bash
mosquitto_sub -h localhost -t "nodes/sensor1/status" -v
```

**Terminal 2 (sensor connecting with a Last Will):**
```bash
# Keep the sensor connected indefinitely with a 10-second Keep-Alive timer
mosquitto_sub -h localhost -t "dummy/topic" \
  -i "SensorNode_01" \
  -k 10 \
  --will-topic "nodes/sensor1/status" \
  --will-payload "LOST_CONNECTION_ALARM" \
  --will-qos 1
```

**Step 3 (abruptly terminate the sensor process):**
In Terminal 2, press `Ctrl + C` or kill the process (`kill -9`) to close the socket without sending a `DISCONNECT` packet.

*Result:* After the broker’s Keep-Alive timeout expires, the following message is published in Terminal 1:
`nodes/sensor1/status LOST_CONNECTION_ALARM`

---

### Lab 5: Publishing and Formatting Structured Data (JSON Payload)

In real-world systems, IoT data is often sent as JSON strings.

**Terminal 1 (subscribe to all weather sensors using a wildcard):**
```bash
mosquitto_sub -h localhost -t "weather/+/telemetry" -v
```

**Terminal 2 (publish structured data from station 1 sensor):**
```bash
mosquitto_pub -h localhost -t "weather/station_tehran/telemetry" -m '{
  "station_id": "THR-01",
  "temperature": 23.4,
  "humidity": 41,
  "pressure": 1013.25,
  "timestamp": 1714470000
}'
```

---

### Lab 6: Connect to and Test a Public Online Broker (No Local Server Required)

If you do not currently have a broker installed, you can use free public test brokers over the Internet:

**Listen on your unique topic on the public EMQX broker:**
```bash
mosquitto_sub -h broker.emqx.io -p 1883 -t "my_unique_project_2026/#" -v
```

**Publish to the public broker:**
```bash
mosquitto_pub -h broker.emqx.io -p 1883 -t "my_unique_project_2026/test" -m "Hello MQTT from Terminal!"
```

---
*Prepared as a free Persian reference for the MQTT protocol. If you find it useful, support this repository by giving it a ⭐ (Star)!*
