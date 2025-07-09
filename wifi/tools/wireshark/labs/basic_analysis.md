# Wireshark Basic Analysis Laboratory

**Objective**: Master the fundamentals of wireless protocol analysis using Wireshark through hands-on packet capture and analysis exercises.

---

## 🎯 Lab Prerequisites

### **Software Requirements**
```bash
# Install Wireshark
sudo apt update
sudo apt install wireshark

# Add user to wireshark group
sudo usermod -a -G wireshark $USER

# Install additional tools
sudo apt install -y tshark tcpdump aircrack-ng
```

### **Hardware Setup**
- Wireless adapter with monitor mode support
- Wireless router for testing
- Target devices (phones, laptops)
- Isolated network environment

### **Initial Configuration**
```bash
# Verify Wireshark installation
wireshark --version

# Check available interfaces
tshark -D

# Verify group membership
groups $USER
```

---

## 📚 Phase 1: Interface Configuration

### **Exercise 1.1: Capture Interface Setup**

**Objective**: Configure Wireshark for wireless packet capture.

```bash
# Start monitor mode
sudo airmon-ng start wlan0

# Launch Wireshark
sudo wireshark

# Or use command line
sudo wireshark -i wlan0mon
```

**Wireshark Setup**:
1. **Interface Selection**: Choose `wlan0mon` from interface list
2. **Capture Options**: Configure promiscuous mode
3. **Display Settings**: Enable time display
4. **Buffer Settings**: Set appropriate ring buffer size

### **Exercise 1.2: Basic Capture Configuration**

**Objective**: Optimize capture settings for wireless analysis.

**Capture Filter Setup**:
```
# Capture all wireless traffic
(no filter needed)

# Capture only management frames
wlan type mgt

# Capture specific channel traffic
wlan and channel 6
```

**Display Configuration**:
```
View → Time Display Format → Time of Day
View → Name Resolution → Disable All (for performance)
```

---

## 🔍 Phase 2: Frame Analysis

### **Exercise 2.1: Management Frame Analysis**

**Objective**: Understand and analyze 802.11 management frames.

```bash
# Start capture
sudo wireshark -i wlan0mon

# Apply display filter
wlan.fc.type == 0
```

**Analysis Tasks**:
1. **Beacon Frame Analysis**:
   - Filter: `wlan.fc.subtype == 0x08`
   - Examine SSID, supported rates, channel info
   - Note beacon interval and capabilities

2. **Probe Request/Response Analysis**:
   - Filter: `wlan.fc.subtype == 0x04 || wlan.fc.subtype == 0x05`
   - Identify client scanning behavior
   - Analyze requested vs advertised capabilities

3. **Authentication Frame Analysis**:
   - Filter: `wlan.fc.subtype == 0x0B`
   - Examine authentication algorithms
   - Identify success/failure patterns

**Lab Report Template**:
```
Management Frame Analysis
========================
Frame Type: Beacon
BSSID: [MAC Address]
SSID: [Network Name]
Channel: [Channel Number]
Beacon Interval: [Milliseconds]
Capabilities: [Capability Info]
Supported Rates: [Rate List]
Security: [Security Type]
```

### **Exercise 2.2: Control Frame Analysis**

**Objective**: Analyze control frames and medium access control.

```bash
# Filter for control frames
wlan.fc.type == 1

# Specific control frame types
wlan.fc.subtype == 0x1B  # RTS frames
wlan.fc.subtype == 0x1C  # CTS frames
wlan.fc.subtype == 0x1D  # ACK frames
```

**Analysis Focus**:
- RTS/CTS handshake patterns
- ACK frame timing
- Hidden node problem detection
- Medium access efficiency

### **Exercise 2.3: Data Frame Analysis**

**Objective**: Examine data frame structure and content.

```bash
# Filter for data frames
wlan.fc.type == 2

# QoS data frames
wlan.fc.subtype == 0x28

# Null data frames
wlan.fc.subtype == 0x24
```

**Analysis Tasks**:
1. Data frame addressing (To DS, From DS)
2. QoS priority analysis
3. Frame fragmentation examination
4. Sequence number analysis

---

## 🔐 Phase 3: Security Protocol Analysis

### **Exercise 3.1: WPA/WPA2 Handshake Analysis**

**Objective**: Capture and analyze WPA/WPA2 4-way handshake.

```bash
# Start capture targeting specific AP
sudo wireshark -i wlan0mon -f "wlan host 00:11:22:33:44:55"

# Filter for EAPOL frames
eapol
```

**Handshake Analysis Steps**:

1. **Message 1 Analysis**:
   ```
   Filter: eapol.keydes.key_info.key_ack == 1 && eapol.keydes.key_info.key_mic == 0
   
   Analysis:
   - ANonce value
   - Key information flags
   - Key descriptor version
   ```

2. **Message 2 Analysis**:
   ```
   Filter: eapol.keydes.key_info.key_mic == 1 && eapol.keydes.key_info.key_ack == 0
   
   Analysis:
   - SNonce value
   - MIC verification
   - RSN information element
   ```

3. **Message 3 Analysis**:
   ```
   Filter: eapol.keydes.key_info.key_ack == 1 && eapol.keydes.key_info.install == 1
   
   Analysis:
   - Install flag set
   - GTK distribution
   - Key data encryption
   ```

4. **Message 4 Analysis**:
   ```
   Filter: eapol.keydes.key_info.key_mic == 1 && eapol.keydes.key_info.secure == 1
   
   Analysis:
   - Secure flag set
   - Handshake completion
   - Key installation confirmation
   ```

### **Exercise 3.2: WEP Analysis**

**Objective**: Analyze WEP encryption implementation.

```bash
# Filter for WEP encrypted frames
wlan.wep

# Examine WEP parameters
wlan.wep.iv    # Initialization Vector
wlan.wep.key   # Key ID
wlan.wep.icv   # Integrity Check Value
```

**Analysis Tasks**:
1. IV collision detection
2. Weak IV identification
3. Key ID usage patterns
4. ICV verification

---

## 📊 Phase 4: Statistical Analysis

### **Exercise 4.1: Protocol Distribution Analysis**

**Objective**: Analyze wireless traffic patterns and protocol distribution.

**Protocol Hierarchy**:
```
Statistics → Protocol Hierarchy
```

**Analysis Tasks**:
1. Identify dominant protocols
2. Calculate management vs data frame ratio
3. Analyze encryption usage
4. Identify unusual traffic patterns

### **Exercise 4.2: Conversation Analysis**

**Objective**: Analyze communication patterns between wireless devices.

**Conversation Statistics**:
```
Statistics → Conversations → WLAN
```

**Analysis Focus**:
- Client-AP communication patterns
- Data transfer volumes
- Connection duration
- Throughput analysis

### **Exercise 4.3: Expert Information Analysis**

**Objective**: Identify network issues using Wireshark's expert system.

**Expert Information**:
```
Analyze → Expert Information
```

**Issue Categories**:
1. **Errors**: Protocol violations, malformed packets
2. **Warnings**: Retransmissions, out-of-order packets
3. **Notes**: Connection events, unusual behavior
4. **Chats**: Normal protocol events

---

## 🎨 Phase 5: Advanced Filtering

### **Exercise 5.1: Complex Filter Creation**

**Objective**: Create advanced filters for specific analysis scenarios.

**Example Filters**:
```bash
# Find all handshakes for specific AP
eapol && wlan.bssid == 00:11:22:33:44:55

# Find weak signal frames
radiotap.dbm_antsignal < -70

# Find retransmissions
wlan.analysis.retransmission

# Find probe requests from specific client
wlan.fc.subtype == 0x04 && wlan.sa == 66:77:88:99:AA:BB

# Find deauthentication attacks
wlan.fc.subtype == 0x0C && wlan.fixed.reason_code
```

### **Exercise 5.2: Custom Column Configuration**

**Objective**: Configure custom columns for efficient analysis.

**Column Configuration**:
```
Right-click column header → Column Preferences

Add columns:
- RSSI: radiotap.dbm_antsignal
- Channel: wlan_radio.channel
- SSID: wlan_mgt.ssid
- Frame Type: wlan.fc.type_subtype
- Encryption: wlan.wep || wlan.rsn || wlan.wpa
```

---

## 🔍 Phase 6: Attack Detection

### **Exercise 6.1: Deauthentication Attack Detection**

**Objective**: Identify and analyze deauthentication attacks.

```bash
# Filter for deauth frames
wlan.fc.subtype == 0x0C

# Analyze reason codes
wlan.fixed.reason_code

# Find mass deauth attacks
wlan.fc.subtype == 0x0C && frame.time_delta < 0.1
```

**Analysis Checklist**:
- [ ] Unusual deauth frequency
- [ ] Specific reason codes
- [ ] Source MAC analysis
- [ ] Targeted vs broadcast deauth
- [ ] Timing patterns

### **Exercise 6.2: Rogue AP Detection**

**Objective**: Identify potential rogue access points.

```bash
# Find multiple APs with same SSID
wlan.fc.subtype == 0x08 && wlan_mgt.ssid == "TargetNetwork"

# Analyze beacon differences
wlan_mgt.beacon.capabilities
wlan_mgt.beacon.interval
wlan_mgt.country_info
```

**Detection Indicators**:
- Duplicate SSIDs with different BSSIDs
- Inconsistent security settings
- Unusual beacon intervals
- Different vendor information

---

## 📈 Phase 7: Performance Analysis

### **Exercise 7.1: Signal Strength Analysis**

**Objective**: Analyze wireless signal quality and performance.

```bash
# Create I/O graph for signal strength
Statistics → I/O Graphs

# Y-axis: AVG(radiotap.dbm_antsignal)
# X-axis: Time
# Filter: wlan.addr == 00:11:22:33:44:55
```

**Analysis Tasks**:
1. Signal strength variations over time
2. Correlation with distance/movement
3. Interference patterns
4. Roaming behavior analysis

### **Exercise 7.2: Throughput Analysis**

**Objective**: Analyze wireless network performance and throughput.

```bash
# Create throughput graph
Statistics → I/O Graphs

# Y-axis: SUM(frame.len)
# X-axis: Time
# Filter: wlan.fc.type == 2
```

**Performance Metrics**:
- Data frame rate
- Retransmission rate
- Average frame size
- Channel utilization

---

## 💾 Phase 8: Export and Reporting

### **Exercise 8.1: Data Export**

**Objective**: Export analysis results for further processing.

**Export Options**:
```bash
# Export filtered packets
File → Export Specified Packets
- Format: PCAP, CSV, JSON, XML
- Packet range: Displayed packets

# Export statistics
Statistics → [Various] → Copy → CSV

# Command line export
tshark -r capture.pcap -Y "eapol" -T fields -e frame.time -e wlan.sa -e wlan.da -E header=y -E separator=, > handshakes.csv
```

### **Exercise 8.2: Report Generation**

**Objective**: Create comprehensive analysis reports.

**Report Template**:
```
Wireless Network Analysis Report
===============================

Executive Summary:
- Network: [SSID]
- Analysis Period: [Start] to [End]
- Total Packets: [Count]
- Security Assessment: [Summary]

Detailed Findings:
1. Network Configuration
   - BSSID: [MAC]
   - Channel: [Number]
   - Security: [Type]
   - Signal Strength: [dBm]

2. Client Analysis
   - Connected Clients: [Count]
   - Client Behavior: [Description]
   - Roaming Patterns: [Analysis]

3. Security Analysis
   - Handshake Capture: [Status]
   - Attack Detection: [Results]
   - Vulnerabilities: [List]

4. Performance Analysis
   - Throughput: [Mbps]
   - Retransmission Rate: [%]
   - Channel Utilization: [%]

Recommendations:
- [Security improvements]
- [Performance optimizations]
- [Monitoring suggestions]
```

---

## 🛠️ Phase 9: Troubleshooting

### **Exercise 9.1: Common Issues**

**Objective**: Identify and resolve common Wireshark issues.

**Permission Issues**:
```bash
# Add user to wireshark group
sudo usermod -a -G wireshark $USER

# Verify permissions
ls -la /usr/bin/dumpcap

# Alternative: Use sudo
sudo wireshark -i wlan0mon
```

**Performance Issues**:
```bash
# Disable name resolution
View → Name Resolution → Disable All

# Use ring buffer
-b filesize:100000 -a files:10

# Limit packet size
-s 200
```

**Capture Issues**:
```bash
# Check interface status
iwconfig wlan0mon

# Verify monitor mode
sudo airmon-ng check

# Check for conflicts
sudo airmon-ng check kill
```

---

## 🎓 Lab Completion

### **Skills Demonstrated**
- [x] Wireshark interface configuration
- [x] 802.11 frame analysis
- [x] Security protocol dissection
- [x] Statistical analysis
- [x] Advanced filtering techniques
- [x] Attack detection methods
- [x] Performance analysis
- [x] Data export and reporting

### **Next Steps**
1. Complete [Security Analysis Lab](security_analysis.md)
2. Practice with different network configurations
3. Develop custom analysis scripts
4. Learn Lua scripting for automation

---

## 📚 Additional Resources

### **Sample Captures**
- Create captures with different security types
- Generate various attack scenarios
- Build comprehensive test dataset

### **Analysis Scripts**
```bash
# Automated handshake detection
tshark -r capture.pcap -Y "eapol" -T fields -e frame.number -e wlan.bssid | sort -u

# Signal strength analysis
tshark -r capture.pcap -Y "wlan.fc.subtype == 0x08" -T fields -e wlan.bssid -e radiotap.dbm_antsignal

# Frame count by type
tshark -r capture.pcap -q -z proto,colinfo,wlan.fc.type,wlan.fc.type
```

### **Documentation**
- Keep detailed analysis notes
- Document filter expressions
- Record performance metrics
- Maintain capture file inventory

---

*Basic Analysis Lab Complete - Ready for Advanced Security Analysis*
