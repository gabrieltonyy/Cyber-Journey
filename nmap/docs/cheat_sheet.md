# Nmap Command Reference & Cheat Sheet

**Quick reference for all Nmap commands, flags, and techniques**

---

## Target Specification

| Command | Example | Description |
|---------|---------|-------------|
| Single IP | `nmap 192.168.1.1` | Scan a single IP address |
| Multiple IPs | `nmap 192.168.1.1 192.168.2.1` | Scan specific IP addresses |
| IP Range | `nmap 192.168.1.1-254` | Scan a range of IPs |
| Domain | `nmap scanme.nmap.org` | Scan a domain name |
| CIDR Notation | `nmap 192.168.1.0/24` | Scan using CIDR notation |
| Target File | `nmap -iL targets.txt` | Scan targets from a file |
| Random Hosts | `nmap -iR 100` | Scan 100 random hosts |
| Exclude Hosts | `nmap --exclude 192.168.1.1` | Exclude listed hosts |
| Exclude File | `nmap --excludefile exclude.txt 192.168.1.0/24` | Exclude targets from file |
| IPv6 Scan | `nmap -6 2607:f0d0:1002:51::4` | Scan IPv6 targets |

---

## Scan Techniques

| Flag | Example | Description |
|------|---------|-------------|
| `-sS` | `nmap -sS 192.168.1.1` | TCP SYN scan (stealth, default) |
| `-sT` | `nmap -sT 192.168.1.1` | TCP connect scan (no raw sockets needed) |
| `-sU` | `nmap -sU 192.168.1.1` | UDP port scan |
| `-sA` | `nmap -sA 192.168.1.1` | TCP ACK scan (firewall testing) |
| `-sW` | `nmap -sW 192.168.1.1` | TCP Window scan |
| `-sM` | `nmap -sM 192.168.1.1` | TCP Maimon scan |
| `-sN` | `nmap -sN 192.168.1.1` | Null scan (no flags set) |
| `-sF` | `nmap -sF 192.168.1.1` | FIN scan |
| `-sX` | `nmap -sX 192.168.1.1` | Xmas scan (FIN, PSH, URG flags) |
| `-sY` | `nmap -sY 192.168.1.1` | SCTP INIT scan |
| `-sZ` | `nmap -sZ 192.168.1.1` | SCTP COOKIE-ECHO scan |
| `-sI` | `nmap -sI zombie:port 192.168.1.1` | Idle scan (zombie host required) |
| `-sO` | `nmap -sO 192.168.1.1` | IP protocol scan |

---

## Host Discovery

| Flag | Example | Description |
|------|---------|-------------|
| `-sL` | `nmap -sL 192.168.1.1-3` | List scan (DNS resolution only) |
| `-sn` | `nmap -sn 192.168.1.0/24` | Ping scan (disable port scan) |
| `-Pn` | `nmap -Pn 192.168.1.1-5` | No ping (treat all hosts as up) |
| `-PS` | `nmap -PS22-25,80 192.168.1.1` | TCP SYN discovery on specific ports |
| `-PA` | `nmap -PA22-25,80 192.168.1.1` | TCP ACK discovery on specific ports |
| `-PU` | `nmap -PU53 192.168.1.1` | UDP discovery on port 53 |
| `-PR` | `nmap -PR 192.168.1.0/24` | ARP discovery (LAN only) |
| `-n` | `nmap -n 192.168.1.1` | No DNS resolution |
| `-R` | `nmap -R 192.168.1.1` | Always resolve DNS |

---

## Port Specification

| Flag | Example | Description |
|------|---------|-------------|
| `-p` | `nmap -p 21 192.168.1.1` | Scan specific port |
| `-p` | `nmap -p 21-100 192.168.1.1` | Port range |
| `-p` | `nmap -p U:53,T:21-25,80 192.168.1.1` | TCP and UDP ports |
| `-p-` | `nmap -p- 192.168.1.1` | All ports (1-65535) |
| `-p` | `nmap -p http,https 192.168.1.1` | Scan by service name |
| `-F` | `nmap -F 192.168.1.1` | Fast scan (top 100 ports) |
| `--top-ports` | `nmap --top-ports 2000 192.168.1.1` | Scan top N ports |
| `-p0-` | `nmap -p0- 192.168.1.1` | Include port 0 in scan |
| `-r` | `nmap -r 192.168.1.1` | Scan ports sequentially (not random) |

---

## Service & Version Detection

| Flag | Example | Description |
|------|---------|-------------|
| `-sV` | `nmap -sV 192.168.1.1` | Version detection |
| `--version-intensity` | `nmap -sV --version-intensity 8 192.168.1.1` | Set intensity (0-9) |
| `--version-light` | `nmap -sV --version-light 192.168.1.1` | Light mode (faster) |
| `--version-all` | `nmap -sV --version-all 192.168.1.1` | Try all probes |
| `-A` | `nmap -A 192.168.1.1` | Aggressive scan (OS, version, scripts, traceroute) |

---

## OS Detection

| Flag | Example | Description |
|------|---------|-------------|
| `-O` | `nmap -O 192.168.1.1` | OS detection |
| `--osscan-limit` | `nmap -O --osscan-limit 192.168.1.1` | Limit to promising targets |
| `--osscan-guess` | `nmap -O --osscan-guess 192.168.1.1` | Guess OS more aggressively |
| `--max-os-tries` | `nmap -O --max-os-tries 1 192.168.1.1` | Limit OS detection attempts |

---

## Timing & Performance

| Flag | Example | Description |
|------|---------|-------------|
| `-T0` | `nmap -T0 192.168.1.1` | Paranoid (slowest) |
| `-T1` | `nmap -T1 192.168.1.1` | Sneaky |
| `-T2` | `nmap -T2 192.168.1.1` | Polite |
| `-T3` | `nmap -T3 192.168.1.1` | Normal (default) |
| `-T4` | `nmap -T4 192.168.1.1` | Aggressive |
| `-T5` | `nmap -T5 192.168.1.1` | Insane (fastest) |

### Fine-Tuning Options

| Flag | Example | Description |
|------|---------|-------------|
| `--min-rate` | `nmap --min-rate 100 192.168.1.1` | Minimum packets per second |
| `--max-rate` | `nmap --max-rate 100 192.168.1.1` | Maximum packets per second |
| `--max-retries` | `nmap --max-retries 3 192.168.1.1` | Maximum probe retransmissions |
| `--host-timeout` | `nmap --host-timeout 30m 192.168.1.1` | Give up on slow hosts |
| `--min-hostgroup` | `nmap --min-hostgroup 50 192.168.1.0/24` | Parallel host scan group size |
| `--max-hostgroup` | `nmap --max-hostgroup 100 192.168.1.0/24` | Maximum parallel hosts |

---

## NSE Scripts

| Flag | Example | Description |
|------|---------|-------------|
| `-sC` | `nmap -sC 192.168.1.1` | Default scripts |
| `--script=default` | `nmap --script=default 192.168.1.1` | Default scripts (explicit) |
| `--script` | `nmap --script=banner 192.168.1.1` | Specific script |
| `--script` | `nmap --script=http* 192.168.1.1` | Script category wildcard |
| `--script` | `nmap --script=http,banner 192.168.1.1` | Multiple scripts |
| `--script` | `nmap --script "not intrusive" 192.168.1.1` | Exclude script categories |
| `--script-args` | `nmap --script snmp-sysdescr --script-args snmpcommunity=admin 192.168.1.1` | Script arguments |

### Useful Script Categories

- `auth` - Authentication bypass and brute force
- `broadcast` - Network broadcast/multicast discovery  
- `brute` - Brute force attacks
- `default` - Default scripts (safe)
- `discovery` - Service and host discovery
- `dos` - Denial of service attacks
- `exploit` - Known vulnerability exploits
- `external` - Scripts requiring external resources
- `fuzzer` - Fuzzing scripts
- `intrusive` - Intrusive scripts
- `malware` - Malware detection
- `safe` - Safe scripts
- `version` - Version detection enhancement
- `vuln` - Vulnerability detection

### Useful NSE Script Examples

| Script | Command | Description |
|--------|---------|-------------|
| HTTP Site Map | `nmap --script http-sitemap-generator target.com` | Generate website sitemap |
| DNS Brute Force | `nmap --script dns-brute domain.com` | Brute force DNS hostnames |
| SMB Enumeration | `nmap --script smb-enum* 192.168.1.1` | Comprehensive SMB enumeration |
| Whois Lookup | `nmap --script whois* domain.com` | Domain whois information |
| SQL Injection | `nmap -p80 --script http-sql-injection target.com` | Check for SQL injection vulns |
| XSS Detection | `nmap -p80 --script http-unsafe-output-escaping target.com` | Detect XSS vulnerabilities |
| SSL Certificate | `nmap -p443 --script ssl-cert target.com` | SSL certificate details |
| SNMP Info | `nmap --script snmp-sysdescr --script-args snmpcommunity=public target` | SNMP system description |

---

## Firewall/IDS Evasion

| Flag | Example | Description |
|------|---------|-------------|
| `-f` | `nmap -f 192.168.1.1` | Fragment packets |
| `--mtu` | `nmap --mtu 32 192.168.1.1` | Set custom MTU |
| `-D` | `nmap -D RND:10 192.168.1.1` | Decoy scan with random IPs |
| `-D` | `nmap -D 1.2.3.4,ME,5.6.7.8 192.168.1.1` | Decoy scan with specific IPs |
| `-S` | `nmap -S 1.2.3.4 192.168.1.1` | Spoof source IP |
| `-e` | `nmap -e eth0 192.168.1.1` | Use specific interface |
| `-g` | `nmap -g 53 192.168.1.1` | Spoof source port |
| `--source-port` | `nmap --source-port 53 192.168.1.1` | Spoof source port (alternative) |
| `--data` | `nmap --data 0xdeadbeef 192.168.1.1` | Append custom binary data |
| `--data-string` | `nmap --data-string "string" 192.168.1.1` | Append custom string data |
| `--data-length` | `nmap --data-length 200 192.168.1.1` | Append random data |
| `--spoof-mac` | `nmap --spoof-mac 0 192.168.1.1` | Spoof MAC address |
| `--spoof-mac` | `nmap --spoof-mac Dell 192.168.1.1` | Spoof MAC with vendor |
| `--badsum` | `nmap --badsum 192.168.1.1` | Bad checksum |
| `--proxies` | `nmap --proxies http://proxy:8080 192.168.1.1` | Use proxy |
| `--scan-delay` | `nmap --scan-delay 1s 192.168.1.1` | Add delay between probes |
| `--randomize-hosts` | `nmap --randomize-hosts 192.168.1.0/24` | Randomize target order |

### Complete Evasion Example
```bash
nmap -f -T1 -n -Pn --data-length 200 -D 192.168.1.101,192.168.1.102,ME,192.168.1.104 192.168.1.1
```

### Advanced Evasion Examples
```bash
# Ultra-stealth scan with all evasion techniques
nmap -sS -f -T0 --scan-delay 5s --randomize-hosts --data-length 25 \
     --spoof-mac Dell -D RND:5 --source-port 53 192.168.1.0/24

# Firewall bypass using common service ports
nmap -sS --source-port 80 -g 443 192.168.1.1

# TCP sequence prediction evasion
nmap --badsum --ip-options "R" 192.168.1.1
```

---

## Output Options

| Flag | Example | Description |
|------|---------|-------------|
| `-oN` | `nmap -oN scan.txt 192.168.1.1` | Normal output |
| `-oX` | `nmap -oX scan.xml 192.168.1.1` | XML output |
| `-oG` | `nmap -oG scan.grep 192.168.1.1` | Grepable output |
| `-oA` | `nmap -oA scan 192.168.1.1` | All output formats |
| `-oS` | `nmap -oS scan.skid 192.168.1.1` | Script kiddie output |
| `-v` | `nmap -v 192.168.1.1` | Verbose output |
| `-vv` | `nmap -vv 192.168.1.1` | Very verbose |
| `-d` | `nmap -d 192.168.1.1` | Debug output |
| `-dd` | `nmap -dd 192.168.1.1` | More debug output |
| `--reason` | `nmap --reason 192.168.1.1` | Show reason for port state |
| `--open` | `nmap --open 192.168.1.1` | Show only open ports |
| `--packet-trace` | `nmap --packet-trace 192.168.1.1` | Show packet details |
| `--append-output` | `nmap --append-output -oN scan.txt 192.168.1.1` | Append to existing file |
| `--resume` | `nmap --resume results.file` | Resume interrupted scan |
| `--stats-every` | `nmap --stats-every 10s 192.168.1.1` | Show stats every N seconds |
| `--log-errors` | `nmap --log-errors errors.log 192.168.1.1` | Log errors to file |

---

## Useful Command Combinations

### Quick Network Discovery
```bash
nmap -sn 192.168.1.0/24
```

### Fast Port Scan
```bash
nmap -F -T4 192.168.1.1
```

### Comprehensive Scan
```bash
nmap -A -T4 -oA comprehensive_scan 192.168.1.1
```

### Stealth Scan
```bash
nmap -sS -T1 -f --data-length 200 192.168.1.1
```

### Vulnerability Scan
```bash
nmap -sV --script vuln 192.168.1.1
```

### UDP Service Discovery
```bash
sudo nmap -sU --top-ports 1000 192.168.1.1
```

### Web Server Enumeration
```bash
nmap -p 80,443 -sV --script http-enum,http-title 192.168.1.1
```

### SMB Enumeration
```bash
nmap -p 445 --script smb-enum-shares,smb-enum-users 192.168.1.1
```

---

## Useful Output Processing

### Extract Live Hosts
```bash
nmap -sn 192.168.1.0/24 | grep "Nmap scan report" | cut -d " " -f5
```

### Extract Open Ports
```bash
nmap -p- --open 192.168.1.1 | grep "^[0-9]" | cut -d '/' -f1
```

### Compare Scans
```bash
ndiff scan1.xml scan2.xml
```

### Convert XML to HTML
```bash
xsltproc nmap.xml -o nmap.html
```

### Parse XML with xmlstarlet
```bash
xmlstarlet sel -t -v "//host/address/@addr" -n scan.xml
```

### Extract services from grepable output
```bash
grep "/open/" scan.grep | cut -d' ' -f2,3
```

---

## Advanced Scanning Techniques

### Network Discovery & Reconnaissance
```bash
# Comprehensive network discovery
nmap -sn --traceroute --script broadcast-dhcp-discover,broadcast-dns-service-discovery 192.168.1.0/24

# IPv6 neighbor discovery
nmap -6 --script targets-ipv6-multicast-echo,targets-ipv6-multicast-invalid-dst fe80::/64

# Gateway and router discovery
nmap --script broadcast-listener --script-args=newtargets 192.168.1.0/24
```

### Service-Specific Enumeration
```bash
# Web application scanning
nmap -p80,443 --script http-enum,http-headers,http-methods,http-title target.com

# Database enumeration
nmap -p1433,3306,5432 --script ms-sql-info,mysql-info,pgsql-databases target

# Mail server enumeration  
nmap -p25,110,143,993,995 --script smtp-commands,pop3-capabilities,imap-capabilities target

# SMB/NetBIOS enumeration
nmap -p445 --script smb-protocols,smb-security-mode,smb-os-discovery target

# DNS enumeration
nmap -p53 --script dns-zone-transfer,dns-recursion,dns-cache-snoop target
```

### Vulnerability Assessment
```bash
# CVE vulnerability scanning
nmap --script vuln,exploit --script-args vulns.showall target

# SSL/TLS security assessment
nmap -p443 --script ssl-cert,ssl-enum-ciphers,ssl-heartbleed,ssl-poodle target

# Web vulnerability scanning
nmap -p80,443 --script http-sql-injection,http-stored-xss,http-dombased-xss target
```

### Performance Optimization
```bash
# High-speed scanning for large networks
nmap -T5 --min-rate 1000 --max-retries 1 --host-timeout 5m 10.0.0.0/8

# Memory-efficient scanning
nmap --max-hostgroup 10 --min-hostgroup 1 large-network.txt

# Parallel port scanning
nmap --min-parallelism 100 --max-parallelism 300 target
```

---

## Miscellaneous Options

| Flag | Example | Description |
|------|---------|-------------|
| `-6` | `nmap -6 2607:f0d0:1002:51::4` | Enable IPv6 scanning |
| `-A` | `nmap -A 192.168.1.1` | Aggressive scan (OS, version, scripts, traceroute) |
| `--traceroute` | `nmap --traceroute 192.168.1.1` | Trace path to target |
| `--script-updatedb` | `nmap --script-updatedb` | Update NSE script database |
| `--iflist` | `nmap --iflist` | List available network interfaces |
| `--privileged` | `nmap --privileged` | Assume user is privileged |
| `--unprivileged` | `nmap --unprivileged` | Assume user lacks privileges |
| `--release-memory` | `nmap --release-memory` | Release memory before quitting |
| `-h` | `nmap -h` | Display help screen |
| `-V` | `nmap -V` | Display version information |

---

## Common Port Assignments

| Port | Service | Description |
|------|---------|-------------|
| 21 | FTP | File Transfer Protocol |
| 22 | SSH | Secure Shell |
| 23 | Telnet | Unencrypted remote access |
| 25 | SMTP | Simple Mail Transfer Protocol |
| 53 | DNS | Domain Name System |
| 67/68 | DHCP | Dynamic Host Configuration Protocol |
| 69 | TFTP | Trivial File Transfer Protocol |
| 80 | HTTP | Web traffic |
| 88 | Kerberos | Authentication protocol |
| 110 | POP3 | Post Office Protocol v3 |
| 111 | RPC | Remote Procedure Call |
| 119 | NNTP | Network News Transfer Protocol |
| 123 | NTP | Network Time Protocol |
| 135 | RPC | Microsoft Remote Procedure Call |
| 137-139 | NetBIOS | NetBIOS services |
| 143 | IMAP | Internet Message Access Protocol |
| 161/162 | SNMP | Simple Network Management Protocol |
| 389 | LDAP | Lightweight Directory Access Protocol |
| 443 | HTTPS | Secure web traffic |
| 445 | SMB | Server Message Block |
| 514 | Syslog | System logging |
| 636 | LDAPS | Secure LDAP |
| 993 | IMAPS | Secure IMAP |
| 995 | POP3S | Secure POP3 |
| 1433 | MSSQL | Microsoft SQL Server |
| 1521 | Oracle | Oracle Database |
| 2049 | NFS | Network File System |
| 3306 | MySQL | MySQL Database |
| 3389 | RDP | Remote Desktop Protocol |
| 5432 | PostgreSQL | PostgreSQL Database |
| 5900 | VNC | Virtual Network Computing |
| 6379 | Redis | Redis Database |
| 8080 | HTTP-Alt | Alternative HTTP port |
| 8443 | HTTPS-Alt | Alternative HTTPS port |
| 9200 | Elasticsearch | Elasticsearch REST API |
| 27017 | MongoDB | MongoDB Database |

---

## Quick Reference Scripts

### Update Script Database
```bash
sudo nmap --script-updatedb
```

### List Available Scripts
```bash
nmap --script-help=discovery
```

### Get Script Documentation
```bash
nmap --script-help=http-enum
```

---

**Remember**: Always ensure you have proper authorization before scanning networks or systems you don't own!
