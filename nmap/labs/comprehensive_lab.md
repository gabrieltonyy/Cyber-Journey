# Comprehensive Nmap Laboratory

**Objective**: Practice all phases of Nmap scanning in a structured, hands-on environment that simulates real-world network reconnaissance and security assessment scenarios.

---

## Lab Overview

This comprehensive laboratory guides you through all five phases of Nmap scanning:

1. **Host Discovery** - Map live systems on the network
2. **Port Scanning** - Identify open services and attack surface  
3. **Version Detection** - Fingerprint software versions and configurations
4. **OS Fingerprinting** - Determine operating systems and device types
5. **Further Probing** - Advanced techniques with NSE scripts and evasion

Each section builds upon the previous, creating a complete reconnaissance methodology suitable for penetration testing, security auditing, and network management.

---

## Prerequisites

### Required Software
- **Nmap** (latest version recommended)
- **Netcat/Ncat** for manual verification
- **Wireshark** for packet analysis (optional)
- **Root/Administrator privileges** for advanced techniques

### Required Infrastructure
- **Test Network**: Isolated lab environment (VMs, containers, or dedicated lab subnet)
- **Target Systems**: At least 3 different systems (Linux, Windows, network appliance)
- **Vulnerable Targets**: Metasploitable2, DVWA, or similar intentionally vulnerable systems
- **Network Tools**: Router/firewall for testing evasion techniques

### Safety and Legal Requirements
⚠️ **CRITICAL**: Only scan networks and systems you own or have explicit written permission to test. Unauthorized scanning may violate laws and organizational policies.

---

## Phase 1: Network Discovery Laboratory

### Exercise 1.1: Multi-Method Host Discovery
**Objective**: Compare different discovery techniques and understand their effectiveness.

```bash
# Create results directory
mkdir -p lab_results/phase1

# Method 1: Default ping scan
echo "=== Default Ping Scan ===" | tee lab_results/phase1/discovery_comparison.txt
nmap -sn 192.168.1.0/24 | tee -a lab_results/phase1/discovery_comparison.txt

# Method 2: ARP-only discovery (LAN)
echo -e "\n=== ARP Discovery ===" | tee -a lab_results/phase1/discovery_comparison.txt  
sudo nmap -sn -PR 192.168.1.0/24 | tee -a lab_results/phase1/discovery_comparison.txt

# Method 3: TCP SYN discovery
echo -e "\n=== TCP SYN Discovery ===" | tee -a lab_results/phase1/discovery_comparison.txt
nmap -sn -PS22,80,443 192.168.1.0/24 | tee -a lab_results/phase1/discovery_comparison.txt

# Method 4: UDP discovery  
echo -e "\n=== UDP Discovery ===" | tee -a lab_results/phase1/discovery_comparison.txt
nmap -sn -PU53,161,123 192.168.1.0/24 | tee -a lab_results/phase1/discovery_comparison.txt
```

**Analysis Questions:**
1. Which method discovered the most hosts?
2. Are there hosts that only responded to specific probe types?
3. How do the response times compare between methods?

### Exercise 1.2: Stealth vs. Speed Trade-offs
**Objective**: Understand the relationship between scan stealth and efficiency.

```bash
# Speed test different discovery approaches
echo "Testing discovery speed and stealth..." | tee lab_results/phase1/speed_comparison.txt

# Fast discovery
echo "=== Fast Discovery ===" | tee -a lab_results/phase1/speed_comparison.txt
time nmap -sn -T5 --min-rate 1000 192.168.1.0/24 | tee -a lab_results/phase1/speed_comparison.txt

# Balanced discovery
echo -e "\n=== Balanced Discovery ===" | tee -a lab_results/phase1/speed_comparison.txt  
time nmap -sn -T3 192.168.1.0/24 | tee -a lab_results/phase1/speed_comparison.txt

# Stealth discovery
echo -e "\n=== Stealth Discovery ===" | tee -a lab_results/phase1/speed_comparison.txt
time nmap -sn -T1 --scan-delay 2s 192.168.1.0/24 | tee -a lab_results/phase1/speed_comparison.txt
```

### Exercise 1.3: IPv6 Discovery Challenge
**Objective**: Practice IPv6 network discovery techniques.

```bash
# IPv6 link-local discovery
sudo nmap -6 -sn fe80::/64 2>/dev/null | tee lab_results/phase1/ipv6_discovery.txt

# If you have IPv6 global addresses
nmap -6 -sn 2001:db8::/32 2>/dev/null | tee -a lab_results/phase1/ipv6_discovery.txt
```

---

## Phase 2: Port Scanning Laboratory

### Exercise 2.1: Comprehensive Port Analysis
**Objective**: Master different scanning techniques and understand their applications.

**Step 1: Select Target Hosts**
```bash
# Extract live hosts from discovery phase
grep "Nmap scan report" lab_results/phase1/discovery_comparison.txt | awk '{print $5}' > lab_results/targets.txt

# Select first 3 targets for detailed analysis
head -3 lab_results/targets.txt > lab_results/detailed_targets.txt
```

**Step 2: Multi-Technique Port Scanning**
```bash
mkdir -p lab_results/phase2

# For each target, run comprehensive port analysis
while read target; do
    echo "Analyzing target: $target"
    
    # TCP SYN scan (stealth)
    sudo nmap -sS -p- -T4 -oA lab_results/phase2/${target}_tcp_syn $target
    
    # TCP Connect scan (comparison) 
    nmap -sT -F -T4 -oA lab_results/phase2/${target}_tcp_connect $target
    
    # UDP top ports scan
    sudo nmap -sU --top-ports 100 -T4 -oA lab_results/phase2/${target}_udp $target
    
    # ACK scan for firewall analysis
    sudo nmap -sA -p 1-1000 -T4 -oA lab_results/phase2/${target}_ack $target
    
done < lab_results/detailed_targets.txt
```

### Exercise 2.2: Performance Optimization Laboratory
**Objective**: Learn to balance speed, accuracy, and network impact.

```bash
# Test different timing templates on same target
target=$(head -1 lab_results/detailed_targets.txt)

echo "Performance comparison for $target" | tee lab_results/phase2/performance_test.txt

# T0 - Paranoid (very slow)
echo "=== T0 Paranoid ===" | tee -a lab_results/phase2/performance_test.txt
time sudo nmap -sS -T0 -p 1-100 $target | tee -a lab_results/phase2/performance_test.txt

# T3 - Normal (default)
echo -e "\n=== T3 Normal ===" | tee -a lab_results/phase2/performance_test.txt
time sudo nmap -sS -T3 -p 1-100 $target | tee -a lab_results/phase2/performance_test.txt

# T5 - Insane (very fast)
echo -e "\n=== T5 Insane ===" | tee -a lab_results/phase2/performance_test.txt
time sudo nmap -sS -T5 -p 1-100 $target | tee -a lab_results/phase2/performance_test.txt

# Custom optimized scan
echo -e "\n=== Custom Optimized ===" | tee -a lab_results/phase2/performance_test.txt
time sudo nmap -sS --min-rate 1000 --max-retries 1 -p 1-100 $target | tee -a lab_results/phase2/performance_test.txt
```

### Exercise 2.3: Firewall Evasion Challenge
**Objective**: Practice techniques to bypass basic packet filtering.

```bash
target=$(head -1 lab_results/detailed_targets.txt)

# Standard scan (baseline)
sudo nmap -sS -p 80,443 $target -oN lab_results/phase2/baseline_scan.txt

# Fragmented packets
sudo nmap -sS -f -p 80,443 $target -oN lab_results/phase2/fragmented_scan.txt

# Source port spoofing  
sudo nmap -sS --source-port 53 -p 80,443 $target -oN lab_results/phase2/spoofed_port_scan.txt

# Decoy scan
sudo nmap -sS -D RND:5 -p 80,443 $target -oN lab_results/phase2/decoy_scan.txt

# Null scan (evasion)
sudo nmap -sN -p 80,443 $target -oN lab_results/phase2/null_scan.txt
```

---

## Phase 3: Version Detection Laboratory

### Exercise 3.1: Service Fingerprinting Mastery
**Objective**: Master version detection techniques and understand their limitations.

```bash
mkdir -p lab_results/phase3

# For each target with open ports
while read target; do
    echo "Version detection for $target"
    
    # Basic version detection
    nmap -sV $target -oA lab_results/phase3/${target}_version_basic
    
    # Intensive version detection
    nmap -sV --version-intensity 9 $target -oA lab_results/phase3/${target}_version_intensive
    
    # Light version detection (for comparison)
    nmap -sV --version-light $target -oA lab_results/phase3/${target}_version_light
    
    # Version detection with default scripts
    nmap -sV -sC $target -oA lab_results/phase3/${target}_version_scripts
    
done < lab_results/detailed_targets.txt
```

### Exercise 3.2: Manual Banner Grabbing Laboratory
**Objective**: Learn to manually verify and supplement Nmap's version detection.

```bash
# Extract open ports from previous scans
grep "open" lab_results/phase2/*_tcp_syn.nmap | head -10 > lab_results/phase3/open_ports.txt

# Manual banner grabbing for each service
echo "Manual banner grabbing results:" | tee lab_results/phase3/manual_banners.txt

# HTTP banner grabbing
echo -e "\n=== HTTP Banners ===" | tee -a lab_results/phase3/manual_banners.txt
printf "HEAD / HTTP/1.0\r\n\r\n" | nc target.domain 80 | tee -a lab_results/phase3/manual_banners.txt

# SSH version extraction
echo -e "\n=== SSH Banners ===" | tee -a lab_results/phase3/manual_banners.txt
nc target.domain 22 | head -1 | tee -a lab_results/phase3/manual_banners.txt

# SMTP banner grabbing
echo -e "\n=== SMTP Banners ===" | tee -a lab_results/phase3/manual_banners.txt
echo "HELO test" | nc target.domain 25 | tee -a lab_results/phase3/manual_banners.txt
```

### Exercise 3.3: SSL/TLS Security Assessment
**Objective**: Perform comprehensive SSL/TLS security analysis.

```bash
# Find HTTPS services
grep "443/tcp.*open" lab_results/phase2/*.nmap | cut -d: -f1 | head -3 > lab_results/phase3/https_targets.txt

while read scan_file; do
    target=$(basename $scan_file | cut -d_ -f1)
    echo "SSL analysis for $target"
    
    # SSL certificate analysis
    nmap -p443 --script ssl-cert $target -oN lab_results/phase3/${target}_ssl_cert.txt
    
    # SSL cipher enumeration
    nmap -p443 --script ssl-enum-ciphers $target -oN lab_results/phase3/${target}_ssl_ciphers.txt
    
    # SSL vulnerability checks
    nmap -p443 --script ssl-heartbleed,ssl-poodle $target -oN lab_results/phase3/${target}_ssl_vulns.txt
    
done < lab_results/phase3/https_targets.txt
```

---

## Phase 4: OS Fingerprinting Laboratory

### Exercise 4.1: Multi-Platform OS Detection
**Objective**: Practice OS fingerprinting across different platforms and configurations.

```bash
mkdir -p lab_results/phase4

# OS detection for all targets
while read target; do
    echo "OS fingerprinting for $target"
    
    # Standard OS detection
    sudo nmap -O $target -oN lab_results/phase4/${target}_os_detection.txt
    
    # OS detection with guessing
    sudo nmap -O --osscan-guess $target -oN lab_results/phase4/${target}_os_guess.txt
    
    # Combined OS and version detection
    sudo nmap -O -sV $target -oA lab_results/phase4/${target}_os_version
    
    # Enhanced OS detection with scripts
    sudo nmap -O --script smb-os-discovery,http-server-header $target -oN lab_results/phase4/${target}_os_enhanced.txt
    
done < lab_results/detailed_targets.txt
```

### Exercise 4.2: Network Device Classification
**Objective**: Identify and classify different types of network devices.

```bash
# Target network infrastructure devices
echo "192.168.1.1" > lab_results/phase4/infrastructure_targets.txt  # Router
echo "192.168.1.10" >> lab_results/phase4/infrastructure_targets.txt # Printer/NAS (if available)

while read device; do
    echo "Device classification for $device"
    
    # OS detection with SNMP
    sudo nmap -O --script snmp-sysdescr --script-args snmpcommunity=public $device -oN lab_results/phase4/${device}_device_class.txt
    
    # Web interface analysis
    nmap -p80,443 --script http-title,http-server-header $device -oN lab_results/phase4/${device}_web_interface.txt
    
done < lab_results/phase4/infrastructure_targets.txt
```

### Exercise 4.3: Virtualization Detection Challenge
**Objective**: Distinguish between physical and virtual systems.

```bash
# OS fingerprinting with virtualization focus
while read target; do
    echo "Virtualization analysis for $target"
    
    # Detailed OS fingerprinting
    sudo nmap -O --osscan-guess -v $target > lab_results/phase4/${target}_virtualization.txt 2>&1
    
    # Look for virtualization indicators in MAC addresses and TTL values
    sudo nmap -sn $target | grep -E "(MAC|TTL)" >> lab_results/phase4/${target}_virtualization.txt
    
done < lab_results/detailed_targets.txt
```

---

## Phase 5: Advanced Probing Laboratory

### Exercise 5.1: NSE Script Mastery
**Objective**: Master the Nmap Scripting Engine for advanced reconnaissance.

```bash
mkdir -p lab_results/phase5

# Update NSE database
sudo nmap --script-updatedb

while read target; do
    echo "Advanced scripting for $target"
    
    # Vulnerability scanning
    nmap --script vuln $target -oN lab_results/phase5/${target}_vulnerabilities.txt
    
    # Web application enumeration
    nmap -p80,443 --script http-enum,http-methods,http-robots.txt $target -oN lab_results/phase5/${target}_web_enum.txt
    
    # SMB enumeration (if Windows target)
    nmap -p445 --script smb-enum-shares,smb-enum-users,smb-os-discovery $target -oN lab_results/phase5/${target}_smb_enum.txt
    
    # Brute force detection
    nmap --script auth $target -oN lab_results/phase5/${target}_auth_methods.txt
    
done < lab_results/detailed_targets.txt
```

### Exercise 5.2: Comprehensive Evasion Laboratory
**Objective**: Master advanced evasion techniques for bypassing security controls.

```bash
target=$(head -1 lab_results/detailed_targets.txt)

echo "Advanced evasion testing for $target" | tee lab_results/phase5/evasion_test.txt

# Ultra-stealth scan
echo "=== Ultra-Stealth Scan ===" | tee -a lab_results/phase5/evasion_test.txt
sudo nmap -sS -T0 -f --scan-delay 5s --randomize-hosts --data-length 25 --spoof-mac Dell -D RND:3 --source-port 53 $target | tee -a lab_results/phase5/evasion_test.txt

# Idle scan (requires zombie host)
# echo "=== Idle Scan ===" | tee -a lab_results/phase5/evasion_test.txt  
# sudo nmap -sI zombie_host:113 $target | tee -a lab_results/phase5/evasion_test.txt

# Custom data payload
echo -e "\n=== Custom Data Payload ===" | tee -a lab_results/phase5/evasion_test.txt
sudo nmap -sS --data-string "HTTPRequest" $target | tee -a lab_results/phase5/evasion_test.txt
```

### Exercise 5.3: Automation and Integration Laboratory
**Objective**: Create automated scanning workflows and integrate results.

```bash
# Create comprehensive scanning script
cat > lab_results/phase5/automated_scan.sh << 'EOF'
#!/bin/bash
TARGET=$1
OUTPUT_DIR="automated_results"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target_ip_or_range>"
    exit 1
fi

mkdir -p $OUTPUT_DIR

echo "Starting comprehensive automated scan of $TARGET"

# Phase 1: Discovery
echo "Phase 1: Host Discovery"
nmap -sn $TARGET -oA $OUTPUT_DIR/01_discovery

# Phase 2: Port Scanning  
echo "Phase 2: Port Scanning"
nmap -sS -p- -T4 $TARGET -oA $OUTPUT_DIR/02_ports

# Phase 3: Service Detection
echo "Phase 3: Service Detection"  
nmap -sV --script=default $TARGET -oA $OUTPUT_DIR/03_services

# Phase 4: OS Detection
echo "Phase 4: OS Detection"
nmap -O --osscan-guess $TARGET -oA $OUTPUT_DIR/04_os

# Phase 5: Vulnerability Scan
echo "Phase 5: Vulnerability Assessment"
nmap --script vuln $TARGET -oA $OUTPUT_DIR/05_vulnerabilities

echo "Scan complete! Results in $OUTPUT_DIR/"
EOF

chmod +x lab_results/phase5/automated_scan.sh

# Test automated script
./lab_results/phase5/automated_scan.sh $(head -1 lab_results/detailed_targets.txt)
```

---

## Lab Analysis and Reporting

### Exercise 6.1: Result Correlation and Analysis
**Objective**: Synthesize findings from all phases into actionable intelligence.

```bash
mkdir -p lab_results/analysis

# Create comprehensive analysis report
cat > lab_results/analysis/lab_summary.md << 'EOF'
# Comprehensive Nmap Laboratory - Analysis Report

## Executive Summary
[Summarize key findings, security concerns, and recommendations]

## Network Topology
[Describe discovered network structure and device types]

## Host Inventory
[List all discovered hosts with OS and key services]

## Security Findings
[Detail vulnerabilities, misconfigurations, and risks]

## Recommendations
[Provide specific remediation steps and security improvements]
EOF

# Generate host inventory from all scans
echo "# Host Inventory" > lab_results/analysis/host_inventory.txt
grep -h "Nmap scan report" lab_results/phase*/*.nmap | sort -u >> lab_results/analysis/host_inventory.txt

# Extract all open services
echo "# Service Inventory" > lab_results/analysis/service_inventory.txt
grep -h "/tcp.*open" lab_results/phase*/*.nmap | sort -u >> lab_results/analysis/service_inventory.txt
```

### Exercise 6.2: Vulnerability Assessment Report
**Objective**: Create a professional vulnerability assessment report.

```bash
# Extract vulnerability findings
grep -r "VULNERABLE" lab_results/phase5/ > lab_results/analysis/vulnerabilities.txt

# Create remediation priority matrix
cat > lab_results/analysis/remediation_priorities.txt << 'EOF'
# Vulnerability Remediation Priorities

## Critical (Fix Immediately)
- [ ] Unpatched services with known exploits
- [ ] Default credentials  
- [ ] Unnecessary administrative interfaces

## High (Fix Within 30 Days)
- [ ] Outdated software versions
- [ ] Weak SSL/TLS configurations
- [ ] Information disclosure issues

## Medium (Fix Within 90 Days)  
- [ ] Non-essential services
- [ ] Banner information leakage
- [ ] Missing security headers

## Low (Monitor/Document)
- [ ] Version information disclosure
- [ ] Fingerprinting responses
- [ ] Network topology visibility
EOF
```

---

## Advanced Challenge Exercises

### Challenge 1: Stealth Assessment
**Objective**: Complete full reconnaissance while avoiding detection.

**Rules:**
- Use only timing T1 or slower
- Implement randomization and delays
- Avoid aggressive probes
- Document stealth techniques used

```bash
# Ultra-stealth comprehensive scan
sudo nmap -sS -T1 --scan-delay 10s --randomize-hosts -f --data-length 200 \
         -D RND:5 --source-port 53 $(cat lab_results/targets.txt) \
         -oA lab_results/challenge_stealth
```

### Challenge 2: Zero-Day Discovery Simulation
**Objective**: Identify potentially exploitable services without known CVEs.

**Focus Areas:**
- Custom web applications
- Non-standard ports and services  
- Configuration weaknesses
- Default credentials
- Information disclosure

```bash
# Custom application discovery
nmap --script http-enum,http-methods,http-robots.txt,http-git \
     -p 8000-9000 $(cat lab_results/targets.txt) \
     -oN lab_results/challenge_custom_apps.txt
```

### Challenge 3: Network Architecture Mapping
**Objective**: Create detailed network diagram from Nmap data.

**Deliverables:**
- Network topology diagram
- Device type classification
- Trust boundary identification
- Attack path analysis

---

## Conclusion and Next Steps

### Skills Demonstrated
Upon completing this laboratory, you will have demonstrated proficiency in:

- **Network Discovery**: Multiple techniques for host identification
- **Port Scanning**: Various scan types and optimization strategies  
- **Service Detection**: Version identification and banner analysis
- **OS Fingerprinting**: Operating system and device classification
- **Advanced Techniques**: NSE scripting and evasion methods
- **Analysis and Reporting**: Professional documentation and prioritization

### Integration Opportunities
This laboratory's techniques integrate with:

- **Vulnerability Management**: Feed discoveries into vulnerability scanners
- **Penetration Testing**: Use reconnaissance for attack planning
- **Network Monitoring**: Baseline normal services and detect changes
- **Compliance Assessment**: Verify security standard compliance
- **Asset Management**: Maintain accurate inventory systems

### Continuous Learning
- Practice on new targets and environments
- Stay current with NSE script updates
- Explore specialized scanning for specific technologies
- Develop custom automation and reporting workflows
- Contribute to the security community through responsible disclosure

**Remember**: Use these skills responsibly and only on systems you own or have explicit permission to test. Great power comes with great responsibility in cybersecurity.

---

*Laboratory Version 1.0 - Updated 2025*
