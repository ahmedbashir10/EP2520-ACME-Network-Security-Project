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

## **Network Overview**

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

## DuckDNS Setup (Dynamic WAN IP Resolution)**

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

## WireGuard Configuration**

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

## Verifying and Troubleshooting**

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

## Maintaining the Connection**

- Keep **WireGuard ON** for access to Stockholm/London.
- If you **change networks**, restart the tunnel.
- Regularly verify **DuckDNS updates your WAN IP**.

---

## FreeIPA

---

## FreeRadius

---

## Nextcloud

---

## Snort
The IDS is implemented using Snort. Installation is as follows:

### Installation 
On the machine, do:

```
sudo apt update && sudo apt upgrade -y
sudo apt-get install snort -y
```

On the router, do:

```
opkg install iptables
opkg install iptables-mod-tee
```

### Configuration
To mirror incoming traffic from the router, do:

```
iptables -t mangle -A PREROUTING -d 192.168.2.0/24 -j TEE --gateway 192.168.2.10
```

Note: Change IPs depending on your setup.

Change your HOME\_NET to the network which is to be monitored, in this case it is 192.168.2.0/24. For EXTERNAL\_NET, the "inverse" of the HOME\_NET is set

`/etc/snort/snort.conf`

```
ipvar HOME_NET 192.168.2.0/24
```

Create some simple custom rules to try potential malicious activity:

`/etc/snort/rules/local.rules`

```
alert icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"ICMP Ping Sucker!"; sid:1000001;)
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 (msg:"SSH Connection Attempt"; sid:1000002;)
alert tcp $EXTERNAL_NET any -> $HOME_NET 21 (msg:"FTP login attempt"; sid:1000003;)
```

### Run Snort

```
sudo snort -c /etc/snort/snort.conf -i enp0s1 -A fast
```

This should create alerts and log them. To test, try in another terminal window

```
sudo tail -f /var/log/snort/alert
```

## Fail2Ban

### Installation 

```
sudo apt update && sudo apt install fail2ban -y
```

### Configuration

`/etc/fail2ban/filter.d/snort.conf`

```
[Definition]
failregex = ^.*\s+<HOST>\s+->\s+.*$
ignoreregex =
```

Configure the file that decides how rules and bans are applied

`/etc/fail2ban/jail.local`

```
[snort]
enabled = true
filter = snort
logpath = /var/log/snort/alert
backend = auto
maxretry = 3
findtime = 600
bantime = 3600
ignoreip = 10.0.2.2 10.0.3.3 10.0.1.1
action = iptables-ssh[name=snort, port=all, protocol=all]
datepattern = %%m/%%d-%%H:%%M:%%S
```

Note: ignoreip describes which IPs not to block. The IPs describe the tunnels for the VPN. Make sure that logpath is where Snort saves the logs. 

Add an action file that executes actions such as bans, unbans

`/etc/fail2ban/action.d/iptables-ssh.conf`

```
[INCLUDES]
before = iptables-common.conf

[Definition]
# Setup
actionstart = /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -N f2b-snort"
              /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -A f2b-snort -j RETURN"
              /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -I INPUT -j f2b-snort"                         

# Cleanup
actionstop = /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -D INPUT -j f2b-snort"            
             /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -X f2b-snort"

# Check
actioncheck = /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -n -L INPUT | grep -q 'f2b-snort'"

# Ban IP
actionban = /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -I f2b-snort 1 -s <ip> -j DROP"                       
# Unban IP
actionunban = /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -D f2b-snort -s <ip> -j DROP"                       
[Init]
banaction = /usr/bin/ssh -i /home/ritze/.ssh/id_rsa root@192.168.2.1 "iptables -w -C INPUT -j f2b-snort || iptables -w -I INPUT -j f2b->
```

Note: As the fail2ban needs access to the router, an SSH key-pair is created to perform the actions. This can be done by

```
ssh-keygen -t rsa -b 2048
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.2.1
```

### Start Fail2Ban

```
sudo systemctl start fail2ban
sudo systemctl status fail2ban
```

If correctly setup, a machine performing, for example, an ICMP request towards the router should be banned from the network. 

---


