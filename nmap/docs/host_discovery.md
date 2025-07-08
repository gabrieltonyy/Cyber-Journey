# Phase 1: Host Discovery

**Objective**: Identify which IP addresses on a network are "up" (live) before scanning ports, mapping network topology, and understanding the attack surface.

---

## 1. Why Host Discovery?

- **Efficiency**: Skip offline hosts and focus on live systems to reduce scan time significantly  
- **Stealth**: Quickly map active devices without probing every port, reducing detection risk  
- **Baseline**: Build your inventory of target machines for further analysis
- **Network Mapping**: Understand network topology, subnets, and device distribution
- **Resource Planning**: Estimate scope and time requirements for comprehensive scans

---

## 2. Key Techniques & Commands

| Technique                  | Command                                               | Description                                    |
|----------------------------|-------------------------------------------------------|------------------------------------------------|
| **List Scan (DNS only)**   | `nmap -sL 192.168.1.0/24`                             | No packets sent; lists targets and DNS names  |
| **Ping Scan**              | `nmap -sn 10.0.0.0/24`                                | ICMP & TCP "ping" probes; shows live hosts only |
| **No Ping (treat all up)** | `nmap -Pn 192.168.1.1-50`                             | Skips discovery; treats targets as up         |
| **ARP Ping (LAN only)**    | `nmap -PR 192.168.1.0/24`                             | ARP requests on local network (very reliable) |
| **TCP SYN Discovery**      | `nmap -PS22,80,443 192.168.1.0/24`                    | TCP SYN packets to specific ports             |
| **TCP ACK Discovery**      | `nmap -PA22,80,443 192.168.1.0/24`                    | TCP ACK packets to specific ports             |
| **UDP Discovery**          | `nmap -PU53,161,123 192.168.1.0/24`                   | UDP packets to specific ports                 |
| **ICMP Echo Discovery**    | `nmap -PE 192.168.1.0/24`                             | Standard ICMP echo requests                   |
| **ICMP Timestamp**         | `nmap -PP 192.168.1.0/24`                             | ICMP timestamp requests                       |
| **ICMP Netmask**           | `nmap -PM 192.168.1.0/24`                             | ICMP address mask requests                    |
| **IPv6 Host Discovery**    | `nmap -6 -sn fe80::/64`                               | Ping-scan IPv6 link-local range               |
| **Disable DNS Resolution** | `nmap -n -sn 192.168.1.0/24`                          | Skip reverse DNS for faster discovery         |
| **Force DNS Resolution**   | `nmap -R -sn 192.168.1.0/24`                          | Always resolve DNS even for IP addresses      |

---

## 3. Advanced Discovery Techniques

### 3.1 Custom Discovery Combinations
```bash
# Multi-method discovery for maximum coverage
nmap -PE -PP -PM -PS21,22,23,25,53,80,110,111,135,139,143,443,993,995,1723,3306,3389,5900,8080 192.168.1.0/24

# Stealth discovery avoiding ICMP
nmap -PS80,443 -PA22,80 -PU53,161 --disable-arp-ping 192.168.1.0/24

# IPv6 comprehensive discovery
nmap -6 -PE -PS22,80,443 fe80::/64
```

### 3.2 Discovery Through Proxies
```bash
# Discovery through SOCKS proxy
nmap -sn --proxies socks4://127.0.0.1:9050 target-network

# Discovery with specific interface
nmap -e eth1 -sn 192.168.1.0/24
```

### 3.3 Broadcast Discovery
```bash
# Use broadcast scripts for additional discovery
nmap --script broadcast-ping,broadcast-dhcp-discover,broadcast-dns-service-discovery
```

---

## 4. Understanding Discovery Methods

### 4.1 ICMP-Based Discovery
- **Echo Request (Ping)**: Most common, often blocked by firewalls
- **Timestamp Request**: Sometimes works when echo is blocked
- **Address Mask Request**: Rarely used, may bypass basic filters

### 4.2 TCP-Based Discovery  
- **SYN Discovery**: Sends SYN to common ports, waits for SYN/ACK or RST
- **ACK Discovery**: Uses ACK packets, useful for stateless firewall detection

### 4.3 UDP-Based Discovery
- **UDP Ping**: Sends UDP packets to likely-closed ports
- **Common UDP Ports**: DNS (53), SNMP (161), NTP (123)

### 4.4 ARP Discovery (Local Network)
- **Most Reliable**: Cannot be blocked by host-based firewalls
- **Layer 2**: Works at data link layer, bypasses IP-level filtering
- **Automatic**: Nmap uses ARP automatically on local subnets

---

## 3. Examples

1. **Discover live hosts on your home LAN**  
   ```bash
   nmap -sn 192.168.1.0/24
   ```
   > Output shows IPs of devices that replied to ICMP/TCP probes.

2. **List targets without sending packets**
   ```bash
   nmap -sL example.com
   ```
   > Useful to verify DNS entries or prepare target lists.

3. **Force scan even if ICMP is blocked**
   ```bash
   nmap -Pn 203.0.113.0/28
   ```
   > Use when hosts drop pings but you suspect services are running.

4. **ARP discovery for 100% accuracy on LAN**
   ```bash
   sudo nmap -sn -PR 192.168.0.0/24
   ```
   > ARP cannot be blocked by host firewall; perfect for local networks.

---

## 5. Examples

1. **Discover live hosts on your home LAN**  
   ```bash
   nmap -sn 192.168.1.0/24
   ```
   > Output shows IPs of devices that replied to ICMP/TCP probes.

2. **List targets without sending packets**
   ```bash
   nmap -sL example.com
   ```
   > Useful to verify DNS entries or prepare target lists.

3. **Force scan even if ICMP is blocked**
   ```bash
   nmap -Pn 203.0.113.0/28
   ```
   > Use when hosts drop pings but you suspect services are running.

4. **ARP discovery for 100% accuracy on LAN**
   ```bash
   sudo nmap -sn -PR 192.168.0.0/24
   ```
   > ARP cannot be blocked by host firewall; perfect for local networks.

5. **Comprehensive discovery with multiple methods**
   ```bash
   nmap -PE -PS80 -PA3389 -PU40125 192.168.1.0/24
   ```
   > Combines ICMP echo, TCP SYN, TCP ACK, and UDP discovery.

6. **Fast discovery without DNS resolution**
   ```bash
   nmap -sn -n --min-rate 1000 10.0.0.0/16
   ```
   > Speed-optimized discovery for large networks.

---

## 6. Interpreting Discovery Results

### 6.1 Host States
- **Host is up**: Host responded to at least one discovery probe
- **Host seems down**: No response to any discovery probes  
- **Skipping host**: When using `-Pn`, this message won't appear

### 6.2 Response Analysis
```text
Nmap scan report for 192.168.1.100
Host is up (0.00045s latency).
MAC Address: 00:0C:29:68:4B:A2 (VMware)
```

**Key Information:**
- **Latency**: Network delay, indicates proximity/connection quality
- **MAC Address**: Physical hardware address (local network only)  
- **Vendor**: OUI (Organizationally Unique Identifier) lookup

### 6.3 Network Distance Indicators
- **Very low latency (<1ms)**: Same network segment
- **Low latency (1-10ms)**: Same building/campus network
- **Medium latency (10-100ms)**: Regional connection
- **High latency (>100ms)**: Wide area network/internet

---

## 7. Hands-On Exercises

1. **Exercise: Home Network Comprehensive Mapping**
   * Run multiple discovery methods and compare results:
   ```bash
   # Method 1: Default ping scan
   nmap -sn 192.168.1.0/24
   
   # Method 2: ARP-only discovery  
   nmap -sn -PR 192.168.1.0/24
   
   # Method 3: TCP SYN discovery
   nmap -sn -PS22,80,443 192.168.1.0/24
   ```
   * Compare results to your router's DHCP client list and note differences.

2. **Exercise: IPv6 Discovery Techniques**
   * If your network supports IPv6, discover link-local devices:
   ```bash
   # Link-local discovery
   nmap -6 -sn fe80::/64
   
   # Global IPv6 discovery (if available)
   nmap -6 -sn 2001:db8::/32
   ```
   * Identify any IPv6-only devices (IoT, printers, modern devices).

3. **Exercise: Stealth Discovery Comparison**
   * Test different discovery methods for stealth:
   ```bash
   # Standard discovery (easily detected)
   nmap -sn 192.168.1.0/24
   
   # Stealth discovery (harder to detect)
   nmap -PS443 -PA80 --disable-arp-ping 192.168.1.0/24
   ```
   * Monitor with Wireshark to see packet differences.

4. **Exercise: DNS vs No-DNS Performance**
   * Compare discovery speed with and without DNS:
   ```bash
   # With DNS resolution
   time nmap -sn 192.168.1.0/24
   
   # Without DNS resolution
   time nmap -sn -n 192.168.1.0/24
   ```
   * Measure and document the time difference.

5. **Exercise: Discovery Through Filtering**
   * Set up a simple firewall rule and test discovery methods:
   ```bash
   # Block ICMP (simulate filtered environment)
   sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
   
   # Test different discovery methods
   nmap -PE 192.168.1.1    # ICMP echo (should fail)
   nmap -PS80 192.168.1.1  # TCP SYN (should work)
   nmap -PR 192.168.1.1    # ARP (should work on LAN)
   ```

---

## 8. Tips & Best Practices

### 8.1 Performance Optimization
* **Use `-n`** to skip DNS resolution for faster discovery on large networks
* **Combine methods**: `-PE -PS80,443 -PA3389` for comprehensive coverage
* **Adjust timing**: Use `-T4` for faster discovery on reliable networks
* **Parallel processing**: `--min-hostgroup 256` for large network ranges

### 8.2 Stealth Considerations
* **ARP discovery** is silent and reliable on local networks
* **TCP discovery** on common ports (80, 443) blends with normal traffic
* **Avoid ICMP** in security-conscious environments
* **Randomize host order**: `--randomize-hosts` to avoid patterns

### 8.3 Network-Specific Strategies
* **Corporate Networks**: Use TCP discovery on business ports (80, 443, 22)
* **Home Networks**: ARP discovery provides complete accuracy
* **Cloud Environments**: Expect high latency, use `-Pn` for known hosts
* **IPv6 Networks**: Always test both IPv4 and IPv6 discovery methods

### 8.4 Legal and Ethical Guidelines
* **Own networks only**: Only scan networks you own or have explicit permission to scan
* **Respect bandwidth**: Use appropriate timing to avoid network disruption  
* **Document findings**: Keep records of discovery results for network management
* **Privacy considerations**: Be mindful of discovering personal devices

---

## 5. Tips & Best Practices

* **Run as root/admin** for best accuracy (ARP & raw packets).
* **Combine with `-oG`** to save grepable output:
  ```bash
  nmap -sn 192.168.1.0/24 -oG livehosts.txt
  ```
* **Filter results** using grep/awk to extract only live IPs for further scanning.
* **Respect privacy** and only scan networks you own or have explicit permission to scan.

### Netcat Tip: Quick Live-Host Check

Instead of `nmap -sn`, you can do a lightweight TCP ping on port 80 with Netcat:

```bash
nc -zv 192.168.1.0/24 80
```

**Command breakdown:**
- `-z` = Zero-I/O mode (scan without sending data)
- `-v` = Verbose output (show connection attempts)
- `80` = Target port to test connectivity

This will print which IPs respond on TCP 80—great for a quick sanity check when ICMP is blocked.

> 💡 **Want to learn more Netcat techniques?** Check out our [Netcat Learning Guide](../netcat/) for comprehensive coverage of this versatile tool.

---

## 9. Advanced Discovery Scripting

### 9.1 Automated Discovery Pipeline
```bash
#!/bin/bash
# comprehensive-discovery.sh

NETWORK="192.168.1.0/24"
OUTPUT_DIR="discovery_results"

mkdir -p $OUTPUT_DIR

# Stage 1: Fast ARP discovery
echo "Stage 1: ARP Discovery..."
nmap -sn -PR $NETWORK -oG $OUTPUT_DIR/arp_discovery.grep

# Stage 2: TCP discovery for non-ARP responsive hosts  
echo "Stage 2: TCP Discovery..."
nmap -sn -PS22,80,443 $NETWORK -oG $OUTPUT_DIR/tcp_discovery.grep

# Stage 3: UDP discovery for additional coverage
echo "Stage 3: UDP Discovery..."  
nmap -sn -PU53,161,123 $NETWORK -oG $OUTPUT_DIR/udp_discovery.grep

# Combine and deduplicate results
grep "Up" $OUTPUT_DIR/*.grep | cut -d' ' -f2 | sort -u > $OUTPUT_DIR/live_hosts.txt

echo "Discovery complete. Live hosts saved to $OUTPUT_DIR/live_hosts.txt"
```

### 9.2 Discovery Result Processing
```bash
# Extract live IPs for further scanning
grep "Up" discovery.grep | awk '{print $2}' > live_ips.txt

# Extract MAC addresses and vendors
grep "Up" discovery.grep | grep -oE "([0-9A-F]{2}:){5}[0-9A-F]{2}" > mac_addresses.txt

# Create target list for next phase
while read ip; do echo $ip >> targets_for_port_scan.txt; done < live_ips.txt
```
