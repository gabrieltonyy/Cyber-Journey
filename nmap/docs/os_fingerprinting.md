# Phase 4: OS Fingerprinting

**Objective**  
Determine the target's operating system (and sometimes device type/version) by analyzing subtle differences in network stack behavior.

---

## 1. Why OS Fingerprinting?

- **Attack Validation**  
  Tailor exploits or payloads for specific OS versions.

- **Asset Inventory**  
  Know whether hosts run Windows, Linux, macOS, network appliances, etc.

- **Defense Planning**  
  Apply OS-specific hardening (patches, firewall rules, configuration).

- **Incident Response**  
  Quickly classify compromised hosts by OS for cleanup.

---

## 2. How OS Fingerprinting Works

Nmap sends a series of TCP/IP probes (using raw packets) that test:

- **TCP options** (window size, MSS, scale, timestamps)  
- **TCP flags** and sequence quirks  
- **ICMP request/reply variations**  
- **IP ID generation patterns**  
- **Initial TTL values**  

It compares responses against a database of known fingerprints to guess the OS.

---

## 3. Key Options & Scripts

| Option                     | Description                                                             |
|----------------------------|-------------------------------------------------------------------------|
| `-O`                       | Enable OS detection.                                                   |
| `--osscan-guess`           | If no exact match, guess best fit.                                      |
| `--osscan-limit`           | Only run OS detection if at least one open/closed port found.           |
| `-A`                       | Includes `-O` (and `-sV`, `-sC`, traceroute).                           |
| `--script=firewalk`        | Use Firewalk technique to map firewall rules (advanced).                |
| `--script=os-fingerprint`  | Dump the raw fingerprint details.                                       |

---

## 4. Examples

1. **Basic OS Detection**  
   ```bash
   sudo nmap -O 192.168.2.10
   ```
   > Outputs guessed OS, accuracy percentage, network distance.

2. **Aggressive Scan with OS**
   ```bash
   sudo nmap -A 192.168.2.10
   ```
   > Performs version detection, default scripts, traceroute, and OS detection.

3. **Limit OS Scans to Likely Targets**
   ```bash
   sudo nmap -O --osscan-limit 10.0.0.0/24
   ```
   > Only attempts OS detection on hosts with at least one open port.

4. **Force OS Guessing**
   ```bash
   sudo nmap -O --osscan-guess <target>
   ```
   > Provides probable OS matches when no exact fingerprint found.

5. **View Raw Fingerprint**
   ```bash
   sudo nmap --script=os-fingerprint -O <target>
   ```
   > Shows the fingerprint data sent and received for custom analysis.

---

## 5. Interpreting OS Detection Output

```text
OS details: Linux 3.2 – 4.9 (92%)
Network Distance: 1 hop
```

* **OS details**: Lists likely OS family/version and confidence.
* **Network Distance**: Number of hops (TTL-based).

If accuracy is low (<70%), consider:

* More open/closed ports for fingerprinting.
* Running against different network segments.
* Using `--osscan-guess` to see next-best matches.

---

## 6. Hands-On Exercises

1. **Metasploitable2 OS Guess**
   ```bash
   sudo nmap -O <metasploitable-ip>
   ```
   * Verify if Nmap correctly identifies Linux.
   * Try `--osscan-guess` to observe alternatives.

2. **Windows VM Detection**
   * Spin up a Windows Server VM.
   * Run:
     ```bash
     sudo nmap -O <windows-ip>
     ```
   * Note TTL and fingerprint differences vs. Linux.

3. **Firewall Impact**
   * On a host behind a simple firewall, run:
     ```bash
     sudo nmap -O <firewalled-ip>
     ```
   * Observe how filtering affects accuracy; use `--osscan-guess`.

4. **Network Appliance**
   * If available, target a router or IoT device:
     ```bash
     sudo nmap -O <router-ip>
     ```
   * Compare TTL hop count and reported OS.

---

## 7. Tips & Best Practices

* **Run as root/Administrator** for raw packet privileges.
* **Ensure at least one open and one closed port** to improve fingerprinting.
* **Use `--osscan-limit`** on large networks for efficiency.
* **Combine with `-v`** to see fingerprinting progress:`nmap -v -O <target>`.
* **Keep Nmap's OS database updated** (`nmap --script-updatedb`).

#### Netcat Note

OS fingerprinting requires raw sockets and specialized TCP/IP probe techniques; Netcat won't help here beyond basic service checks—stick with Nmap for OS detection and guessing.

> 💡 **Want to learn more Netcat techniques?** Check out our [Netcat Learning Guide](../netcat/) for comprehensive coverage of this versatile tool.
