# 🛠️ macOS Service Hardening – Port Exposure Review

**Status:** ✅ Complete  
**Author:** Josh Divis  
**Platform:** macOS  
**Focus:** Port hardening, legacy service review, local enumeration

---

## 🔍 Summary

During a local `nmap` scan on `127.0.0.1`, I discovered three open ports associated with default macOS services:

| Port | Service | Description |
|------|---------|-------------|
| 5000 | UPnP (AirTunes) | Apple AirPlay streaming |
| 7000 | afs3-fileserver | Possibly linked to Apple Control Center |
| 8021 | ftp-proxy | Apple’s legacy FTP proxy daemon |

This write-up documents how I investigated and either disabled or reviewed each for further action.

---

## 🧪 Investigation Steps

### ✅ 1. Port Scan
```bash
nmap -sV -Pn -T4 127.0.0.1
```

---

### ✅ 2. ftp-proxy on Port 8021

- Located: `/System/Library/LaunchDaemons/com.apple.ftp-proxy.plist`
- Verified as legacy, unused via:
  ```bash
  sudo lsof -i :8021
  sudo tcpdump -i lo0 port 8021
  ```
- Disabled:
  ```bash
  sudo launchctl bootout system /System/Library/LaunchDaemons/com.apple.ftp-proxy.plist
  ```

✅ Verified port closed and service removed.

---

### ✅ 3. AirPlay (UPnP) on Ports 5000 & 7000

- Associated with AirPlay Receiver
- Disabled via System Settings:
  > System Settings → General → AirDrop & Handoff → Turn off AirPlay Receiver

✅ Both ports closed post-change.

---

### ⚠️ 4. Port 7000 – ControlCe / afs3-fileserver

- Listener: `ControlCe`
- No active traffic via `tcpdump`
- Still under review for precise origin

---

## 🛠️ Tools Used

- `nmap`
- `lsof`
- `tcpdump`
- `launchctl`
- System Settings GUI

---

## 📘 Lessons Learned

- Default macOS services may expose legacy ports
- Not all open ports = danger, but fewer is better
- Real-world security = documentation, verification, and repeatability

---

## 🔐 Final Status

| Port | Action Taken |
|------|----------------------------|
| 8021 | Disabled via launchctl     |
| 5000 | Closed via GUI             |
| 7000 | Under review, not in use   |
