
# My locally hosted Debian 13 home server

In this project I would like to share my experience of building and deploying a home server used to provide me and my friends the ability to play Minecraft together without having to pay for expensive hosting services and giving old hardware a new life.

Starting with a minimal Debian installation and no internet connection, this project documents how I turned an old good-for-nothing laptop into a powerful home server.

From manually installing packages via USB to clever but compliant workarounds of university network restrictions, this showcases practical problem-solving in Linux system administration and network engineering.

### Technology Stack

- OS: Debian 13 (minimal, headless)
- Runtime: OpenJDK Java Runtime
- Process Manager: GNU Screen + systemd
- Network: wpa_supplicant / networkmanager, eduroam (WPA2-Enterprise)
- Tunneling: Playit.gg
- Remote Access: OpenSSH
- Automation: systemd services, bash scripting
- Package Management: dpkg (offline installation), apt

## Introduction

I've had this idea brewing for a while before finally acting on it. I had an old laptop that was essentially useless after its GPU burnt out. The idea was simple: turn this unusable work laptop into a dedicated Minecraft server that my friends and I could use to play together with demanding mods, without dealing with laggy and expensive hosting services. After all, the laptop still had impressive specs: an Intel i7-9750H and 16GB of DDR4 RAM - too good to waste.

I chose Debian as my distribution due to its minimal and reliable foundation, which is crucial for server stability. My main goal wasn't just the end result or simply playing games, but gaining hands-on experience with server administration and infrastructure - valuable skills I can apply in future employment.

Below is a step-by-step overview of the entire build process.

# Overview
## Initial System Configuration
1. Base Debian Installation

- Installed minimal Debian 13 (no desktop environment)
- Configured console environment

2. Package Management Challenges (No Internet)

##### **Problem:** Minimal installation missing essential packages (sudo, rfkill, etc.)
##### **Solution:** Offline package installation workflow.

- Downloaded .deb packages on Arch laptop from https://packages.debian.org/
- Transferred packages via USB drive
- Mounted USB on Debian server
- Installed packages manually: `dpkg -i [package name].deb`


##### Packages installed: sudo, rfkill, and dependencies

3. Terminal Configuration

- Disabled the annoying terminal bell/beep
- Created aliases in ~/.bashrc:
```
alias shutup='sudo rmmod pcspkr'
```

4. Network Setup

- Connected to eduroam Wi-Fi network (enterprise WPA2)
- Configured wpa_supplicant with PEAP authentication
Manual network configuration:
- Created /etc/wpa_supplicant/eduroam.conf with university credentials
- Used wpa_supplicant and dhclient for connectivity
- Dealt with rfkill blocking (unblocked wireless interface with rfkill unblock wifi)
- Brought up network interface: `ip link set wlp0s20f3 up`
- Later manually set up NetworkManager for autostart and easier, more reliable connection to the internet



5. SSH Access Setup

- Configured SSH server for remote access
- Connected from my Arch laptop on the same network for easier management

6. Power Management Configuration

- Prevented laptop from suspending when lid closes
- Edited /etc/systemd/logind.conf:

```
Set HandleLidSwitch=ignore
Set HandleLidSwitchExternalPower=ignore
Set HandleLidSwitchDocked=ignore
```


- Restarted systemd-logind service

7. Screen Management

- Configured commands to turn off screen when not needed (power saving)
- Created aliases in ~/.bashrc:

```
alias screenoff='echo 0 | sudo tee /sys/class/backlight/intel_backlight/brightness'
alias screenon='echo 120000 | sudo tee /sys/class/backlight/intel_backlight/brightness'
alias screendim='echo 10000 | sudo tee /sys/class/backlight/intel_backlight/brightness'
```

## Minecraft Server Setup
8. Testing Phase (Arch Laptop)

- Installed Java Runtime Environment
- Downloaded Minecraft server jar
- Tested server locally with 1GB RAM allocation
- Verified basic functionality

9. Playit.gg Tunnel Setup (Testing)
##### **Problem:** I do not own the eduroam network, and as a result I do not have admin access. Setting up traditional port forwarding would be impossible.
##### **Solution:** set up network tunneling via global proxy.
- Installed playit on Arch laptop
- Configured tunnel for external access
- Tested external connectivity
- Verified workaround of network restrictions

10. Production Deployment (Debian Server)

- Installed Java on Debian: `sudo apt install default-jre`
- Transferred the server folder via SCP 

11. Resource Allocation

- Allocated a minimum of 4GB of RAM while allowing server to scale up to 14GB: `java -Xms4G -Xmx14G -jar server.jar nogui`
##### Left headroom for:

- Later planned Node.js chatbot to assist me with managing my community (~500MB)
- System processes (~1GB)
- Buffer for stability



12. Process Management with Screen

- Installed and configured GNU Screen
- Launched server in detached screen session: screen -S minecraft
- Enabled server console access via SSH reconnection: screen -r minecraft
- Session survives SSH disconnects (detach with Ctrl+A, D)

13. Playit.gg Production Setup

- Installed playit on Debian server
- Configured tunnel pointing to port 25565
- Obtained permanent public address for players
- Verified external accessibility

14. To-do list:
- Create automation script for starting the services
- Enable auto-start on boot
- Set up automatic backups
- Install demanding mods
- Configure proper restart policies

## result

### Final architecture

```
Internet
    ↓
Playit.gg Tunnel
    ↓
Eduroam Network (NAT/Firewall)
    ↓
Debian Server (Laptop)
    ├── Minecraft Server (Screen session, 4GB RAM)
    ├── Playit.gg Agent (Systemd service)
    ├── Node.js Chatbot (planned, ~500MB RAM)
    └── SSH Server (remote management)
```

### Technical challenges solved

- Offline package management: Resolved dependency issues in air-gapped environment using USB transfer and manual dpkg installation
- Network restrictions: worked around public network NAT/firewall using tunneling
- Power management: Balanced screen-off for power saving with system availability
- Remote management: Enabled full headless operation via SSH
- Process persistence: Ensured services survive SSH disconnects
- Resource optimization: Balanced memory allocation across multiple services

### Skills Demonstrated

- Linux system administration
- Offline package management and dependency resolution
- Network configuration and troubleshooting
- Service automation and orchestration
- Remote server management
- Resource optimization
- Problem-solving in restricted/air-gapped environments

# Bottom Line

This project transformed an otherwise unusable laptop into a fully functional production server, proving that with the right approach, old hardware doesn't have to end up in a landfill. What started as a way to play Minecraft with friends became a comprehensive learning experience in Linux system administration, network engineering, and infrastructure management.

The challenges I faced - from offline package installation to navigating network restrictions - taught me more about practical server administration than any tutorial could. Working in a restricted environment forced me to think creatively and understand systems at a deeper level, skills that translate directly to real-world DevOps and infrastructure roles.

Beyond the technical achievement, this project demonstrated that you don't need expensive cloud hosting or enterprise equipment to run reliable services. A $0 budget, an old laptop, and determination were enough to build something that works just as well, if not better, than paid alternatives.

Most importantly, this server is able to run 24/7 if necessary, providing my friends and I with a lag-free gaming experience while I gain ongoing experience in server maintenance, troubleshooting, and optimization. The knowledge gained here forms a solid foundation for future projects and professional work in system administration and infrastructure engineering.

**Current Status:** Server running stable, with plans to expand functionality through automation scripts, monitoring dashboards, and community management bots.
