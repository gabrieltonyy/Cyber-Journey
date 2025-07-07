# Phase 2: Port Scanning

**Objective**  
Discover which ports on a live host are **open**, **closed**, or **filtered**—forming the basis for deeper service and vulnerability analysis.

---

## 1. Why Port Scanning?

- **Identify Attack Surface**  
  Every open port represents a potential entry point for attackers.

- **Prioritize Services**  
  Focus on commonly exposed services (HTTP, SSH, DNS, etc.).

- **Measure Defenses**  
  "Filtered" ports indicate firewall or packet‐filter rules in action.

---

## 2. Scan Types & Key Options

| Scan Type              | Option         | Description                                                                 |
|------------------------|----------------|-----------------------------------------------------------------------------|
| **TCP SYN (Stealth)**  | `-sS`          | Sends SYN, waits for SYN-ACK (open) or RST (closed); does not complete handshake. |
| **TCP Connect**        | `-sT`          | Uses OS connect syscall; full handshake—easier to detect, needs no raw sockets. |
| **UDP Scan**           | `-sU`          | Sends empty UDP packets; no reply = open|filtered, ICMP port-unreachable = closed. |
| **ACK Scan**           | `-sA`          | Sends ACK; RST = unfiltered, no reply = filtered; used for firewall mapping. |
| **Null, FIN, Xmas**    | `-sN`/`-sF`/`-sX` | Sends unusual flag combinations; no reply = open|filtered (stealth evasion). |
| **Full Port Range**    | `-p-`          | Scans all 65,535 TCP ports (combined with `-sS` or `-sT`).                    |
| **Top Ports**          | `-F`           | Scan the 100 most common ports quickly.                                      |
| **Specific Ports**     | `-p 22,80,443` | Scan only listed ports or ranges (`-p 1-1024`).                              |
| **Idle Scan**          | `-sI`          | Uses a "zombie" host for blind scanning; very stealthy (advanced).           |

---

## 3. Examples

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
   > May bypass simple packet filters; Linux-only.

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

> 💡 **Want to learn more Netcat techniques?** Check out our [Netcat Learning Guide](../netcat/) for comprehensive coverage of this versatile tool.

---

## 4. Interpreting Port States

| STATE          | Meaning                                                                  |
| -------------- | ------------------------------------------------------------------------ |
| **open**       | Application is listening and responded (e.g., SYN-ACK).                  |
| **closed**     | No application; host sent RST to indicate closed port.                   |
| **filtered**   | No response (dropped) or ICMP unreachable—firewall likely blocking.      |
| **unfiltered** | ACK scan only: host responded → port reachable, but open/closed unknown. |
| **open\|filtered** | No response on stealth scan—could be open (no reply) or filtered. |

---

## 5. Hands-On Exercises

1. **Home Server Scan**
   * Choose a home device (e.g., NAS or IoT).
   * Run `nmap -sS <device-ip>` and note open ports.
   * Cross-reference service against device documentation.

2. **Firewall Audit**
   * On your bridged router (192.168.2.1), run:
     ```bash
     nmap -sA -p 1-500 192.168.2.1
     ```
   * Identify which ports are `filtered` vs. `unfiltered`.
   * Adjust firewall rules to explicitly block unused ports.

3. **UDP Discovery**
   * Find DNS and SNMP servers in your network:
     ```bash
     sudo nmap -sU -p 53,161 192.168.1.0/24
     ```
   * Verify with `dig` (for DNS) and `snmpwalk`.

4. **Complete Port Range**
   * On Metasploitable2, sweep all TCP ports:
     ```bash
     nmap -p- -T3 <metasploitable-ip>
     ```
   * Compare against top-ports scan (`-F`) to see what you miss.

---

## 6. Tips & Best Practices

* **Run as root/admin** for raw scans (`-sS`, `-sU`, `-sN`, etc.).
* **Use timing templates** (`-T0` to `-T5`) to balance speed vs. stealth.
* **Save outputs** (`-oA`, `-oX`, `-oG`) for reporting and automation.
* **Combine with grep** for parsing:
  ```bash
  nmap -sS <ip> -oG - | grep "/open/"
  ```
* **Scan responsibly**: only on systems you own or have permission to test.
