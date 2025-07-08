# Phase 3: Version & Service Detection

**Objective**  
Go beyond "which ports are open" to discover **what software** is running on each port, **which versions**, and gather detailed service information for vulnerability assessment and asset inventory.

---

## 1. Why Version & Service Detection?

- **Vulnerability Assessment**  
  Exact version numbers enable precise CVE mapping and exploit identification.

- **Asset Inventory**  
  Maintain accurate records of software versions and patch levels across your infrastructure.

- **Attack Vector Analysis**  
  Identify outdated, unpatched, or misconfigured services that may be exploitable.

- **Compliance Monitoring**  
  Verify software versions meet organizational security and compliance requirements.

- **Automation Integration**  
  Feed versioned data into vulnerability scanners, patch management, and SIEM systems.

- **Configuration Assessment**  
  Detect default configurations, weak SSL/TLS settings, and exposed administrative interfaces.

---

## 2. Key NSE & Scan Options

| Option                          | Description                                                             | Performance Impact |
|---------------------------------|-------------------------------------------------------------------------|-------------------|
| `-sV`                           | Probe open ports and match service/version from Nmap's probe database  | Medium |
| `--version-intensity <0–9>`     | Control number of probes. Higher = deeper detection (slower)           | 0=Fast, 9=Thorough |
| `--version-light`               | Shorthand for `--version-intensity 2` (fewer probes, faster)           | Low |
| `--version-all`                 | Shorthand for `--version-intensity 9` (all probes, most aggressive)    | High |
| `-A`                            | "Aggressive" = `-sV -O -sC --traceroute` all in one                    | High |
| `-sC` or `--script=default`     | Run default (safe) scripts, including some service probes              | Medium |
| `--script=banner`               | Grab raw banners from services that support them (SMTP, HTTP, etc.)    | Low |
| `--script=ssl-cert`             | Enumerate SSL cert details (common names, SAN, issuer, validity)       | Low |
| `--script=ssl-enum-ciphers`     | List and grade supported TLS cipher suites                             | Medium |
| `--script=http-title`           | Extract HTTP page titles and server headers                            | Low |
| `--script=smtp-commands`        | Enumerate SMTP server capabilities and commands                         | Low |
| `--script=ssh-hostkey`          | Extract SSH server host keys and algorithms                             | Low |

---

## 3. Advanced Service Detection Techniques

### 3.1 Service-Specific Enumeration
```bash
# Web application fingerprinting
nmap -p80,443 --script http-title,http-server-header,http-methods,http-robots.txt target

# Database version detection
nmap -p1433,3306,5432 --script ms-sql-info,mysql-info,pgsql-databases target

# Mail server enumeration
nmap -p25,110,143,993,995 --script smtp-commands,pop3-capabilities,imap-capabilities target

# SSH service analysis
nmap -p22 --script ssh-hostkey,ssh2-enum-algos,ssh-auth-methods target

# FTP service detection
nmap -p21 --script ftp-anon,ftp-bounce,ftp-syst target
```

### 3.2 SSL/TLS Security Assessment
```bash
# Comprehensive SSL analysis
nmap -p443,993,995 --script ssl-cert,ssl-enum-ciphers,ssl-heartbleed,ssl-poodle target

# Certificate chain analysis
nmap -p443 --script ssl-cert-intaddr,ssl-known-key target

# TLS configuration assessment
nmap -p443 --script ssl-dh-params,tls-nextprotoneg target
```

### 3.3 Custom Service Probing
```bash
# Custom probe with specific intensity
nmap -sV --version-intensity 7 -p 1-1000 target

# Version detection with custom timeout
nmap -sV --version-intensity 5 --host-timeout 10m target

# Combine version detection with vulnerability scanning
nmap -sV --script vuln -p open target
```

---

## 4. Examples

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

7. **Comprehensive Web Server Analysis**
   ```bash
   nmap -p80,443 -sV --script http-title,http-server-header,http-methods,http-enum target
   ```
   > Detailed web server fingerprinting and directory enumeration.

8. **Database Service Detection**
   ```bash
   nmap -p1433,3306,5432 -sV --script ms-sql-info,mysql-info,pgsql-databases target
   ```
   > Identifies database types, versions, and accessible databases.

9. **Mail Server Comprehensive Scan**
   ```bash
   nmap -p25,465,587,993,995 -sV --script smtp-commands,smtp-enum-users target
   ```
   > Mail server version detection with command enumeration.

10. **Custom Intensity Scanning**
    ```bash
    nmap -sV --version-intensity 6 -p 1-10000 target
    ```
    > Balanced approach between speed and thoroughness.

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

**Advanced banner grabbing techniques:**
```bash
# FTP banner and capabilities
echo -e "USER anonymous\nQUIT" | nc target.com 21

# SSH version string  
nc target.com 22 | head -1

# DNS version (if allowed)
echo -e "version.bind\nquit" | nc target.com 53

# SNMP community string testing
echo -e "public\nquit" | nc -u target.com 161
```

This can reveal custom headers or hidden version strings that Nmap's probes might miss.

> 💡 **Want to learn more Netcat techniques?** Check out our [Netcat Learning Guide](../netcat/) for comprehensive coverage of this versatile tool.

---

## 5. Interpreting Results

```text
PORT    STATE SERVICE    VERSION
22/tcp  open  ssh        OpenSSH 8.4p1 Debian 5+deb11u1 (SSH-2.0-OpenSSH_8.4p1)
80/tcp  open  http       Apache httpd 2.4.54 ((Ubuntu))
143/tcp open  imap       Dovecot imapd 2.3.16 (Ubuntu)
443/tcp open  ssl/http   nginx 1.18.0 (Ubuntu) (TLSv1.2; TLSv1.3)
3306/tcp open mysql      MySQL 8.0.32-0ubuntu0.22.04.2
```

**Detailed Analysis:**

* **Service**: Matched from Nmap's `nmap-service-probes` database
* **Version**: Exact software version string extracted from banner/probe response
* **OS Information**: Parentheses often show operating system or distribution details  
* **Protocol Details**: Additional protocol information (SSH version, TLS versions)
* **Extra Information**: May include patch levels, build information, or configuration details

### 5.1 Common Service Patterns

**Web Servers:**
```text
80/tcp  open  http       Apache httpd 2.4.41 ((Ubuntu))
443/tcp open  ssl/http   nginx 1.18.0 (Ubuntu)
8080/tcp open http-proxy Apache Tomcat 9.0.65
```

**Database Services:**
```text  
1433/tcp open ms-sql-s   Microsoft SQL Server 2019 15.00.4153.10
3306/tcp open mysql      MySQL 8.0.32-0ubuntu0.22.04.2
5432/tcp open postgresql PostgreSQL DB 14.6 (Ubuntu 14.6-0ubuntu0.22.04.1)
```

**Mail Services:**
```text
25/tcp  open smtp        Postfix smtpd 3.6.4
110/tcp open pop3        Dovecot pop3d 2.3.16
993/tcp open ssl/imap    Dovecot imapd 2.3.16
```

### 5.2 Handling Unrecognized Services

**Unrecognized services** appear as `unknown` or generic service names:
```text
8888/tcp open  unknown
9999/tcp open  abyss?
10000/tcp open snet-sensor-mgmt?
```

**Resolution strategies:**
- Use `--version-intensity 9` for maximum probe coverage
- Perform manual banner grabbing with netcat
- Research port numbers in service databases
- Use custom NSE scripts for specific applications
- Check application documentation for default ports

### 5.3 SSL/TLS Certificate Analysis

When using `--script ssl-cert`:
```text
| ssl-cert: Subject: commonName=example.com/organizationName=Example Corp
| Subject Alternative Name: DNS:example.com, DNS:www.example.com
| Issuer: commonName=Let's Encrypt Authority X3
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-01-01T00:00:00
| Not valid after:  2023-04-01T00:00:00
```

**Key security indicators:**
- **Expiration dates**: Check for expired or soon-to-expire certificates
- **Signature algorithm**: Identify weak algorithms (MD5, SHA1)
- **Key size**: Flag weak key sizes (<2048 bits for RSA)
- **Subject Alternative Names**: Verify all intended domains are covered
- **Certificate chain**: Ensure proper CA validation

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
