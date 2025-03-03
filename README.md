# owncloudraspberrypi
Home Lab project utilizing Raspberry Pi &amp; ownCloud for self-hosting your own personal cloud
# Part 1: Setting a Static IP on My Raspberry Pi 4 for OwnCloud

This is the first part of my project to host OwnCloud on a Raspberry Pi 4 using local storage. On March 2, 2025, I set a static IP (`192.168.50.50`) on my Pi’s Wi-Fi (`wlan0`) without changing my router. Below are the skills I picked up, a copy/paste guide that works, and the issues I ran into with how I fixed them.

---

## Skills Gained and Goal

### Skills Gained
- **Network Troubleshooting**: Figured out `dhcpcd` wasn’t on my Pi and switched to NetworkManager.
- **Command-Line Use**: Ran `ip`, `nmcli`, and `systemctl` to set and check network stuff.
- **Problem-Solving**: Fixed SSH drops, extra IPs, and no internet by testing step-by-step.
- **Documentation**: Wrote this guide to explain it clearly for beginners.

### Goal
Give my Raspberry Pi 4 a static IP (`192.168.50.50`) on Wi-Fi so OwnCloud always has the same address, all set up on the Pi itself.

---

## Copy/Paste Guide: Set Your Static IP

Here’s what worked for me after figuring it out. If you’ve got a Raspberry Pi 4 with Raspberry Pi OS, Wi-Fi on, and SSH ready, just copy and paste these commands. I’ll explain what you’ll see so you know what’s up.

### Prerequisites
- Raspberry Pi 4 with Wi-Fi connected and SSH turned on.
- A terminal—either SSH in or use a screen and keyboard.

### Steps
```bash
# Check current IP and gateway
ip addr
# This shows your network stuff. Look for "wlan0" - mine had "192.168.50.183/24" under "inet".
ip route | grep default
# This finds your router’s IP. Mine said "default via 192.168.50.1" - that’s the gateway.

# Verify NetworkManager is running
sudo systemctl status NetworkManager
# This checks if NetworkManager is on. You’ll see "active (running)" if it’s good. If not, install it later.
nmcli con show
# Lists your Wi-Fi connection. Mine showed "preconfigured" for my network "DAVIS 2G". Note your name.

# Set static IP (replace "preconfigured" with your Wi-Fi name from above)
sudo nmcli con mod "preconfigured" ipv4.addresses 192.168.50.50/24
# Sets your Pi to use 192.168.50.50. Nothing shows up after this - it just saves it.
sudo nmcli con mod "preconfigured" ipv4.gateway 192.168.50.1
# Tells your Pi where the router is (192.168.50.1 was mine). No output here either.
sudo nmcli con mod "preconfigured" ipv4.dns "8.8.8.8 8.8.4.4"
# Sets Google’s DNS so internet names work. Still no message - it’s quiet.
sudo nmcli con mod "preconfigured" ipv4.method manual
# Stops your Pi from grabbing random IPs. Again, no output, just sets it.

# Apply changes
sudo systemctl restart NetworkManager
# Restarts the network system. Might take a sec. If SSH drops, wait and try "ssh <user>@192.168.50.50".

# Verify (reconnect if SSH drops: ssh <user>@192.168.50.50)
ip addr
# Check "wlan0" again. Should now say "192.168.50.50/24" under "inet" and nothing else.
ping 8.8.8.8
# Tests the internet. You’ll see "64 bytes from 8.8.8.8" lines if it’s working. Press Ctrl+C to stop.
nmcli con show "preconfigured" | grep ipv4
# Shows your settings. Look for "manual", "192.168.50.50/24", "192.168.50.1", and DNS.

# Test reboot
sudo reboot
# Restarts your Pi. It’ll disconnect you. Wait a minute for it to come back up.
# After reboot, SSH in again: ssh <user>@192.168.50.50
ip addr
# Same check - "wlan0" should still be "192.168.50.50/24".
ping 8.8.8.8
# Internet should still work with those "64 bytes" replies.
