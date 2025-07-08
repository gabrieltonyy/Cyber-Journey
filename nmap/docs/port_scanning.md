# Phase 2: Port Scanning

**Objective**  
Discover which ports on a live host are **open**, **closed**, or **filtered**—forming the foundation for deeper service analysis, vulnerability assessment, and attack surface mapping.

---

## 1. Why Port Scanning?

- **Identify Attack Surface**  
  Every open port represents a potential entry point for attackers and legitimate access.

- **Prioritize Services**  
  Focus on commonly exposed services (HTTP, SSH, DNS, databases) for further investigation.

- **Measure Defenses**  
  "Filtered" ports indicate firewall or packet‐filter rules in action, revealing security posture.

- **Service Discovery**  
  Understand what services are running before attempting version detection or exploitation.

- **Compliance Assessment**  
  Verify that only authorized services are exposed according to security policies.

---

## 2. Scan Types & Key Options

| Scan Type              | Option         | Description                                                                 | Use Case |
|------------------------|----------------|-----------------------------------------------------------------------------|----------|
| **TCP SYN (Stealth)**  | `-sS`          | Sends SYN, waits for SYN-ACK (open) or RST (closed); does not complete handshake | Default choice, fast and stealthy |
| **TCP Connect**        | `-sT`          | Uses OS connect syscall; full handshake—easier to detect, needs no raw sockets | Non-privileged scanning, through proxies |
| **UDP Scan**           | `-sU`          | Sends empty UDP packets; no reply = open\|filtered, ICMP port-unreachable = closed | DNS, SNMP, DHCP service discovery |
| **ACK Scan**           | `-sA`          | Sends ACK; RST = unfiltered, no reply = filtered; used for firewall mapping | Firewall rule testing |
| **Window Scan**        | `-sW`          | Uses TCP window field to differentiate ports; works on some systems | Advanced firewall testing |
| **Maimon Scan**        | `-sM`          | FIN/ACK probe; relies on RFC 793 compliance | Stealth scanning of specific systems |
| **Null, FIN, Xmas**    | `-sN`/`-sF`/`-sX` | Sends unusual flag combinations; no reply = open\|filtered | Stealth evasion, Unix/Linux systems |
| **Full Port Range**    | `-p-`          | Scans all 65,535 TCP ports (combined with `-sS` or `-sT`) | Comprehensive discovery |
| **Top Ports**          | `-F`           | Scan the 100 most common ports quickly | Fast initial assessment |
| **Specific Ports**     | `-p 22,80,443` | Scan only listed ports or ranges (`-p 1-1024`) | Targeted service checking |
| **Idle Scan**          | `-sI`          | Uses a "zombie" host for blind scanning; very stealthy (advanced) | Anonymous scanning |
| **Protocol Scan**      | `-sO`          | IP protocol scan (ICMP, TCP, UDP, etc.) | Network protocol discovery |
| **SCTP Scans**         | `-sY`/`-sZ`    | SCTP INIT/COOKIE-ECHO scans | Telecommunications networks |

---

## 3. Advanced Port Scanning Techniques

### 3.1 Custom Port Specifications
```bash
# Scan specific service combinations
nmap -p 21,22,23,25,53,80,110,111,135,139,143,443,993,995,1723,3306,3389,5900 target

# Mixed TCP and UDP scanning
nmap -p T:80,443,U:53,161,123 target

# Port ranges with exclusions
nmap -p 1-1000 --exclude-ports 25,135 target

# Scan by service names
nmap -p ftp,ssh,telnet,smtp,http,https target
```

### 3.2 Performance Optimization
```bash
# High-speed scanning
nmap -sS -T4 --min-rate 1000 --max-retries 2 target

# Parallel port scanning
nmap --min-parallelism 100 --max-parallelism 300 target

# Efficient large network scanning
nmap -sS -p 22,80,443 --min-hostgroup 50 network/24
```

### 3.3 Stealth and Evasion
```bash
# Ultra-stealth scanning
nmap -sS -T1 --scan-delay 5s --randomize-hosts target

# Firewall evasion combination
nmap -sF -f --data-length 200 --source-port 53 target

# Decoy scanning for anonymity
nmap -sS -D RND:10 target
```

---

## 4. Examples

1. **Basic SYN Scan**  
   ```bash
   nmap -sS 192.168.2.10
   ```
   > Fast, stealthy probe of the 1,000 most common TCP ports.

2. **Scan Specific Ports**
   ```bash
   nmap -sS -p 22,80,443 192.168.2.10
   ```
   > Only checks SSH, HTTP, and HTTPS.

3. **UDP Service Discovery**
   ```bash
   sudo nmap -sU -p 53,161 192.168.2.10
   ```
   > Checks DNS (53) and SNMP (161); may require root privileges.

4. **Full TCP Port Sweep & Save Results**
   ```bash
   nmap -sS -p- -T4 -oA fullscan 192.168.2.10
   ```
   > Scans all ports quickly (`-T4`), outputs in Normal/XML/Grepable.

5. **Firewall Rule Mapping (ACK Scan)**
   ```bash
   nmap -sA -p 1-1000 192.168.2.10
   ```
   > Determines which ports the firewall filters vs. lets through.

6. **Stealth Evasion (Null Scan)**
   ```bash
   sudo nmap -sN -p 1-1024 192.168.2.10
   ```
   > May bypass simple packet filters; works on Unix/Linux systems.

7. **High-Performance Comprehensive Scan**
   ```bash
   nmap -sS -p- --min-rate 5000 --max-retries 1 -T4 192.168.2.10
   ```
   > Fast full port scan with aggressive timing.

8. **Service-Focused Scanning**
   ```bash
   nmap -sS -p 21,22,23,25,53,80,110,135,139,143,443,993,995,1433,3306,3389 192.168.2.0/24
   ```
   > Targets common services across a network range.

9. **UDP and TCP Combined Scan**
   ```bash
   nmap -sS -sU -p T:80,443,U:53,161,123 192.168.2.10
   ```
   > Scans both TCP and UDP ports simultaneously.

10. **Idle Scan for Maximum Stealth**
    ```bash
    nmap -sI zombie_host:113 target_host
    ```
    > Uses intermediate host for anonymous scanning.

#### Netcat Tip: Port Scan with nc

For very small scans or scripting, Netcat can check ports too:

```bash
nc -zv 192.168.2.10 22-25 80 443
```

**Command breakdown:**
- `-z` = Zero-I/O mode (just test connection, don't send data)
- `-v` = Verbose (show each connection attempt)
- `22-25` = Port range (SSH, Telnet, SMTP, alternate SMTP)
- `80 443` = Individual ports (HTTP, HTTPS)

Use it when you need a single-line port check without full Nmap output.

```bash
# Quick web server check
for ip in $(cat targets.txt); do nc -zv $ip 80 443 2>&1 | grep succeeded; done

# Check if SSH is open across a range
for i in {1..254}; do nc -zv 192.168.1.$i 22 2>&1 | grep succeeded & done; wait
```

> 💡 **Want to learn more Netcat techniques?** Check out our [Netcat Learning Guide](../netcat/) for comprehensive coverage of this versatile tool.

---

## 5. Interpreting Port States

| STATE          | Meaning                                                                  | Common Causes |
| -------------- | ------------------------------------------------------------------------ | ------------- |
| **open**       | Application is listening and responded (e.g., SYN-ACK)                  | Active service, daemon listening |
| **closed**     | No application; host sent RST to indicate closed port                   | No service running, port available |
| **filtered**   | No response (dropped) or ICMP unreachable—firewall likely blocking      | Firewall rules, packet filtering |
| **unfiltered** | ACK scan only: host responded → port reachable, but open/closed unknown | Stateless firewall, router ACL |
| **open\|filtered** | No response on stealth scan—could be open (no reply) or filtered     | Silent services, aggressive filtering |
| **closed\|filtered** | UDP scan only: no response could indicate closed or filtered port     | UDP nature, firewall policies |

### 5.1 Understanding TCP States in Detail

```text
PORT      STATE         SERVICE    REASON
22/tcp    open          ssh        syn-ack ttl 64
80/tcp    closed        http       reset ttl 64  
443/tcp   filtered      https      no-response
8080/tcp  open|filtered http-proxy no-response
```

**Detailed Analysis:**
- **syn-ack**: Port is open, service responded with SYN-ACK
- **reset**: Port is closed, host sent TCP reset (RST)
- **no-response**: Packets were dropped, likely by firewall
- **ttl value**: Indicates network hops/distance to target

### 5.2 Service Response Patterns
Different services exhibit characteristic response patterns:

- **Web servers**: Usually respond quickly to SYN scans
- **SSH/Telnet**: May have connection rate limiting  
- **Mail servers**: Often implement tarpit delays
- **Databases**: Frequently filtered, only accept from specific IPs
- **Administrative services**: Commonly filtered or bound to localhost

---

## 6. Hands-On Exercises

1. **Home Network Service Discovery**
   * Choose a home device (e.g., NAS, printer, or router):
   ```bash
   # Quick service discovery
   nmap -sS -F 192.168.1.1
   
   # Comprehensive port scan
   nmap -sS -p- --max-retries 1 192.168.1.1
   ```
   * Cross-reference discovered services with device documentation.

2. **Firewall Analysis Laboratory**
   * Set up a simple iptables firewall on a test system:
   ```bash
   # Allow SSH, HTTP, block others
   iptables -A INPUT -p tcp --dport 22 -j ACCEPT
   iptables -A INPUT -p tcp --dport 80 -j ACCEPT  
   iptables -A INPUT -p tcp --dport 1:65535 -j DROP
   ```
   * Test with different scan types:
   ```bash
   nmap -sS target     # SYN scan
   nmap -sA target     # ACK scan (firewall mapping)
   nmap -sF target     # FIN scan (evasion attempt)
   ```
   * Identify which ports are `filtered` vs. `unfiltered`.

3. **UDP vs TCP Service Comparison**
   * Scan for both TCP and UDP services:
   ```bash
   # TCP scan for common services
   nmap -sS -p 21,22,25,53,80,110,143,443,993,995 target
   
   # UDP scan for network services  
   nmap -sU -p 53,67,68,123,161,162,514 target
   ```
   * Compare results and note UDP-only services (DNS, DHCP, SNMP).

4. **Performance vs Accuracy Trade-offs**
   * Compare different timing templates:
   ```bash
   # Paranoid timing (very slow, accurate)
   time nmap -sS -T0 -p 1-100 target
   
   # Normal timing (balanced)
   time nmap -sS -T3 -p 1-100 target
   
   # Insane timing (fast, may miss results)
   time nmap -sS -T5 -p 1-100 target
   ```
   * Document speed vs. accuracy differences.

5. **Stealth Scanning Effectiveness**
   * Test different stealth techniques:
   ```bash
   # Standard SYN scan (baseline)
   nmap -sS target
   
   # Null scan (stealth attempt)
   nmap -sN target
   
   # FIN scan (firewall evasion)
   nmap -sF target
   
   # Fragmented packets
   nmap -sS -f target
   ```
   * Monitor with tcpdump/Wireshark to see packet differences.

6. **Large Network Efficient Scanning**
   * Optimize for scanning entire subnets:
   ```bash
   # Target common services across network
   nmap -sS -p 22,80,443 --min-hostgroup 50 192.168.1.0/24
   
   # High-speed discovery scan
   nmap -sS -F --min-rate 1000 10.0.0.0/16
   ```
   * Measure and optimize scan time vs. thoroughness.

---

## 7. Tips & Best Practices

### 7.1 Scanning Strategy
* **Start with fast scans** (`-F`) to identify active services quickly
* **Run comprehensive scans** (`-p-`) only on confirmed targets
* **Use appropriate timing** (`-T0` to `-T5`) based on network tolerance
* **Combine TCP and UDP** scanning for complete service discovery
* **Save all outputs** (`-oA`) for analysis and reporting

### 7.2 Performance Optimization  
* **Target specific ports** when looking for particular services
* **Use parallel scanning** (`--min-parallelism`) for large networks
* **Adjust retries** (`--max-retries`) based on network reliability
* **Set reasonable timeouts** (`--host-timeout`) for responsive scanning

### 7.3 Stealth and Detection Avoidance
* **Randomize scan order** (`--randomize-hosts`) to avoid patterns
* **Use source port spoofing** (`--source-port 53`) for common services
* **Fragment packets** (`-f`) to evade simple packet filters
* **Implement scan delays** (`--scan-delay`) to avoid rate limiting
* **Vary scan timing** to blend with normal network traffic

### 7.4 Legal and Ethical Guidelines
* **Own networks only**: Scan only systems you own or have explicit permission to test
* **Respect resources**: Avoid overwhelming target systems with aggressive scans
* **Document methodology**: Keep records of scanning techniques and results
* **Follow disclosure**: Report findings responsibly through appropriate channels

### 7.5 Integration with Other Tools
* **Feed results to vulnerability scanners**: Use XML output for automated processing
* **Combine with service detection**: Follow up with `-sV` on discovered ports
* **Export for reporting**: Convert results to readable formats for stakeholders
* **Automate workflows**: Script repetitive scanning tasks for efficiency
