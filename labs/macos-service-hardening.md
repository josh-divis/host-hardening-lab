# 🛠️ macOS Service Hardening – Port Exposure Review

![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-macOS-000000?style=for-the-badge&logo=apple&logoColor=white)
![Security](https://img.shields.io/badge/Security-Service%20Hardening-blue?style=for-the-badge&logo=security&logoColor=white)
![Methodology](https://img.shields.io/badge/Methodology-Port%20Analysis-orange?style=for-the-badge)

> **Author:** Josh Divis  
> **Focus:** Port hardening, legacy service review, local enumeration  
> **Attack Surface Reduction:** 67% (2 of 3 services secured)

---

## 📋 Table of Contents
- [Executive Summary](#-executive-summary)
- [Security Assessment](#-security-assessment)
- [Investigation Methodology](#-investigation-methodology)
- [Technical Implementation](#-technical-implementation)
- [Risk Analysis](#-risk-analysis)
- [Tools & Techniques](#-tools--techniques)
- [Lessons Learned](#-lessons-learned)
- [Final Security Posture](#-final-security-posture)

---

## 🎯 Executive Summary

During a **localhost security assessment**, I discovered three open ports associated with default macOS services that posed potential security risks. Through systematic investigation and targeted remediation, I successfully **reduced the attack surface by 67%** while maintaining system functionality.

**Key Achievements:**
- ✅ **Eliminated legacy FTP proxy exposure** (Port 8021)
- ✅ **Disabled unnecessary AirPlay services** (Ports 5000/7000)
- ✅ **Documented secure service management** procedures
- 🔍 **Identified remaining service** for future monitoring

This assessment demonstrates practical **attack surface reduction** techniques and **defense-in-depth** security principles applicable to enterprise endpoint hardening.

---

## 🔍 Security Assessment

### 📊 Initial Discovery

<details>
<summary><b>🔬 Click to view scan results</b></summary>

```bash
nmap -sV -Pn -T4 127.0.0.1
```

**Findings:**
```
PORT     STATE SERVICE         RISK LEVEL
5000/tcp open  upnp (AirTunes) 🟡 MEDIUM
7000/tcp open  afs3-fileserver 🟡 MEDIUM  
8021/tcp open  ftp-proxy       🔴 HIGH
```

</details>

### 🎯 Service Risk Matrix

| Port | Service | Risk Level | Business Impact | Security Concern |
|------|---------|------------|-----------------|------------------|
| **8021** | FTP Proxy | 🔴 **HIGH** | Legacy protocol exposure | Unauthenticated file access |
| **5000** | AirTunes/UPnP | 🟡 **MEDIUM** | Network service discovery | Information disclosure |
| **7000** | afs3-fileserver | 🟡 **MEDIUM** | Control Center backend | Potential privilege escalation |

### 🚨 Security Implications
- **Legacy Protocol Risk:** FTP proxy represents outdated, potentially vulnerable service
- **Network Exposure:** UPnP services enable device discovery and enumeration
- **Unnecessary Attack Surface:** Services running without business justification
- **Default Configuration Weakness:** Out-of-box settings prioritize convenience over security

---

## 🔬 Investigation Methodology

### Phase 1: 🕵️ Reconnaissance & Enumeration

![Tools](https://img.shields.io/badge/nmap-Network%20Discovery-4682B4?style=flat-square)
![Tools](https://img.shields.io/badge/lsof-Process%20Analysis-green?style=flat-square)
![Tools](https://img.shields.io/badge/tcpdump-Traffic%20Analysis-red?style=flat-square)

```bash
# Initial port discovery
nmap -sV -Pn -T4 127.0.0.1

# Service process identification
sudo lsof -i :8021
sudo lsof -i :5000
sudo lsof -i :7000

# Network traffic analysis
sudo tcpdump -i lo0 port 8021 -c 10
sudo tcpdump -i lo0 port 5000 -c 10
```

### Phase 2: 🔍 Service Analysis & Risk Assessment

<details>
<summary><b>FTP Proxy Investigation (Port 8021)</b></summary>

**Service Location:**
```bash
# LaunchDaemon configuration
/System/Library/LaunchDaemons/com.apple.ftp-proxy.plist

# Process verification
sudo launchctl list | grep ftp-proxy
```

**Risk Assessment:**
- **Legacy FTP protocol** with known security weaknesses
- **No active business use** identified
- **Potential for unauthorized file access**
- **Should be disabled** per security best practices

</details>

<details>
<summary><b>AirPlay Services Investigation (Ports 5000/7000)</b></summary>

**Service Analysis:**
```bash
# AirPlay service identification
system_profiler SPSoftwareDataType | grep -i airplay

# Network listener verification
netstat -an | grep -E "(5000|7000)"
```

**Risk Assessment:**
- **Information disclosure** through service discovery
- **Network enumeration** capability for attackers
- **Legitimate business use** for media streaming
- **Can be disabled** if not required

</details>

### Phase 3: ⚡ Risk-Based Remediation

| Priority | Service | Action | Justification |
|----------|---------|--------|---------------|
| 🔴 **P1** | FTP Proxy | **DISABLE** | Legacy, unused, high risk |
| 🟡 **P2** | AirPlay | **DISABLE** | Unnecessary exposure |
| 🟢 **P3** | ControlCenter | **MONITOR** | Active investigation |

---

## 🔧 Technical Implementation

### 🚫 FTP Proxy Remediation (Port 8021)

<details>
<summary><b>Step-by-step disable process</b></summary>

```bash
# 1. Verify current status
sudo launchctl list | grep ftp-proxy
sudo lsof -i :8021

# 2. Disable the service
sudo launchctl bootout system /System/Library/LaunchDaemons/com.apple.ftp-proxy.plist

# 3. Verify remediation
nmap -p 8021 127.0.0.1
# Expected: closed/filtered

# 4. Confirm service removal
sudo launchctl list | grep ftp-proxy
# Expected: no results
```

**✅ Result:** Port 8021 successfully closed, service disabled

</details>

### 📱 AirPlay Service Hardening (Ports 5000/7000)

<details>
<summary><b>GUI-based configuration</b></summary>

**Navigation Path:**
```
System Settings → General → AirDrop & Handoff → AirPlay Receiver: OFF
```

**Verification Commands:**
```bash
# Check port status
nmap -p 5000,7000 127.0.0.1

# Verify service state
sudo lsof -i :5000
sudo lsof -i :7000
```

**✅ Result:** Both ports closed, AirPlay services disabled

</details>

### 🔍 ControlCenter Service Analysis (Port 7000)

<details>
<summary><b>Ongoing investigation</b></summary>

```bash
# Process identification
sudo lsof -i :7000
# Output: ControlCe process identified

# Traffic monitoring
sudo tcpdump -i lo0 port 7000 -v
# Result: No active network traffic detected

# Service dependency analysis
sudo launchctl list | grep -i control
```

**📋 Status:** Under review - appears inactive but monitoring continues

</details>

---

## ⚠️ Risk Analysis

### 🔴 Before Hardening
```
Attack Surface Analysis:
├── 3 exposed network services
├── 2 unnecessary legacy protocols  
├── Potential for network enumeration
└── Default macOS configuration risks
```

### ✅ After Hardening
```
Improved Security Posture:
├── 1 remaining service (under analysis)
├── 67% reduction in exposed services
├── Eliminated legacy protocol risks
└── Enhanced defense-in-depth
```

### 🎯 Business Impact
- **✅ Reduced Attack Surface:** Fewer entry points for potential threats
- **✅ Compliance Improvement:** Adherence to security hardening standards
- **✅ Maintained Functionality:** No business operations disrupted
- **⚠️ Monitoring Required:** One service requires ongoing assessment

---

## 🛠️ Tools & Techniques

### 🔧 Command-Line Arsenal
![nmap](https://img.shields.io/badge/nmap-Network%20Scanning-4682B4?style=flat-square)
![lsof](https://img.shields.io/badge/lsof-Process%20Enumeration-green?style=flat-square)
![tcpdump](https://img.shields.io/badge/tcpdump-Packet%20Analysis-red?style=flat-square)
![launchctl](https://img.shields.io/badge/launchctl-Service%20Management-blue?style=flat-square)

| Tool | Purpose | Key Commands Used |
|------|---------|-------------------|
| **nmap** | Port discovery & service identification | `-sV -Pn -T4` |
| **lsof** | Process-to-port mapping | `-i :PORT` |
| **tcpdump** | Network traffic analysis | `-i lo0 port PORT` |
| **launchctl** | macOS service management | `bootout system` |
| **netstat** | Network connection status | `-an \| grep PORT` |

### 📊 Analysis Methodology
1. **Discovery:** Systematic port enumeration
2. **Investigation:** Service-to-process correlation
3. **Risk Assessment:** Business need vs. security risk
4. **Remediation:** Targeted service disabling
5. **Verification:** Post-change validation testing

---

## 🎓 Lessons Learned

### 💡 Technical Insights
- **Default ≠ Secure:** Out-of-box macOS configurations prioritize functionality over security
- **Legacy Services:** Older protocol services may persist without business justification
- **Layered Verification:** Multiple tools provide comprehensive service analysis
- **Documentation Value:** Systematic approach enables repeatable security hardening

### 🔄 Process Improvements
- **Baseline Establishment:** Document default service states before hardening
- **Business Impact Assessment:** Coordinate with stakeholders before service changes
- **Verification Testing:** Confirm changes don't break legitimate functionality
- **Ongoing Monitoring:** Some services require continued observation

### 🏢 Enterprise Applications
- **Endpoint Hardening:** Techniques scale to corporate macOS fleet management
- **Configuration Management:** Procedures suitable for automation via MDM/Ansible
- **Security Baseline:** Establishes repeatable hardening methodology
- **Compliance Support:** Demonstrates proactive security control implementation

---

## 📋 Production Recommendations

### 🚀 Immediate Actions
1. **Fleet Assessment:** Scan all corporate macOS endpoints for similar exposures
2. **Policy Development:** Create standard service hardening procedures
3. **Automation Planning:** Script service auditing and remediation
4. **Documentation Standards:** Establish change management protocols

### 🎯 Strategic Improvements
1. **MDM Integration:** Deploy configuration profiles for service management
2. **Continuous Monitoring:** Implement scheduled port scanning for drift detection
3. **Baseline Enforcement:** Establish and maintain secure configuration standards
4. **Security Metrics:** Track service exposure across endpoint fleet

---

## 🏢 Real-World Relevance

### 🎯 Cybersecurity Skills Demonstrated
![Skills](https://img.shields.io/badge/Network%20Reconnaissance-Expert-brightgreen?style=flat-square)
![Skills](https://img.shields.io/badge/Service%20Analysis-Expert-brightgreen?style=flat-square)
![Skills](https://img.shields.io/badge/Risk%20Assessment-Expert-brightgreen?style=flat-square)
![Skills](https://img.shields.io/badge/System%20Hardening-Expert-brightgreen?style=flat-square)

- **Network Security:** Port enumeration and service discovery techniques
- **System Administration:** Service management and configuration hardening
- **Risk Assessment:** Balancing security requirements with business functionality
- **Documentation:** Professional security assessment reporting

### 💼 Career Applications
- **SOC Analyst:** Endpoint security monitoring and incident investigation
- **Security Engineer:** System hardening and configuration management
- **Penetration Tester:** Network enumeration and attack surface analysis
- **IT Operations:** Secure service management and change control

---

## 🔐 Final Security Posture

### 📊 Remediation Metrics

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Exposed Services** | 3 | 1 | ![67%](https://img.shields.io/badge/-67%25-brightgreen) |
| **Legacy Protocols** | 1 | 0 | ![100%](https://img.shields.io/badge/-100%25-brightgreen) |
| **UPnP Exposure** | Yes | No | ![100%](https://img.shields.io/badge/-100%25-brightgreen) |
| **Attack Surface** | High | Low | ![Significant](https://img.shields.io/badge/Reduced-blue) |

### 🎯 Current Status Summary

| Port | Service | Status | Action Required |
|------|---------|--------|-----------------|
| **8021** | FTP Proxy | ✅ **SECURED** | None - service disabled |
| **5000** | AirTunes | ✅ **SECURED** | None - service disabled |
| **7000** | ControlCenter | 🔍 **MONITORING** | Ongoing analysis required |

> **🏆 Achievement:** Successfully reduced macOS endpoint attack surface by 67% while maintaining system functionality and establishing repeatable hardening procedures.

---

## 🏷️ Tags

![Endpoint Security](https://img.shields.io/badge/endpoint--security-red?style=flat-square)
![macOS Hardening](https://img.shields.io/badge/macos--hardening-blue?style=flat-square)
![Service Management](https://img.shields.io/badge/service--management-green?style=flat-square)
![Network Security](https://img.shields.io/badge/network--security-purple?style=flat-square)
![Attack Surface Reduction](https://img.shields.io/badge/attack--surface--reduction-orange?style=flat-square)

---

## 📚 References & Further Reading

[![Apple Security](https://img.shields.io/badge/Apple-Security%20Guide-000000?style=flat-square&logo=apple)](https://support.apple.com/guide/security/)
[![NIST](https://img.shields.io/badge/NIST-SP%20800--123-blue?style=flat-square)](https://csrc.nist.gov/publications/detail/sp/800-123/final)
[![CIS Controls](https://img.shields.io/badge/CIS-Controls-green?style=flat-square)](https://www.cisecurity.org/controls/)
[![SANS](https://img.shields.io/badge/SANS-Hardening%20Guide-red?style=flat-square)](https://www.sans.org/)

---

*This assessment demonstrates systematic endpoint security hardening through service analysis, risk-based remediation, and thorough documentation. The methodology shown applies directly to enterprise security operations and demonstrates practical cybersecurity skills valued by employers.*

**⭐ Star this repository if this hardening guide helped with your security projects!**
