# Phase 3: Version & Service Detection

**Objective**  
Go beyond "which ports are open" to discover **what software** is running on each port, **which versions**, and optionally **default scripts** to gather additional service info.

---

## 1. Why Version & Service Detection?

- **Vulnerability Assessment**  
  Exact version numbers let you map services to known CVEs.

- **Asset Inventory**  
  Know exactly what software (and patch levels) your hosts run.

- **Attack Planning**  
  Identify outdated or unpatched services that may be exploitable.

- **Automation**  
  Feed versioned data into vulnerability scanners or patch management.

---

## 2. Key NSE & Scan Options

| Option                          | Description                                                             |
|---------------------------------|-------------------------------------------------------------------------|
| `-sV`                           | Probe open ports and match service/version from Nmap's probe database.  |
| `--version-intensity <0–9>`     | Control number of probes. Higher = deeper detection (slower).           |
| `--version-light`               | Shorthand for `--version-intensity 2` (fewer probes, faster).           |
| `--version-all`                 | Shorthand for `--version-intensity 9` (all probes, most aggressive).    |
| `-A`                            | "Aggressive" = `-sV -O -sC --traceroute` all in one.                    |
| `-sC` or `--script=default`     | Run default (safe) scripts, including some service probes.              |
| `--script=banner`               | Grab raw banners from services that support them (SMTP, HTTP, etc.).    |
| `--script=ssl-cert`             | Enumerate SSL cert details (common names, SAN, issuer, validity).       |
| `--script=ssl-enum-ciphers`     | List and grade supported TLS cipher suites.                             |

---

## 3. Examples

1. **Basic Version Scan**  
   ```bash
   nmap -sV 192.168.2.10
   ```
   > Returns a table of open ports and identified service names & versions.

2. **Deep Version Detection**
   ```bash
   nmap -sV --version-all 192.168.2.10
   ```
   > Sends every probe, may reveal obscure or non-standard banners.

3. **Light Version Scan**
   ```bash
   nmap -sV --version-light 10.0.0.5
   ```
   > Faster, less comprehensive, good for large networks.

4. **Service + Default Scripts**
   ```bash
   nmap -sC -sV 10.0.0.20
   ```
   > Runs default scripts (`http-title`, `ssh-hostkey`, etc.) plus version detection.

5. **SSL Certificate Inspection**
   ```bash
   nmap -p 443 --script ssl-cert,ssl-enum-ciphers example.com
   ```
   > Retrieves certificate details and grades cipher strength.

6. **Banner Grabbing**
   ```bash
   nmap -sV --script banner -p 25,110,143 mail.example.com
   ```
   > Grabs raw service banners from SMTP, POP3, IMAP.

#### Netcat Tip: Manual Banner Grab

When `-sV` reports "unrecognized", open a raw connection:

```bash
printf "GET / HTTP/1.0\r\n\r\n" | nc target.com 80
```

**Command breakdown:**
- `printf` = Send formatted HTTP request
- `GET / HTTP/1.0\r\n\r\n` = Basic HTTP GET request with proper line endings
- `|` = Pipe the request to netcat
- `nc target.com 80` = Connect to target on port 80

Or grab an SMTP banner interactively:

```bash
nc target.com 25
HELO example.com
QUIT
```

**What happens:**
1. `nc target.com 25` = Connect to SMTP service
2. Server sends banner automatically
3. `HELO example.com` = Identify yourself to SMTP server
4. `QUIT` = Close connection gracefully

This can reveal custom headers or hidden version strings that Nmap's probes might miss.

> 💡 **Want to learn more Netcat techniques?** Check out our [Netcat Learning Guide](../netcat/) for comprehensive coverage of this versatile tool.

---

## 4. Interpreting Results

```text
PORT    STATE SERVICE    VERSION
22/tcp  open  ssh        OpenSSH 8.4p1 Debian
80/tcp  open  http       Apache httpd 2.4.41 ((Ubuntu))
143/tcp open  imap       Dovecot imapd 2.3.4
443/tcp open  ssl/http   nginx 1.18.0 (TLSv1.2; TLSv1.3)
```

* **Service**: Matched from Nmap's `nmap-service-probes` database.
* **Version**: Exact software version string.
* **Extra info**: Parentheses often show OS or distribution.

**Unrecognized services** appear as `unknown` or simply `http` without version. Use `--version-intensity 9` or manual banner grabbing to clarify.

---

## 5. Hands-On Exercises

1. **Metasploitable2 Version Audit**
   ```bash
   nmap -sV -p 21,22,23,80,139,445 <metasploitable-ip>
   ```
   * Note each version and lookup one CVE per service.

2. **Ubuntu Server Scan**
   * Spin up a fresh Ubuntu VM.
   * Patch it fully, then run:
     ```bash
     nmap -sV <ubuntu-ip>
     ```
   * Compare results before & after patching.

3. **Custom App on HTTP**
   * Deploy a simple HTTP server (e.g., Python `http.server`).
   * Run `nmap -sV` and observe how Nmap identifies it ("python SimpleHTTPServer").

4. **SSL Hardening Check**
   * Host a test site with both TLS 1.2 and 1.3 enabled.
   * Use:
     ```bash
     nmap -p443 --script ssl-enum-ciphers <test-ip>
     ```
   * Identify any weak ciphers and modify your server config to remove them.

---

## 6. Tips & Best Practices

* **Run version scans only on known open ports** to save time: `-p 22,80,443`.
* **Combine `-sV` with `-oA`** to save outputs for later parsing.
* **Automate CVE lookup** with tools like `searchsploit` or `vulners.nse`.
* **Keep `nmap-service-probes` up to date** via `nmap --script-updatedb`.
* **Use `--version-intensity`** to balance speed vs. depth based on context.
