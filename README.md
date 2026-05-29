# Home-network-audit
Live network security audit using Nmap — host discovery, port scanning, vulnerability assessment
**Type:** Personal Home Network Penetration Test  
**Scope:** Full subnet scan — router, laptops, IoT devices, mobile devices  
**Tools Used:** Nmap, manual service enumeration  
**Status:** Findings documented — remediation in progress

---

## Executive Summary

Conducted a full network scan of a home LAN environment to identify exposed services, misconfigurations, and attack surface across all connected devices. Identified **2 high-risk**, **3 medium-risk**, and **4 low-risk** findings. Key issues include UPnP enabled on the router, an unauthenticated SOCKS5 proxy on an IoT device, and SMB open on a Windows host.

---

## Findings

### 🔴 HIGH — Router (Gateway Device)
**Device:** ISP-provided router (Sagemcom chipset, common in UK deployments)

| Port | Service | Finding |
|------|---------|---------|
| 53 | dnsmasq 2.87 | DNS server exposed on LAN. Older dnsmasq versions carry critical CVEs — cross-referenced NVD |
| 80 / 443 | lighttpd 1.4.63 | Admin panel accessible over both HTTP and HTTPS. Verified default credentials had been changed |
| 49153 | UPnP | **Open and enabled.** UPnP auto-opens firewall ports without user confirmation — a known malware abuse vector |
| 22, 23, 8080 | SSH / Telnet / HTTP proxy | Filtered (not closed) — firewall blocking but services may exist behind it |

**Remediation taken:** Disabled UPnP via router admin panel. Confirmed non-default admin password in use.

---

### 🔴 HIGH — Windows Workstation
**Device:** Windows 10/11 laptop (personal machine)

| Port | Service | Finding |
|------|---------|---------|
| 445 | SMB | Open. This is the port exploited by EternalBlue/WannaCry in 2017. Patching verified |
| 139 | NetBIOS | Legacy Windows file sharing — unnecessary exposure on a home network |
| 135 | RPC | Standard Windows service, noted for completeness |
| 7070 | Unknown SSL | Investigated with `nmap -sV -p 7070` — identified as a local application listener, not externally reachable |

**Remediation taken:** Confirmed Windows fully patched. Flagged NetBIOS for disabling. Investigated port 7070 — determined safe.

---

### 🟡 MEDIUM — Amazon Smart Device
**Device:** Amazon Echo / FireTV (identified by MAC OUI)

| Port | Service | Finding |
|------|---------|---------|
| 1080 | SOCKS5 proxy | **No authentication required.** Any device on the LAN could route traffic through this host anonymously — unusual and worth flagging |
| 8888 | Unknown | Web interface accessible — investigated via browser |

**Notes:** SOCKS5 without auth on a consumer device is an unexpected finding. Likely a developer/debug interface left open. Documented and flagged to household — device is under review.

---

### 🟡 MEDIUM — Unknown Mobile Device
**Device:** Likely iPhone (confirmed by port + MAC address randomisation behaviour)

| Port | Service | Finding |
|------|---------|---------|
| 62078 | iTunes / iPhone sync | Standard iOS port. Device identified as household member's phone — not a rogue device |
| 49152 | Dynamic port | Low concern |

**Notes:** MAC randomisation initially made this appear as an unknown device. Correlating port 62078 with iOS behaviour confirmed identity.

---

### 🟡 MEDIUM — IoT Dongle
**Device:** Hui Zhou Gaoshengda chipset — pattern matches Chromecast or smart TV dongle

| Port | Service | Finding |
|------|---------|---------|
| 8008, 8009, 8443, 9000 | Multiple services | Elevated attack surface for an IoT device. IoT devices rarely receive security patches |

**Remediation taken:** Device moved to isolated guest WiFi network — segmented from main LAN.

---

### ⚪ LOW — Clean Devices

| Device | Finding |
|--------|---------|
| Sony PlayStation | All ports closed — no findings |
| Intel laptop | All ports closed — no findings |
| Two unknown devices | All ports closed — likely phones; low concern |
| New device (joined mid-scan) | All ports closed — identified as a guest device |

---

## Remediation Summary

| Action | Status |
|--------|--------|
| Disable UPnP on router | ✅ Done |
| Verify router admin password is non-default | ✅ Confirmed |
| Confirm Windows is fully patched (SMB/EternalBlue) | ✅ Done |
| Investigate port 7070 on Windows host | ✅ Resolved — local app, not a threat |
| Investigate Amazon SOCKS5 proxy (port 1080) | 🔄 Under review |
| Move IoT device to guest network | ✅ Done |
| Disable NetBIOS on Windows host | 🔄 Pending |

---

## Key Takeaways

- **UPnP is dangerous on home networks** — it silently opens firewall ports on behalf of any device or application that asks. Disabled immediately.
- **IoT devices expand your attack surface significantly** — isolating them to a guest VLAN/network is best practice even if no specific vulnerability is found.
- **MAC address randomisation** complicates device identification during scans — correlating ports and OUI data helps resolve unknowns.
- **SMB (port 445) should always be patched** — EternalBlue is years old but still exploited in the wild. Patch discipline matters.
- **An unauthenticated proxy on a consumer device is a notable anomaly** — worth escalating even on a home network.

---

## Tools & Methodology

```bash
# Initial host discovery
nmap -sn 192.168.x.0/24

# Full port scan on discovered hosts
nmap -sV -O -p- [host]

# Targeted service investigation
nmap -sV -p 7070 [windows-host]
```

*Note: All IPs anonymised. Scans performed on a private residential network with full authorisation (own network).*


file:///C:/Users/USER/Downloads/cyber_network_scanner_1.html
