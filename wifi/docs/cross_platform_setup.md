# Cross-Platform Wireless Security Setup Guide

This guide provides detailed instructions for setting up wireless security testing environments on different operating systems, allowing you to develop skills regardless of your platform preference.

## 🖥️ Platform-Specific Setup

### **Linux Setup (Recommended)**

#### **Kali Linux (Easiest)**
```bash
# System update
sudo apt update && sudo apt upgrade -y

# Install essential tools (usually pre-installed)
sudo apt install -y aircrack-ng wireshark reaver pixiewps hostapd dnsmasq

# Install additional tools
sudo apt install -y hcxtools hashcat john wifite

# Update databases
sudo airodump-ng-oui-update
sudo aircrack-ng-oui-update
```

#### **Ubuntu/Debian**
```bash
# Add repositories
sudo add-apt-repository ppa:hanipouspilot/rtlwifi

# Install aircrack-ng
sudo apt update
sudo apt install -y aircrack-ng

# Install Wireshark
sudo apt install -y wireshark
sudo usermod -a -G wireshark $USER

# Install wireless drivers
sudo apt install -y rtl8812au-dkms
```

#### **Arch Linux**
```bash
# Install from AUR
yay -S aircrack-ng-git
yay -S wireshark-qt

# Install drivers
yay -S rtl8812au-dkms-git
```

### **Windows Setup**

#### **Windows 10/11 Native**
```powershell
# Install Windows Subsystem for Linux (WSL2)
wsl --install -d kali-linux

# Install Wireshark for Windows
# Download from: https://www.wireshark.org/download.html

# Install aircrack-ng Windows binaries
# Download from: https://aircrack-ng.org/doku.php?id=install_aircrack_windows
```

#### **Windows with Cygwin**
```bash
# Install Cygwin with development tools
# Download setup-x86_64.exe from cygwin.com

# Install required packages through Cygwin installer:
# - gcc-core
# - make
# - libpcap-devel
# - openssl-devel
```

#### **Windows PowerShell Scripts**
```powershell
# PowerShell wrapper for aircrack-ng
function Start-WirelessScan {
    param([string]$Interface)
    & "C:\Program Files\Aircrack-ng\bin\airodump-ng.exe" $Interface
}

function Start-WirelessCapture {
    param([string]$Interface, [string]$Output)
    & "C:\Program Files\Aircrack-ng\bin\airodump-ng.exe" -w $Output $Interface
}
```

### **macOS Setup**

#### **Homebrew Installation**
```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install aircrack-ng
brew install aircrack-ng

# Install Wireshark
brew install --cask wireshark

# Install additional tools
brew install hashcat john-jumbo
```

#### **macOS-Specific Considerations**
- Limited monitor mode support
- Use external USB adapters
- Virtual machine recommended for full functionality

## 🔧 Wireless Adapter Compatibility

### **Linux Compatible Adapters**
| Adapter | Chipset | Monitor Mode | Injection | Price Range |
|---------|---------|-------------|-----------|-------------|
| Alfa AWUS036ACS | Realtek RTL8812AU | ✅ | ✅ | $30-50 |
| Alfa AWUS036NHA | Atheros AR9271 | ✅ | ✅ | $25-40 |
| Panda PAU09 | Ralink RT5372 | ✅ | ✅ | $15-25 |
| TP-Link AC600 | Realtek RTL8811AU | ✅ | ✅ | $20-30 |

### **Windows Compatible Adapters**
| Adapter | Windows Driver | Monitor Mode | Notes |
|---------|---------------|-------------|-------|
| Alfa AWUS036ACS | Official | Limited | Use with WSL2 |
| Netgear A6210 | Official | No | Wireshark only |
| Intel AX200 | Built-in | No | Promiscuous mode |

### **Cross-Platform Testing**
```bash
# Test adapter compatibility
lsusb | grep -E "(Realtek|Atheros|Ralink)"
iwconfig
sudo airmon-ng check
```

## 🛠️ Environment Configuration

### **Linux Configuration**
```bash
# Configure monitor mode
sudo airmon-ng start wlan0

# Kill conflicting processes
sudo airmon-ng check kill

# Set regulatory domain
sudo iw reg set US

# Configure interface
sudo ip link set wlan0mon up
sudo iwconfig wlan0mon mode monitor
```

### **Windows Configuration**
```powershell
# Check wireless interfaces
netsh wlan show interfaces

# Enable monitor mode (limited)
netsh wlan set hostednetwork mode=allow

# Configure Wireshark for Windows
# Enable promiscuous mode in capture options
```

### **Virtual Machine Setup**
```bash
# VirtualBox USB passthrough
VBoxManage modifyvm "Kali-Linux" --usb on --usbehci on
VBoxManage list usbhost
VBoxManage controlvm "Kali-Linux" usbattach <device-uuid>

# VMware USB passthrough
# VM Settings → USB & Bluetooth → Add USB device
```

## 📊 Cross-Platform Tool Comparison

### **Aircrack-ng Availability**
| Platform | Installation | Monitor Mode | Injection | Performance |
|----------|-------------|-------------|-----------|-------------|
| **Linux** | Native | Full | Full | Excellent |
| **Windows** | Binary/WSL | Limited | Limited | Good |
| **macOS** | Homebrew | Limited | Limited | Fair |
| **Android** | Termux | Limited | No | Poor |

### **Wireshark Capabilities**
| Platform | Installation | 802.11 Dissection | Live Capture | Export |
|----------|-------------|------------------|-------------|---------|
| **Linux** | Package | Full | Yes | All formats |
| **Windows** | Installer | Full | Yes | All formats |
| **macOS** | Homebrew | Full | Limited | All formats |

## 🎯 Platform-Specific Exercises

### **Linux Exercises**
1. **Full Monitor Mode Testing**
   ```bash
   sudo airmon-ng start wlan0
   sudo airodump-ng wlan0mon
   sudo aireplay-ng --deauth 1 -a [AP_MAC] wlan0mon
   ```

2. **Advanced Injection**
   ```bash
   sudo aireplay-ng -9 wlan0mon  # Injection test
   sudo aireplay-ng -3 -b [AP_MAC] wlan0mon  # ARP replay
   ```

### **Windows Exercises**
1. **WSL2 Integration**
   ```bash
   # In WSL2 Kali
   sudo airmon-ng start wlan0
   # Use USB passthrough for wireless adapter
   ```

2. **Native Windows Analysis**
   ```powershell
   # Capture with Wireshark
   # Analyze in Windows environment
   # Export to Linux for advanced processing
   ```

### **Cross-Platform Exercises**
1. **Mixed Environment Testing**
   - Linux attacker machine
   - Windows target systems
   - macOS monitoring station

2. **Data Portability**
   - Capture on Linux
   - Analyze on Windows
   - Report on macOS

## 🔍 Troubleshooting Guide

### **Common Linux Issues**
```bash
# Driver problems
sudo dkms status
sudo modprobe -r rtl8812au && sudo modprobe rtl8812au

# Permission issues
sudo usermod -a -G wireshark $USER
sudo chmod +x /usr/bin/aircrack-ng

# Monitor mode conflicts
sudo airmon-ng check kill
sudo systemctl stop NetworkManager
```

### **Common Windows Issues**
```powershell
# WSL2 USB issues
# Restart WSL2 service
wsl --shutdown
wsl --distribution kali-linux

# Driver conflicts
# Disable Windows wireless service temporarily
net stop wlansvc
```

### **Universal Solutions**
```bash
# Reset wireless stack
sudo systemctl restart networking
sudo ifconfig wlan0 down && sudo ifconfig wlan0 up

# Update drivers
sudo apt update && sudo apt upgrade
sudo dkms autoinstall
```

## 📚 Platform-Specific Resources

### **Linux Resources**
- [Aircrack-ng Official Documentation](https://aircrack-ng.org/doku.php)
- [Kali Linux Wireless Attacks](https://kali.training/topic/wireless-attacks/)
- [Ubuntu Wireless Troubleshooting](https://help.ubuntu.com/community/WifiDocs/WirelessTroubleShootingGuide)

### **Windows Resources**
- [WSL2 Documentation](https://docs.microsoft.com/en-us/windows/wsl/)
- [Wireshark Windows Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [Windows Wireless APIs](https://docs.microsoft.com/en-us/windows/win32/nativewifi/native-wifi-api-portal)

### **macOS Resources**
- [Homebrew Wireless Tools](https://brew.sh/)
- [macOS Wireless Debugging](https://support.apple.com/en-us/HT202639)
- [Wireshark macOS Guide](https://www.wireshark.org/docs/)

## 🎓 Skills Development Path

### **Beginner (Any Platform)**
1. Install tools on your preferred platform
2. Learn basic wireless concepts
3. Practice packet capture and analysis
4. Understand security protocols

### **Intermediate (Cross-Platform)**
1. Set up multiple OS environments
2. Compare tool capabilities across platforms
3. Develop cross-platform scripts
4. Practice advanced attack techniques

### **Expert (Platform Agnostic)**
1. Master all platform-specific tools
2. Develop custom cross-platform solutions
3. Contribute to open-source projects
4. Teach others platform-specific techniques

---

*This guide enables wireless security learning regardless of your operating system preference, ensuring comprehensive skill development across all major platforms.*
