# 🛡️ Nessus Vulnerability Remediation Lab

**Status:** ✅ Complete  
**Author:** Josh Divis  
**Platform:** macOS  
**Focus:** Vulnerability identification, remediation, verification

---

## 🔍 Overview

This lab documents the process of scanning my MacBook using Nessus, identifying known vulnerabilities, and remediating or mitigating those findings. All actions were verified by performing follow-up scans to confirm the effectiveness of each fix.

---

## 🔧 Tools Used

- **Nessus Essentials** – for local vulnerability scanning
- **Homebrew** – to update packages
- **Manual Configuration** – system settings, service restarts

---

## 🧪 Scan Process

### 1. Initial Scan
Performed a **Basic Network Scan** on localhost.

Findings included:
- Outdated packages (Node.js, Adobe Creative Cloud, etc.)
- X.509 certificate trust chain issues
- FTP proxy exposure (port 8021)
- UPnP/AirTunes service exposure (ports 5000/7000)

---

### 2. Remediation Actions

| Issue | Resolution |
|-------|------------|
| ⚠️ Node.js outdated | Updated via Homebrew |
| ⚠️ Adobe Creative Cloud | Updated via internal updater |
| ⚠️ ftp-proxy active on port 8021 | Disabled via `launchctl` |
| ⚠️ AirPlay (UPnP) on port 5000/7000 | Turned off via System Preferences |
| ⚠️ X.509 cert error | Verified it applied only to local, non-production certs |

All changes were followed by full service or OS-level restarts as needed.

---

### 3. Validation

Re-ran Nessus Basic Network Scan after changes:
- All critical vulnerabilities cleared
- Only "informational" findings remained
- Confirmed affected services no longer listening on previously open ports

---

## 📘 Lessons Learned

- Nessus can be valuable even on personal machines
- Not all "vulnerabilities" are actionable — context matters
- Post-remediation scans are essential to verify success

---

## 🔐 Final Status

| Category | Result |
|----------|--------|
| Critical/High Vulns | ✅ Resolved |
| Services Hardened | ✅ ftp-proxy, AirPlay |
| Cert Warnings | ⚠️ Acceptable for local testing |
| Follow-Up Scan | ✅ Confirmed changes |

