# Phase 1: Host Discovery

**Objective**: Identify which IP addresses on a network are "up" (live) before scanning ports.

---

## 1. Why Host Discovery?

- **Efficiency**: Skip offline hosts and focus on live systems.  
- **Stealth**: Quickly map active devices without probing every port.  
- **Baseline**: Build your inventory of target machines.

---

## 2. Key Techniques & Commands

| Technique                  | Command                                               | Description                                    |
|----------------------------|-------------------------------------------------------|------------------------------------------------|
| **List Scan (DNS only)**   | `nmap -sL 192.168.1.0/24`                             | No packets sent; lists targets and DNS names.  |
| **Ping Scan**              | `nmap -sn 10.0.0.0/24`                                | ICMP & TCP "ping" probes; shows live hosts only. |
| **No Ping (treat all up)** | `nmap -Pn 192.168.1.1-50`                             | Skips discovery; treats targets as up.         |
| **ARP Ping (LAN only)**    | `nmap -PR 192.168.1.0/24`                             | ARP requests on local network (very reliable). |
| **IPv6 Host Discovery**    | `nmap -6 -sn fe80::/64`                               | Ping-scan IPv6 link-local range.               |

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

## 4. Hands-On Exercises

1. **Exercise: Home Network Sweep**
   * Bridge a spare router as a Wi-Fi extender.
   * Run `nmap -sn <extender-subnet>/24` to map all connected devices.
   * Compare results to your router's DHCP client list.

2. **Exercise: IPv6 Discovery**
   * If your network supports IPv6, pick your link-local range (e.g., `fe80::/64`).
   * Run `nmap -6 -sn fe80::/64`.
   * Identify any IPv6-only devices (IoT, printers, etc.).

3. **Exercise: DNS List Verification**
   * Create a file `hosts.txt` with a mix of internal hostnames and bogus names.
   * Run `nmap -sL -iL hosts.txt`.
   * Verify which names resolve and which entries are typos.

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
