# ACME Secure Network Project

## Overview
This project aims to design and implement a secure IT infrastructure for ACME Scandinavia, connecting its headquarters in Stockholm with its branch office in London. The focus is on providing a robust, scalable, and secure network solution while ensuring employee authentication, secure communication, and remote access capabilities.

## Project Goals
- Establish a secure VPN connection between Stockholm and London offices.
- Implement strong authentication using xPKI.
- Set up firewalls, intrusion detection, and secure file-sharing mechanisms.
- Provide secure wireless access for employees using corporate devices.
- Ensure confidentiality and integrity of communication between offices.

## Network Topology
The network comprises two main locations:

![Network Topology](topology.drawio.png)


### Stockholm (Headquarters)
- **Servers**: Web server, Access server, DNS server, Intrusion Detection System (IDS)
- **Authentication**: Public Key Infrastructure (xPKI)
- **Devices**: PCs, laptops, tablets, mobile devices
- **Network Components**: Switch (NETGEAR FS105), Firewall, VPN Router (Asus RT-AX53J)

### London (Branch Office)
- **Servers**: DNS Server
- **Devices**: PCs, laptops, tablets, mobile devices
- **Network Components**: Switch (NETGEAR FS105), Firewall, VPN Router (Asus RT-AX53U)

## Security Features
- **VPN Encryption**: Secure tunneling for remote access and inter-office communication.
- **xPKI Authentication**: Digital certificates for user authentication and secure access.
- **Intrusion Detection System**: Monitors network traffic for potential threats.
- **Firewall Protection**: Restricts unauthorized access and prevents cyber threats.
- **Secure Wireless Access**: Employees connect securely via Wi-Fi.
- **Two-Factor Authentication (2FA)**: Additional security for accessing critical resources.


## Usage Instructions
1. **Connect to VPN**: Employees must authenticate using their xPKI credentials.
2. **Access Resources**: Secure access to the web server, file-sharing platform, and internal systems.
3. **Report Security Events**: IDS logs any suspicious activity for review.
4. **Wireless Access**: Employees must authenticate using secure credentials before connecting to Wi-Fi.

-------------------------------------------------
# ReadMe: Secure Remote Access & London-Stockholm Connection via WireGuard

## Overview

This document provides a **step-by-step guide** for setting up secure remote access using **WireGuard VPN** to connect Home (10.0.2.2/24) and London (10.0.1.1/24) to Stockholm (10.0.0.1/24). Additionally, it covers **firewall rules, routing configurations, dynamic DNS setup (DuckDNS), and troubleshooting**.

WireGuard was chosen due to its **high-speed cryptographic implementation**, simplicity, and ease of deployment across various platforms. This VPN solution ensures encrypted and authenticated communication between network nodes while minimizing latency.

---

## 1. System Configuration

### **Network Overview**

| **Device** | **Role** | **Interface** | **IP Address** |
| --- | --- | --- | --- |
| **Stockholm** | WAN IP | wan | 130.237.11.40  |
| **Stockholm** | WireGuard Server | wg0 | 10.0.0.1/24 |
| **Stockholm** | LAN | br-lan | 192.168.2.1/24 |
| **London** | WireGuard Client | wg0 | 10.0.1.1/24 |
| **London** | WAN IP | wan | 130.237.11.54 |
| **London** | LAN | br-lan | 192.168.1.1/24 |
| **Home** | WireGuard Client | wg0 | 10.0.2.2/24 |

This guide ensures that **WAN IP updates dynamically**, **WireGuard clients always connect via DuckDNS**, and **firewall and routing configurations remain automated**.

---

## **2. DuckDNS Setup (Dynamic WAN IP Resolution)**

To prevent manual IP changes, we configure **DuckDNS** to update Stockholm's public IP automatically.

### **Install DDNS Packages**

```
opkg update
opkg install ddns-scripts ddns-scripts-services luci-app-ddns
```

### **Configure DuckDNS**

Edit `/etc/config/ddns`:

```
config service 'myddns'
    option enabled '1'
    option lookup_host 'stockholmvpn.duckdns.org'
    option domain 'stockholmvpn.duckdns.org'
    option password 'c4b767f6-b3d4-4e1d-a30c-85a8ebe91e0c'  # Replace with your DuckDNS token
    option interface 'wan'
    option ip_source 'network'
    option ip_network 'wan'
    option service_name 'duckdns.org'
    option use_https '1'
    option check_interval '0'    # No scheduled checks, only update on WAN change
    option retry_interval '0'     # No retries needed
    option force_interval '0'     # No forced updates
```

Enable and restart DDNS:

```
/etc/init.d/ddns enable
/etc/init.d/ddns restart
```

---

## **3. WireGuard Configuration**

### **Stockholm (WireGuard Server)**

Configuration File `/etc/config/network`:

```
config interface 'wg0'
    option proto 'wireguard'
    option private_key 'STOCKHOLM_PRIVATE_KEY'
    option listen_port '51820'
    list addresses '10.0.0.1/24'

config wireguard_wg0
    option description 'London'
    option public_key 'LONDON_PUBLIC_KEY'
    list allowed_ips '10.0.1.1/32'
    list allowed_ips '192.168.1.0/24'
    option persistent_keepalive '25'

config wireguard_wg0
    option description 'Home'
    option public_key 'HOME_PUBLIC_KEY'
    list allowed_ips '10.0.2.2/32'
    option persistent_keepalive '25'
```

### **London (WireGuard Client)**

Configuration File `/etc/config/network`:

```
config interface 'wg0'
    option proto 'wireguard'
    option private_key 'LONDON_PRIVATE_KEY'
    option listen_port '51820'
    list addresses '10.0.1.1/24'

config wireguard_wg0
    option description 'Stockholm'
    option public_key 'STOCKHOLM_PUBLIC_KEY'
    option endpoint_host 'stockholmvpn.duckdns.org'
    option endpoint_port '51820'
    list allowed_ips '10.0.0.1/32'
    list allowed_ips '192.168.2.0/24'
    option persistent_keepalive '25'
```

### **Home (WireGuard Client - Phone/Laptop)**

WireGuard App Configuration:

```
[Interface]
PrivateKey = YOUR_PRIVATE_KEY
Address = 10.0.2.2/24
DNS = 10.0.0.1

[Peer]
PublicKey = STOCKHOLM_PUBLIC_KEY
Endpoint = stockholmvpn.duckdns.org:51820
AllowedIPs = 10.0.0.0/24, 10.0.1.0/24, 10.0.2.0/24
PersistentKeepalive = 25
```

---

## **4. Verifying and Troubleshooting**

### **Check System Status**

| **Test** | **Command** | **Expected Result** |
| --- | --- | --- |
| Check WAN IP | `ip addr show wan` | Matches DuckDNS |
| Check DuckDNS Update | `nslookup stockholmvpn.duckdns.org` | Matches WAN IP |
| Check WireGuard Status | `wg show` | Clients connected |
| Check Routes | `ip route show` | `grep 10.0.` | Shows correct subnets |
| Check Firewall | `nft list ruleset` | `grep 51820` | WireGuard traffic allowed |
| View DDNS Logs | `logread -e ddns` | Shows updates and IP changes |

---

## **5. Maintaining the Connection**

- Keep **WireGuard ON** for access to Stockholm/London.
- If you **change networks**, restart the tunnel.
- Regularly verify **DuckDNS updates your WAN IP**.

---

## **🎯 Summary**

✅ **WAN IP changes → DuckDNS updates automatically.**
✅ **WireGuard, Firewall, and Routing always work dynamically.**
✅ **No manual IP changes needed, ever.**

🔥 Your setup is now fully automated and secure! 🚀

