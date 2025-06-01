LoRaWAN Network Setup Guide (Personal Note)
This is my personal note for setting up a LoRaWAN network on Ubuntu 22.04 using ChirpStack, with InfluxDB for data storage and Grafana for visualization. It covers LoRaWAN basics, components, pros/cons, security, and step-by-step setup instructions.

🌐 What is LoRaWAN?
LoRaWAN (Long Range Wide Area Network) is a Low Power, Wide Area Network (LPWAN) protocol designed for wireless communication between IoT devices over long distances with minimal power usage. It uses LoRa (Long Range) modulation and operates in unlicensed radio frequency bands (e.g., 868 MHz in Europe, 915 MHz in the US).

🔧 Key Components of LoRaWAN

End Devices (Nodes) – Sensors, meters, or devices like IMUs (e.g., MPU6050).
Gateways – Relay data between nodes and the network server (e.g., Dragino LG308).
Network Server – Manages the network (e.g., ChirpStack).
Application Server – Processes and visualizes data (e.g., Grafana dashboards).


✅ Advantages of LoRaWAN



Feature
Benefit



📡 Long Range
Up to 15–20 km in rural areas, 2–5 km in urban


🔋 Low Power
Battery life up to 10+ years for nodes


🌍 Low Cost
No SIM cards; uses unlicensed ISM bands


📶 Scalability
Supports thousands of devices per gateway


🔐 Secure
AES-128 encryption for data & network


🏭 Ideal for IoT
Great for smart agriculture, cities, metering, etc.


🌐 Bidirectional Communication
Supports commands from server to devices



🔻 Disadvantages of LoRaWAN

Low Data Rate  
Designed for small, infrequent packets. Not suitable for video or large file transfers.


Limited Payload Size  
51–222 bytes depending on data rate.


Latency  
Not real-time, unsuitable for live tracking or emergency systems.


Duty Cycle Limitations  
1% duty cycle in Europe limits message frequency.


Unlicensed Bands  
Prone to interference in ISM bands (868 MHz EU, 915 MHz US).


Security Risks  
AES-128 encryption, but poor key management can lead to replay attacks or eavesdropping.


Scalability Challenges  
Packet collisions in dense deployments require careful planning.


Limited Mobility Support  
Better for fixed or slow-moving devices.


Downlink Constraints  
Limited to maintain power efficiency.


Gateway Dependency  
Requires gateway infrastructure for coverage.




📌 Use Cases

Smart Agriculture: Soil and weather sensors.
Smart Cities: Streetlights, parking systems.
Industrial Monitoring: Machine vibration (e.g., IMU data).
Environmental Monitoring: Air and water quality.
Utility Metering: Electricity, gas, water.


🔹 LoRaWAN Device Classes



Feature
Class A
Class B
Class C



Receive Downlink
After uplink only
After uplink + Scheduled slots
Anytime (except when transmitting)


Power Usage
Very low
Medium
High


Use Case
Sensors, meters
Tracking, alarms
Actuators, controllers


Latency
High
Medium
Low



Class A: Lowest power, uplink-initiated (e.g., temperature sensors).
Class B: Scheduled receive slots for semi-frequent downlinks.
Class C: Always listening, high power, for mains-powered devices.


📡 Maximum Data Rate in LoRaWAN



Region
Bandwidth
Spreading Factor (SF)
Max Data Rate



EU868
125 kHz
SF7
~5.47 kbps


US915
125 kHz / 500 kHz (SF7)
SF7
~21.87 kbps (500 kHz)



Spreading Factor (SF): SF7 (fastest, short range) to SF12 (slowest, long range).
Realistic Rates (125 kHz): SF7 (5.47 kbps) to SF12 (0.29 kbps).
Not Suitable For: Video, voice, or large file transfers.


🔐 LoRaWAN Security Overview
Security Layers

Network Layer: Uses NwkSKey for device authenticity and MAC commands.
Application Layer: Uses AppSKey for end-to-end payload encryption.

Key Types



Key
Purpose
Visibility



AppKey
Used during OTAA activation
Device and join server only


NwkSKey
Secures communication with network server
Shared with network server


AppSKey
Secures application data
Shared with application server only


Activation Methods

OTAA (Over-The-Air Activation): Dynamic keys, secure, recommended.
ABP (Activation By Personalization): Static keys, less secure, manual setup.

Common Security Risks



Risk
Description
Mitigation



Replay Attacks
Reusing old messages
Use frame counters


Key Leakage
Poor key storage
Use secure elements


Poor Key Management
Reusing or exposing keys
Unique keys per device


Tampering
Physical access to secrets
Tamper-proof enclosures


Weak ABP Security
Static keys prone to reuse
Prefer OTAA


Best Practices

Use OTAA for secure key generation.
Store keys in secure elements (e.g., ATECC608A).
Update firmware regularly.
Use unique keys per device and validate frame counters.


🔧 LoRaWAN Network Setup on Ubuntu 22.04
Prerequisites

Ubuntu 22.04 server (or VM).
Root access (user: root, pass: [your password] or dragino for Dragino gateways).
Internet connection.
LoRaWAN gateway (e.g., Dragino LPS8, LG308).
LoRaWAN end device (e.g., with MPU6050 IMU for motion data).
Optional: Secure element for key storage.

Step-by-Step Instructions
1. Set Static IP for Gateway/Server
Assign 10.130.1.1 to your Ubuntu server or gateway.
sudo nano /etc/netplan/01-netcfg.yaml

Add:
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses: [10.130.1.1/24]
      gateway4: 10.130.1.254
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]

Apply:
sudo netplan apply

Verify:
ip addr

2. Install ChirpStack
ChirpStack is an open-source LoRaWAN network server stack.

Add ChirpStack repository:

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00
echo "deb https://artifacts.chirpstack.io/packages/4.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list
sudo apt update


Install ChirpStack components:

sudo apt install chirpstack-gateway-bridge chirpstack-network-server chirpstack-application-server

3. Configure ChirpStack
Edit configuration files:
sudo nano /etc/chirpstack-network-server/chirpstack-network-server.toml

Set gateway and MQTT settings:
[network_server.network_settings]
  enabled_regions=["eu868"]

[integration.mqtt]
  server="tcp://10.130.1.1:1883"

For EU868 frequencies (from your note):

865.0625 MHz
865.4025 MHz
865.985 MHz

Add to [network_server.network_settings]:
  [[network_server.network_settings.uplink_channels]]
    frequency=865062500
    min_dr=0
    max_dr=5

  [[network_server.network_settings.uplink_channels]]
    frequency=865402500
    min_dr=0
    max_dr=5

  [[network_server.network_settings.uplink_channels]]
    frequency=865985000
    min_dr=0
    max_dr=5

4. Set Up MQTT Broker (Mosquitto)
Install Mosquitto:
sudo apt install mosquitto mosquitto-clients

Start and enable:
sudo systemctl enable mosquitto
sudo systemctl start mosquitto

5. Install InfluxDB
Install InfluxDB for time-series data storage:
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
echo "deb https://repos.influxdata.com/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
sudo apt update
sudo apt install influxdb2

Start InfluxDB:
sudo systemctl enable influxdb
sudo systemctl start influxdb

Create a database:
influx
> CREATE DATABASE lorawan_data

6. Install Telegraf
Telegraf bridges MQTT to InfluxDB:
sudo apt install telegraf

Configure Telegraf:
sudo nano /etc/telegraf/telegraf.conf

Add:
[[inputs.mqtt_consumer]]
  servers = ["tcp://10.130.1.1:1883"]
  topics = ["application/+/device/+/event/up"]
  data_format = "json"

[[outputs.influxdb]]
  urls = ["http://10.130.1.1:8086"]
  database = "lorawan_data"

Restart Telegraf:
sudo systemctl restart telegraf

7. Install Grafana
Install Grafana for visualization:
sudo apt-get install -y apt-transport-https
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana

Start Grafana:
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

Access Grafana at http://10.130.1.1:3000 (default login: admin/admin).
8. Configure Grafana

Add InfluxDB as a data source:
URL: http://10.130.1.1:8086
Database: lorawan_data


Create dashboards for sensor data (e.g., temperature, humidity, IMU motion).

9. Configure LoRaWAN Device (OTAA Recommended)
For a device (e.g., with MPU6050 IMU):
uint8_t joinEUI[8]  = { 0x70, 0xB3, 0xD5, 0x7E, 0xF0, 0x00, 0x00, 0x00 };
uint8_t devEUI[8]   = { 0x00, 0x04, 0xA3, 0xB2, 0xC3, 0xD4, 0xE5, 0xF6 };
uint8_t appKey[16]  = { 0xA1, 0xA2, 0xA3, 0xA4, 0xA5, 0xA6, 0xA7, 0xA8, 0xA9, 0xAA, 0xAB, 0xAC, 0xAD, 0xAE, 0xAF, 0xB0 };

ChipStack_SetDevEUI(devEUI);
ChipStack_SetJoinEUI(joinEUI);
ChipStack_SetAppKey(appKey);
ChipStack_JoinRequest();

Register the device in ChirpStack’s web interface (http://10.130.1.1:8080).
10. Verify Gateway LEDs
For Dragino gateways:

PWR: Solid ON = Powered.
WAN: Blinking = Network active.
LoRa: Blinking = Sending/receiving packets.
SYS: Check for errors (refer to manual for blink patterns).

Gateway EUI is found in the web interface or device label.

🔒 Security Notes

Use OTAA for secure device activation.
Store keys in a secure element (e.g., ATECC608A).
Disable password-based SSH login:sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart sshd


Use firewall rules:sudo ufw allow 22,1883,8080,3000
sudo ufw enable




🚀 Final Notes

Monitor data flow: Device → Gateway → ChirpStack → MQTT → InfluxDB → Grafana.
Create Grafana dashboards for real-time IoT data (e.g., IMU motion, temperature).
Check ChirpStack logs if data isn’t flowing:sudo journalctl -u chirpstack-network-server



This setup is now ready for smart agriculture, industrial monitoring, or other IoT projects!
