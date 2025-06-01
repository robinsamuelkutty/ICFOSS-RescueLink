# 🌐 LoRaWAN Network Setup Guide (Personal Note)

This is my personal note for setting up a **LoRaWAN network on Ubuntu 22.04** using **ChirpStack**, with **InfluxDB** for data storage and **Grafana** for visualization. It covers LoRaWAN basics, components, pros/cons, security, and step-by-step setup instructions.

---

## 🌐 What is LoRaWAN?

**LoRaWAN** (Long Range Wide Area Network) is a Low Power, Wide Area Network (LPWAN) protocol designed for wireless communication between IoT devices over long distances with minimal power usage. It uses LoRa (Long Range) modulation and operates in unlicensed radio frequency bands (e.g., **868 MHz** in Europe, **915 MHz** in the US).

---

## 🔧 Key Components of LoRaWAN

- **End Devices (Nodes)** – Sensors, meters, or devices like IMUs (e.g., MPU6050).
- **Gateways** – Relay data between nodes and the network server (e.g., Dragino LG308).
- **Network Server** – Manages the network (e.g., ChirpStack).
- **Application Server** – Processes and visualizes data (e.g., Grafana dashboards).

---

## ✅ Advantages of LoRaWAN

| Feature                  | Benefit                                         |
|--------------------------|--------------------------------------------------|
| 📡 Long Range             | Up to 15–20 km in rural areas, 2–5 km in urban   |
| 🔋 Low Power              | Battery life up to 10+ years for nodes           |
| 🌍 Low Cost               | No SIM cards; uses unlicensed ISM bands          |
| 📶 Scalability            | Supports thousands of devices per gateway        |
| 🔐 Secure                 | AES-128 encryption for data & network            |
| 🏭 Ideal for IoT          | Great for smart agriculture, cities, metering    |
| 🌐 Bidirectional Communication | Supports commands from server to devices |

---

## 🔻 Disadvantages of LoRaWAN

- **Low Data Rate**: Designed for small, infrequent packets. Not suitable for video or large file transfers.
- **Limited Payload Size**: 51–222 bytes depending on data rate.
- **Latency**: Not real-time, unsuitable for live tracking or emergency systems.
- **Duty Cycle Limitations**: 1% duty cycle in Europe limits message frequency.
- **Unlicensed Bands**: Prone to interference in ISM bands (868 MHz EU, 915 MHz US).
- **Security Risks**: AES-128 encryption, but poor key management can lead to replay attacks or eavesdropping.
- **Scalability Challenges**: Packet collisions in dense deployments require careful planning.
- **Limited Mobility Support**: Better for fixed or slow-moving devices.
- **Downlink Constraints**: Limited to maintain power efficiency.
- **Gateway Dependency**: Requires gateway infrastructure for coverage.

---

## 📌 Use Cases

- **Smart Agriculture**: Soil and weather sensors.
- **Smart Cities**: Streetlights, parking systems.
- **Industrial Monitoring**: Machine vibration (e.g., IMU data).
- **Environmental Monitoring**: Air and water quality.
- **Utility Metering**: Electricity, gas, water.

---

## 🔹 LoRaWAN Device Classes

| Feature            | Class A                     | Class B                      | Class C                      |
|--------------------|-----------------------------|------------------------------|------------------------------|
| Receive Downlink   | After uplink only           | After uplink + Scheduled slots | Anytime (except when transmitting) |
| Power Usage        | Very low                    | Medium                       | High                         |
| Use Case           | Sensors, meters             | Tracking, alarms             | Actuators, controllers       |
| Latency            | High                        | Medium                       | Low                          |

- **Class A**: Lowest power, uplink-initiated (e.g., temperature sensors).
- **Class B**: Scheduled receive slots for semi-frequent downlinks.
- **Class C**: Always listening, high power, for mains-powered devices.

---

## 📡 Maximum Data Rate in LoRaWAN

| Region | Bandwidth | Spreading Factor (SF) | Max Data Rate            |
|--------|-----------|------------------------|---------------------------|
| EU868  | 125 kHz   | SF7                    | ~5.47 kbps                |
| US915  | 125/500 kHz | SF7                  | ~21.87 kbps (500 kHz)     |

- **Spreading Factor (SF)**: SF7 (fastest, short range) to SF12 (slowest, long range).
- **Realistic Rates (125 kHz)**: SF7 (5.47 kbps) to SF12 (0.29 kbps).
- **Not Suitable For**: Video, voice, or large file transfers.

---

## 🔐 LoRaWAN Security Overview

### Security Layers

- **Network Layer**: Uses `NwkSKey` for device authenticity and MAC commands.
- **Application Layer**: Uses `AppSKey` for end-to-end payload encryption.

### Key Types

| Key     | Purpose                        | Visibility                        |
|---------|--------------------------------|-----------------------------------|
| AppKey  | Used during OTAA activation    | Device and join server only       |
| NwkSKey | Secures communication          | Shared with network server        |
| AppSKey | Secures application data       | Shared with application server    |

### Activation Methods

- **OTAA (Over-The-Air Activation)**: Dynamic keys, secure, recommended.
- **ABP (Activation By Personalization)**: Static keys, less secure, manual setup.

### Common Security Risks

| Risk           | Description                  | Mitigation                        |
|----------------|------------------------------|------------------------------------|
| Replay Attacks | Reusing old messages         | Use frame counters                 |
| Key Leakage    | Poor key storage             | Use secure elements                |
| Poor Key Management | Reusing or exposing keys | Unique keys per device             |
| Tampering      | Physical access to secrets   | Tamper-proof enclosures            |
| Weak ABP Security | Static keys prone to reuse | Prefer OTAA                        |

### Best Practices

- Use OTAA for secure key generation.
- Store keys in secure elements (e.g., ATECC608A).
- Update firmware regularly.
- Use unique keys per device and validate frame counters.

---

## 🔧 LoRaWAN Network Setup on Ubuntu 22.04

### Prerequisites

- Ubuntu 22.04 server (or VM)
- Root access (user: `root`, pass: `[your password]` or `dragino`)
- Internet connection
- LoRaWAN gateway (e.g., Dragino LPS8, LG308)
- LoRaWAN end device (e.g., with MPU6050 IMU)
- Optional: Secure element for key storage

---
