# Nmap Learning Guide

Welcome to the **Nmap Learning Guide**, a step-by-step repository designed to help you master Nmap's core capabilities in a clear, hands-on way. Whether you're new to network security or brushing up your skills, this repo breaks down Nmap into five essential phases:

1. **Host Discovery**  
   Detect which systems are alive on your network.

2. **Port Scanning**  
   Determine which ports are open, closed, or filtered.

3. **Version/Service Detection**  
   Identify running services and their exact software versions.

4. **OS Fingerprinting**  
   Guess the target's operating system and network distance.

5. **Further Probing**  
   Dive deeper with NSE scripts, timing templates, and evasion techniques.

Each phase lives under `docs/` with its own dedicated Markdown file. Together they include:

- **Conceptual overview**  
- **Key commands & options**  
- **Real-world examples**  
- **Legal, hands-on exercises**

Feel free to:

- Fork the repo and explore each doc in sequence  
- Try out the exercises in your own lab environment  
- Contribute improvements or new labs via pull requests

Happy scanning! 🔍🔐

## 🛠️ Netcat (Ncat) as a Complementary Tool

While this guide focuses on Nmap's discovery and scanning phases, there are times when you'll want an **interactive**, **banner-grabbing**, or **service-emulation** tool. That's where **Netcat (Ncat)** shines:

- **Quick port checks** (`nc -zv`)
- **Manual banner grabs** for mis-fingerprinted services
- **Custom probes** or simple HTTP/SMTP/FTP clients
- **File transfers**, **bind/reverse shells**, and **proxy chaining**

Throughout the phase docs you'll find "Netcat Tips" that show how to switch from Nmap's automated scans to Netcat's hands-on techniques when more control or deeper interaction is needed.
