# # Home Network Security Audit & Hardening Report

## 1. Executive Summary
Conducted a localized vulnerability assessment and perimeter hardening of an ISP-provided residential gateway. The objective was to identify misconfigurations, deprecate obsolete cryptographic protocols, and establish a baseline defense posture using mobile-centric diagnostic tools. Successfully remediated critical wireless vulnerabilities and navigated vendor-locked firmware to maximize network security.

## 2. Scope & Environment
* **Target Device:** Skyworth GN542VF (ISP: True Online)
* **Local Subnet:** `192.168.1.0/24`
* **Assessment Tools:** Fing Network Scanner (iOS), Safari (Web Interface Management)

## 3. Initial Reconnaissance & Baseline
An initial local ARP sweep and port scan were conducted to map the network baseline.
* **Device Inventory:** Successfully identified active endpoints, resolving generic hostnames via MAC address tables to identify specific IoT devices (Smart Camera, TrueID TV Box).
* **Open Ports (Gateway):** Identified Port 80 (HTTP), Port 53 (DNS), and Port 5555 (rplay/ADB - likely utilized by ISP media boxes).

![Open Ports Scanning]()

## 4. Vulnerability Findings & Remediation

### Finding 1: Deprecated Cryptographic Protocols
* **Observation:** Both the 2.4GHz and 5GHz bands were configured to WPA-PSK/WPA2-PSK Mixed Mode utilizing TKIP/AES encryption. 
* **Risk:** TKIP is an obsolete standard vulnerable to downgrade and key-recovery attacks. Mixed mode allows an attacker to force the network to use the weaker TKIP protocol.
* **Remediation:** Enforced strict **WPA2-PSK** security and **AES** (CCMP) encryption across all broadcast bands.
* **Troubleshooting Note:** During remediation of the 5GHz band, a firmware logic error occurred (`"This SSID has already been used by SSID6"`). Discovered a hidden, duplicate SSID profile causing an internal conflict. Renamed and disabled the ghost profile (`ChaiWat_5G_Ghost`), which allowed the security patch to be successfully applied to the primary interface.

![Deprecated Protocols]()

### Finding 2: Wi-Fi Protected Setup (WPS) Enabled
* **Observation:** WPS was enabled on both wireless bands by default.
* **Risk:** WPS split-PIN authentication is highly susceptible to offline brute-force attacks (e.g., Pixie Dust), which can expose the plaintext WPA2 passphrase regardless of complexity.
* **Remediation:** Disabled WPS functionality on all active radios.

### Finding 3: Perimeter Defense & Stealth Mode Inactive
* **Observation:** The gateway's Stateful Packet Inspection (SPI) firewall was disabled, and the WAN interface was configured to respond to external ICMP (Ping) requests.
* **Risk:** Leaves the network discoverable to automated internet-wide scanners and removes the primary barrier against unsolicited ingress traffic.
* **Remediation:** Enabled the IPv4 Firewall (Low/Standard setting) and disabled WAN Ping to drop external ICMP requests, placing the edge device in "stealth mode."

![Enabled IPv4 Firewall]()

### Finding 4: Insecure Local Protocols (Vendor Locked)
* **Observation:** Within the Access Control List (ACL), unencrypted legacy protocols including Telnet, FTP, and TFTP are enabled on the Local Area Network (LAN) interface.
* **Risk:** If an internal endpoint is compromised, Telnet traffic can be sniffed to capture administrative credentials in plain text.
* **Remediation Status:** **Unresolved (Accepted Risk / Vendor Limitation).**
* **Notes:** The ISP utilizes customized firmware with strict Role-Based Access Control (RBAC). The customer-level administrative account lacks the necessary privileges to modify LAN ACL rules. These ports are likely hardcoded open for ISP field technician diagnostics and TR-069 remote management. The risk is mitigated by the newly enforced WPA2-AES wireless encryption, significantly reducing the likelihood of unauthorized LAN access.

## 5. Conclusion
By migrating away from TKIP, disabling WPS, activating the firewall, and documenting ISP limitations, the network's security posture has been significantly elevated from a vulnerable default state to a hardened perimeter. 
