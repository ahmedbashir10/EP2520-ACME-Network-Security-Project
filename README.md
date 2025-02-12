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

## Implementation Requirements
- **Hardware**: Switches, routers, firewalls, corporate laptops, and mobile devices.
- **Software**: VPN software, xPKI certificate management, IDS software, Nextcloud for secure file sharing.
- **Network Configuration**: Proper IP address management, DNS settings, and firewall rules.

## Project Timeline
- **Phase 1 (Solution Design)**: Requirement analysis and system design (Deadline: Feb 17, 2025)
- **Phase 2 (Implementation)**: Deployment and testing of the solution (Deadline: Mar 10, 2025)
- **Phase 3 (Presentation & Demo)**: Showcasing the implemented system (Deadline: Mar 14, 2025)

## Usage Instructions
1. **Connect to VPN**: Employees must authenticate using their xPKI credentials.
2. **Access Resources**: Secure access to the web server, file-sharing platform, and internal systems.
3. **Report Security Events**: IDS logs any suspicious activity for review.
4. **Wireless Access**: Employees must authenticate using secure credentials before connecting to Wi-Fi.


