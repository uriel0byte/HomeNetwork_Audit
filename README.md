# Home Network Security Audit & Hardening Report

## 1. Executive Summary

Performed a vulnerability assessment and security hardening of an ISP-provided residential gateway. The assessment used Fing (iOS) for network reconnaissance and the router's web management interface for configuration changes. Four findings were identified: deprecated wireless encryption, WPS enabled by default, firewall inactive, and legacy unencrypted protocols open on the LAN. Three were fully remediated. The fourth is an accepted risk due to ISP firmware restrictions that prevent customer-level changes to LAN ACL rules.

---

## 2. Scope & Environment

| Field | Detail |
|-------|--------|
| Target Device | Skyworth GN542VF (ISP: True Online) |
| Local Subnet | `192.168.1.0/24` |
| Assessment Tools | Fing Network Scanner (iOS), Router Web Management Interface |

---

## 3. Initial Reconnaissance & Baseline

An ARP sweep and port scan were run against the local subnet to establish a baseline inventory.

**Device Inventory:** Identified all active endpoints. Generic hostnames were resolved using MAC address lookup to identify specific devices, including a Smart Camera and TrueID TV Box.

**Open Ports (Gateway):**

| Port | Service | Notes |
|------|---------|-------|
| 53 | DNS | Expected |
| 80 | HTTP | Router management interface |
| 5555 | rplay | Identified by Fing; likely the TrueID TV Box's AirPlay receiver service |

Port 5555 running rplay is expected behavior for a media box with AirPlay support. It's worth noting as an open service — if the TrueID box has unpatched firmware vulnerabilities, this port is a potential entry point for anyone already on the network.

![Open Ports Scanning](screenshots/01_recon_and_baseline/IMG_1595.jpeg)

---

## 4. Vulnerability Findings & Remediation

### Finding 1: Deprecated Cryptographic Protocols

**Severity:** High
> TKIP vulnerabilities give an attacker a direct path to recovering the network passphrase without needing physical access. Mixed mode makes it worse — the network is only as strong as its weakest client.

**Observation:** Both the 2.4GHz and 5GHz bands were running WPA-PSK/WPA2-PSK Mixed Mode with TKIP/AES encryption.

**Risk:** TKIP is obsolete and vulnerable to downgrade and key-recovery attacks. Mixed mode is the problem — it allows a client or attacker to force negotiation down to TKIP, effectively bypassing the stronger AES protection entirely.

**Remediation:** Enforced WPA2-PSK with AES (CCMP) exclusively across both bands. Mixed mode disabled.

**Troubleshooting:** Applying the change to the 5GHz band returned a firmware error: `"This SSID has already been used by SSID6"`. Investigation revealed a hidden duplicate SSID profile (`ChaiWat_5G_Ghost`) causing an internal conflict. The ghost profile was renamed and disabled, which cleared the conflict and allowed the configuration change to apply successfully.

![Deprecated Protocols](screenshots/02_vulneralbility_findings/IMG_1601.jpeg)

---

### Finding 2: WPS Enabled by Default

**Severity:** High
> Password complexity is irrelevant here. The Pixie Dust attack bypasses the WPA2 passphrase entirely by attacking the WPS PIN exchange — a strong password provides zero additional protection while WPS is active.

**Observation:** WPS was enabled on both wireless bands out of the box.

**Risk:** WPS PIN authentication is vulnerable to the Pixie Dust attack, an offline cryptographic attack that exploits weak nonce generation in WPS implementations to recover the plaintext WPA2 passphrase regardless of password complexity.

**Remediation:** Disabled WPS on all active radios.

---

### Finding 3: Firewall Disabled, WAN Ping Responding

**Severity:** Medium
> Unlike Findings 1 and 2, this doesn't hand an attacker the passphrase directly. The risk here is increased exposure and discoverability — a disabled firewall with a responding WAN ping makes the device an easier target, but exploitation still requires an additional step.

**Observation:** The gateway's SPI (Stateful Packet Inspection) firewall was off by default. The WAN interface was also configured to respond to external ICMP ping requests.

**Risk:** A disabled SPI firewall removes the primary filter against unsolicited inbound traffic. A responding WAN ping makes the device visible to automated internet-wide scanners, increasing exposure to opportunistic attacks.

**Remediation:** Enabled the IPv4 SPI firewall (Low setting) and disabled WAN ping response. External ICMP requests are now dropped silently.

![Enabled IPv4 Firewall](screenshots/03_remidation_applied/IMG_1607.jpeg)

---

### Finding 4: Legacy Protocols Open on LAN (Unresolved)

**Severity:** Medium
> This requires an attacker to already have LAN access, which is why it sits at Medium rather than High. It's a meaningful post-compromise risk, but not an initial access vector on its own. Remediating Finding 1 reduces the likelihood of unauthorized LAN access in the first place.

**Observation:** The LAN ACL has Telnet, FTP, and TFTP enabled on the LAN interface.

**Risk:** Telnet transmits all data in cleartext, including credentials. If any internal device is compromised, an attacker with LAN access can intercept administrative sessions via passive sniffing.

**Remediation Status:** Unresolved — Accepted Risk.

**Notes:** The ISP firmware uses hardened RBAC that restricts customer accounts from modifying LAN ACL rules. These ports are almost certainly open for ISP field technician access and TR-069 remote management. There is no legitimate way to close them at the customer level without flashing custom firmware, which would void ISP support and likely break provisioning. Residual risk is partially offset by the WPA2-AES enforcement from Finding 1, which raises the bar for unauthorized LAN access. This remains a documented limitation.

---

### Screenshots

📁 [View all before/after configuration screenshots](screenshots/)

---

## 5. Summary

| Finding | Severity | Status |
|---------|----------|--------|
| TKIP/Mixed Mode wireless encryption | High | Remediated |
| WPS enabled by default | High | Remediated |
| SPI firewall disabled, WAN ping active | Medium | Remediated |
| Telnet/FTP/TFTP open on LAN (ISP firmware) | Medium | Accepted Risk |

Three of four findings were fully remediated through the router's web interface. The remaining finding is a vendor limitation with no available fix at the customer privilege level. It is documented here as an accepted risk, not an oversight.
