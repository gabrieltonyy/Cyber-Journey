# Wireshark Wireless Analysis Cheat Sheet

**Complete reference for Wireshark display filters, capture techniques, and wireless protocol analysis**

---

## 🔍 Display Filters - 802.11 Wireless

### **Basic Frame Filters**

| Filter | Description | Example |
|--------|-------------|---------|
| `wlan` | All wireless frames | `wlan` |
| `wlan.fc.type == 0` | Management frames | `wlan.fc.type == 0` |
| `wlan.fc.type == 1` | Control frames | `wlan.fc.type == 1` |
| `wlan.fc.type == 2` | Data frames | `wlan.fc.type == 2` |
| `wlan.fc.subtype` | Frame subtype | `wlan.fc.subtype == 0x08` |

### **Management Frame Subtypes**

| Subtype | Filter | Description |
|---------|--------|-------------|
| 0x00 | `wlan.fc.subtype == 0x00` | Association Request |
| 0x01 | `wlan.fc.subtype == 0x01` | Association Response |
| 0x02 | `wlan.fc.subtype == 0x02` | Reassociation Request |
| 0x03 | `wlan.fc.subtype == 0x03` | Reassociation Response |
| 0x04 | `wlan.fc.subtype == 0x04` | Probe Request |
| 0x05 | `wlan.fc.subtype == 0x05` | Probe Response |
| 0x08 | `wlan.fc.subtype == 0x08` | Beacon Frame |
| 0x09 | `wlan.fc.subtype == 0x09` | ATIM |
| 0x0A | `wlan.fc.subtype == 0x0A` | Disassociation |
| 0x0B | `wlan.fc.subtype == 0x0B` | Authentication |
| 0x0C | `wlan.fc.subtype == 0x0C` | Deauthentication |
| 0x0D | `wlan.fc.subtype == 0x0D` | Action |

### **Control Frame Subtypes**

| Subtype | Filter | Description |
|---------|--------|-------------|
| 0x18 | `wlan.fc.subtype == 0x18` | Block ACK Request |
| 0x19 | `wlan.fc.subtype == 0x19` | Block ACK |
| 0x1A | `wlan.fc.subtype == 0x1A` | PS-Poll |
| 0x1B | `wlan.fc.subtype == 0x1B` | RTS (Request to Send) |
| 0x1C | `wlan.fc.subtype == 0x1C` | CTS (Clear to Send) |
| 0x1D | `wlan.fc.subtype == 0x1D` | ACK |
| 0x1E | `wlan.fc.subtype == 0x1E` | CF-End |
| 0x1F | `wlan.fc.subtype == 0x1F` | CF-End+CF-ACK |

### **Data Frame Subtypes**

| Subtype | Filter | Description |
|---------|--------|-------------|
| 0x20 | `wlan.fc.subtype == 0x20` | Data |
| 0x21 | `wlan.fc.subtype == 0x21` | Data + CF-ACK |
| 0x22 | `wlan.fc.subtype == 0x22` | Data + CF-Poll |
| 0x23 | `wlan.fc.subtype == 0x23` | Data + CF-ACK + CF-Poll |
| 0x24 | `wlan.fc.subtype == 0x24` | Null Data |
| 0x28 | `wlan.fc.subtype == 0x28` | QoS Data |
| 0x2C | `wlan.fc.subtype == 0x2C` | QoS Null Data |

---

## 🔐 Security Protocol Filters

### **WPA/WPA2 Analysis**

| Filter | Description |
|--------|-------------|
| `eapol` | All EAPOL frames |
| `eapol.type == 3` | EAPOL-Key frames |
| `eapol.keydes.type == 2` | WPA Key frames |
| `eapol.keydes.key_info.key_type == 1` | Pairwise keys |
| `eapol.keydes.key_info.key_type == 0` | Group keys |
| `eapol.keydes.key_info.install == 1` | Key installation |
| `eapol.keydes.key_info.ack == 1` | Key ACK |
| `eapol.keydes.key_info.mic == 1` | MIC present |
| `eapol.keydes.key_info.secure == 1` | Secure bit set |
| `eapol.keydes.key_info.error == 1` | Error bit set |
| `eapol.keydes.key_info.request == 1` | Request bit set |
| `eapol.keydes.key_info.encrypted_key_data == 1` | Encrypted key data |

### **WPA Handshake Analysis**

| Handshake Step | Filter |
|----------------|--------|
| Message 1 | `eapol.keydes.key_info.key_type == 1 && eapol.keydes.key_info.key_ack == 1 && eapol.keydes.key_info.key_mic == 0` |
| Message 2 | `eapol.keydes.key_info.key_type == 1 && eapol.keydes.key_info.key_mic == 1 && eapol.keydes.key_info.key_ack == 0` |
| Message 3 | `eapol.keydes.key_info.key_type == 1 && eapol.keydes.key_info.key_ack == 1 && eapol.keydes.key_info.key_mic == 1 && eapol.keydes.key_info.install == 1` |
| Message 4 | `eapol.keydes.key_info.key_type == 1 && eapol.keydes.key_info.key_mic == 1 && eapol.keydes.key_info.key_ack == 0 && eapol.keydes.key_info.install == 0` |

### **WEP Analysis**

| Filter | Description |
|--------|-------------|
| `wlan.wep` | WEP encrypted frames |
| `wlan.wep.iv` | WEP Initialization Vector |
| `wlan.wep.key` | WEP Key ID |
| `wlan.wep.icv` | WEP Integrity Check Value |

### **WPS Analysis**

| Filter | Description |
|--------|-------------|
| `wps` | WiFi Protected Setup |
| `wps.wifi_protected_setup_state` | WPS State |
| `wps.selected_registrar` | Selected Registrar |
| `wps.device_password_id` | Device Password ID |
| `wps.configuration_methods` | Configuration Methods |

---

## 🎯 Address and Network Filters

### **MAC Address Filtering**

| Filter | Description |
|--------|-------------|
| `wlan.addr == 00:11:22:33:44:55` | Specific MAC address |
| `wlan.sa == 00:11:22:33:44:55` | Source address |
| `wlan.da == 00:11:22:33:44:55` | Destination address |
| `wlan.bssid == 00:11:22:33:44:55` | BSSID (AP address) |
| `wlan.ta == 00:11:22:33:44:55` | Transmitter address |
| `wlan.ra == 00:11:22:33:44:55` | Receiver address |

### **SSID and Network Filters**

| Filter | Description |
|--------|-------------|
| `wlan_mgt.ssid == "NetworkName"` | Specific SSID |
| `wlan_mgt.ssid contains "Guest"` | SSID contains string |
| `wlan_mgt.ssid matches ".*[0-9].*"` | SSID with numbers |
| `wlan.ssid` | Any SSID field |

### **Channel and Frequency**

| Filter | Description |
|--------|-------------|
| `radiotap.channel.freq == 2412` | Channel 1 (2.4 GHz) |
| `radiotap.channel.freq == 2437` | Channel 6 (2.4 GHz) |
| `radiotap.channel.freq == 2462` | Channel 11 (2.4 GHz) |
| `radiotap.channel.freq == 5180` | Channel 36 (5 GHz) |
| `wlan_radio.channel` | Channel number |

---

## 📊 Statistical Analysis

### **Expert Information Filters**

| Filter | Description |
|--------|-------------|
| `_ws.expert` | All expert information |
| `_ws.expert.severity == "Error"` | Error level issues |
| `_ws.expert.severity == "Warn"` | Warning level issues |
| `_ws.expert.severity == "Note"` | Note level issues |
| `_ws.expert.group == "Security"` | Security-related issues |

### **Performance Analysis**

| Filter | Description |
|--------|-------------|
| `wlan.analysis.retransmission` | Retransmitted frames |
| `wlan.analysis.duplicate` | Duplicate frames |
| `radiotap.dbm_antsignal` | Signal strength |
| `radiotap.dbm_antnoise` | Noise level |
| `wlan.duration` | Frame duration |

---

## 🔧 Advanced Filtering

### **Logical Operators**

| Operator | Example | Description |
|----------|---------|-------------|
| `&&` | `wlan.fc.type == 0 && wlan.fc.subtype == 0x08` | AND operation |
| `\|\|` | `wlan.fc.subtype == 0x0B \|\| wlan.fc.subtype == 0x0C` | OR operation |
| `!` | `!wlan.fc.type == 0` | NOT operation |
| `in` | `wlan.fc.subtype in {0x0B 0x0C}` | IN operation |

### **Comparison Operators**

| Operator | Example | Description |
|----------|---------|-------------|
| `==` | `wlan.fc.type == 0` | Equal to |
| `!=` | `wlan.fc.type != 0` | Not equal to |
| `>` | `frame.len > 1000` | Greater than |
| `<` | `frame.len < 100` | Less than |
| `>=` | `radiotap.dbm_antsignal >= -50` | Greater than or equal |
| `<=` | `radiotap.dbm_antsignal <= -70` | Less than or equal |

### **String Matching**

| Function | Example | Description |
|----------|---------|-------------|
| `contains` | `wlan_mgt.ssid contains "Guest"` | Contains substring |
| `matches` | `wlan_mgt.ssid matches ".*[0-9].*"` | Regex matching |
| `startswith` | `wlan_mgt.ssid startswith "WiFi"` | Starts with string |
| `endswith` | `wlan_mgt.ssid endswith "5G"` | Ends with string |

---

## 📡 Common Analysis Scenarios

### **Handshake Analysis**
```
# Complete WPA handshake
eapol && (wlan.addr == 00:11:22:33:44:55 || wlan.addr == 66:77:88:99:AA:BB)

# Failed handshake attempts
eapol && _ws.expert.severity == "Warn"

# Handshake with specific AP
eapol && wlan.bssid == 00:11:22:33:44:55
```

### **Attack Detection**
```
# Deauthentication attacks
wlan.fc.subtype == 0x0C && wlan.fixed.reason_code

# Multiple deauth from same source
wlan.fc.subtype == 0x0C && wlan.sa == 00:11:22:33:44:55

# Beacon flooding
wlan.fc.subtype == 0x08 && frame.time_delta < 0.1
```

### **Client Analysis**
```
# Probe requests from specific client
wlan.fc.subtype == 0x04 && wlan.sa == 00:11:22:33:44:55

# Association attempts
wlan.fc.subtype == 0x00 || wlan.fc.subtype == 0x01

# Roaming behavior
wlan.fc.subtype == 0x02 || wlan.fc.subtype == 0x03
```

---

## 🛠️ Capture Filters

### **Basic Capture Filters**

| Filter | Description |
|--------|-------------|
| `type mgt` | Management frames only |
| `type ctl` | Control frames only |
| `type data` | Data frames only |
| `subtype beacon` | Beacon frames only |
| `subtype probe-req` | Probe requests only |
| `subtype probe-resp` | Probe responses only |

### **Advanced Capture Filters**
```bash
# Capture specific AP traffic
wlan addr1 00:11:22:33:44:55 or wlan addr2 00:11:22:33:44:55

# Capture management frames only
wlan type mgt

# Capture handshake traffic
wlan type data and wlan subtype data
```

---

## 📈 Statistics and Analysis

### **Protocol Hierarchy**
```
Statistics → Protocol Hierarchy
- Shows protocol distribution
- Identifies unusual traffic patterns
- Provides bandwidth analysis
```

### **Conversations**
```
Statistics → Conversations → WLAN
- Shows client-AP communications
- Identifies traffic patterns
- Provides throughput analysis
```

### **I/O Graphs**
```
Statistics → I/O Graphs
- Visualizes traffic over time
- Identifies peak usage periods
- Shows attack patterns
```

### **Expert Information**
```
Analyze → Expert Information
- Shows protocol violations
- Identifies security issues
- Provides performance warnings
```

---

## 🔐 Decryption Configuration

### **WEP Decryption**
```
Edit → Preferences → Protocols → IEEE 802.11
- Enable decryption
- Add WEP keys: wep:01:02:03:04:05
- Key format: key:01:02:03:04:05:06:07:08:09:0a:0b:0c:0d
```

### **WPA/WPA2 Decryption**
```
Edit → Preferences → Protocols → IEEE 802.11
- Enable decryption
- Add WPA-PWD: NetworkName:password
- Add WPA-PSK: NetworkName:psk_in_hex
```

### **TLS/SSL Decryption**
```
Edit → Preferences → Protocols → TLS
- RSA keys list: Add private key
- Pre-Master-Secret log filename
```

---

## 🎨 Display Configuration

### **Custom Columns**
```
Right-click column header → Column Preferences
- Add RSSI: radiotap.dbm_antsignal
- Add Channel: wlan_radio.channel
- Add SSID: wlan_mgt.ssid
- Add Frame Type: wlan.fc.type_subtype
```

### **Coloring Rules**
```
View → Coloring Rules
- Management frames: wlan.fc.type == 0 (Blue)
- Control frames: wlan.fc.type == 1 (Green)
- Data frames: wlan.fc.type == 2 (Gray)
- EAPOL frames: eapol (Red)
- Beacon frames: wlan.fc.subtype == 0x08 (Yellow)
```

---

## 🔍 Search and Navigation

### **Find Packet**
```
Edit → Find Packet (Ctrl+F)
- Display filter: eapol
- Hex value: 01 02 03 04
- String: NetworkName
- Regular expression: .*[0-9].*
```

### **Go to Packet**
```
Go → Go to Packet (Ctrl+G)
- Packet number: 1234
- Time: 14:30:00
- Offset: +100
```

### **Follow Stream**
```
Right-click → Follow → TCP Stream
- Shows conversation flow
- Highlights related packets
- Provides raw data view
```

---

## 📤 Export Options

### **Export Formats**
```
File → Export Specified Packets
- All packets / Selected packets / Marked packets
- Displayed packets only
- Format: pcap, pcapng, csv, json, xml
```

### **Export Objects**
```
File → Export Objects
- HTTP objects
- TFTP objects
- SMB objects
- IMF objects
```

### **Export Packet Data**
```
File → Export Packet Dissections
- As Plain Text
- As CSV
- As JSON
- As XML
```

---

## 🎯 Common One-Liners

### **Quick Analysis**
```bash
# Find all handshakes
eapol

# Find deauth attacks
wlan.fc.subtype == 0x0C

# Find probe requests
wlan.fc.subtype == 0x04

# Find weak signals
radiotap.dbm_antsignal < -70

# Find retransmissions
wlan.analysis.retransmission
```

### **Security Analysis**
```bash
# WPA handshake analysis
eapol && wlan.bssid == 00:11:22:33:44:55

# Find management frame anomalies
wlan.fc.type == 0 && _ws.expert.severity == "Warn"

# Detect beacon flooding
wlan.fc.subtype == 0x08 && frame.time_delta < 0.1

# Find authentication failures
wlan.fc.subtype == 0x0B && wlan.fixed.status_code != 0
```

---

## 📊 Performance Optimization

### **Capture Optimization**
```bash
# Use ring buffer
-b filesize:1000 -a files:10

# Capture specific interface
-i wlan0mon

# Use capture filter
-f "wlan type mgt"

# Limit packet size
-s 200
```

### **Analysis Optimization**
```bash
# Disable name resolution
View → Name Resolution → Disable All

# Use specific protocols only
Analyze → Enabled Protocols → Disable unused

# Limit displayed packets
View → Limit to Display Filter
```

---

## 🔗 Integration Commands

### **Command Line Tools**
```bash
# Basic capture
tshark -i wlan0mon -w capture.pcap

# Live analysis
tshark -i wlan0mon -Y "eapol" -T fields -e frame.time -e wlan.sa -e wlan.da

# Statistics
tshark -r capture.pcap -q -z proto,colinfo,wlan.fc.type,wlan.fc.type

# Export to CSV
tshark -r capture.pcap -T fields -e frame.time -e wlan.sa -e wlan.da -E header=y -E separator=, > output.csv
```

### **Automated Analysis**
```bash
# Find handshakes
tshark -r capture.pcap -Y "eapol" -T fields -e frame.number -e wlan.bssid

# Count frame types
tshark -r capture.pcap -q -z proto,colinfo,wlan.fc.type,wlan.fc.type

# Extract SSIDs
tshark -r capture.pcap -Y "wlan.fc.subtype == 0x08" -T fields -e wlan_mgt.ssid | sort -u
```

---

**Remember**: Always capture and analyze traffic legally and ethically. Only analyze networks you own or have explicit permission to monitor.

---

*Wireshark v4.0+ - The world's foremost network protocol analyzer*
