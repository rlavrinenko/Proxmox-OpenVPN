# Proxmox-OpenVPN
PVE OpenVPN Access — модуль VPN-доступа и Site-to-Site для Proxmox VE
PVE OpenVPN Access — VPN Access and Site-to-Site Module for Proxmox VE

PVE OpenVPN Access is a local extension for Proxmox VE that integrates directly into the native Proxmox Web interface on port 8006.

The module turns Proxmox into a centralized OpenVPN gateway for:
- remote user access;
- Site-to-Site connections;
- connecting Proxmox local networks with remote networks behind MikroTik;
- restricting users to specific IP addresses and ports;
- managing routing and firewall rules directly from the Proxmox interface.

No separate management server or additional Web panel is required.

Module access:

https://PROXMOX-IP:8006

Node
└── OpenVPN Access


MAIN FEATURES

- Create multiple independent OpenVPN servers on one Proxmox node.
- TCP/UDP support.
- OpenVPN port configuration.
- VPN subnet configuration.
- Public server address configuration.
- NAT / Masquerade and Routed modes.
- Enable and disable OpenVPN servers.
- Edit existing servers.

- Create individual VPN users.
- Create MikroTik Site-to-Site peers.
- Automatic fixed VPN IP assignment.
- Manual VPN IP assignment.
- Enable and disable users.
- Edit existing users and peers.

- Automatic CA creation.
- Automatic server certificate generation.
- Automatic client certificate generation.
- Automatic OpenVPN configuration generation.
- Ready-to-use .ovpn profiles.

- Restrict a VPN user to a specific internal IP address.
- Restrict access to specific TCP/UDP ports.
- Allow one user to access multiple internal servers.
- Default DENY firewall policy.
- Everything that is not explicitly allowed is blocked.

- Works with existing Proxmox Linux Bridge interfaces:
  vmbr0
  vmbr1
  vmbr2
  and others.

- Multiple Proxmox local networks.
- Multiple networks behind MikroTik.
- Proxmox ↔ MikroTik Site-to-Site routing.
- Automatic OpenVPN route generation.
- Automatic OpenVPN iroute generation.
- Automatic Linux routing configuration.
- Automatic nftables configuration.

- MikroTik RouterOS configuration generation.
- Self-contained .rsc generation.
- Automatic MikroTik route configuration.
- Automatic MikroTik firewall configuration.
- Multiple remote subnets behind one MikroTik peer.

- Site-to-Site directions:
  Proxmox → MikroTik
  MikroTik → Proxmox
  Both directions

- OpenVPN profiles compatible with MikroTik RouterOS.
- For MikroTik TCP connections:
  proto tcp

- For UDP:
  proto udp

- Supported TLS control modes:
  None
  tls-auth

- Additional TLS control modes may be available for standard OpenVPN clients depending on server configuration.

- Automatic configuration restoration after Proxmox reboot.
- OpenVPN diagnostics.
- Proxmox API integration diagnostics.
- PKI and configuration preservation during module updates.


REMOTE USER ACCESS

For a standard VPN user, access can be limited only to required internal resources.

Example:

User:
office

VPN IP:
10.250.10.10

Allowed resources:

vmbr0
└── 192.168.99.20
    └── TCP 3389

vmbr1
└── 10.10.20.15
    ├── TCP 22
    └── TCP 443


The user can access only:

192.168.99.20:3389

10.10.20.15:22
10.10.20.15:443


Access attempts to other addresses are blocked.

For example:

192.168.99.21
192.168.99.1
10.10.20.16
Proxmox management
other local networks


The client can receive only the required routes.

Example:

192.168.99.20/32
10.10.20.15/32

Instead of receiving a route to the entire local network.


MIKROTIK SITE-TO-SITE

A dedicated VPN peer is created for MikroTik.

Example:

Server:
CLOUD

Peer:
mikrot

VPN IP:
10.250.10.20

Type:
MikroTik Site-to-Site peer


A single MikroTik peer can be assigned multiple Proxmox networks.

Example:

Proxmox networks:

vmbr0
192.168.99.0/24

vmbr1
10.10.10.0/24

vmbr2
172.20.0.0/24


And multiple networks behind MikroTik:

192.168.55.0/24
192.168.60.0/24
10.20.0.0/16


Logical structure:

CLOUD / mikrot
│
├── Proxmox networks
│   ├── vmbr0 / 192.168.99.0/24
│   ├── vmbr1 / 10.10.10.0/24
│   └── vmbr2 / 172.20.0.0/24
│
└── MikroTik networks
    ├── 192.168.55.0/24
    ├── 192.168.60.0/24
    └── 10.20.0.0/16


The OpenVPN server automatically creates routes:

route 192.168.55.0 255.255.255.0
route 192.168.60.0 255.255.255.0
route 10.20.0.0 255.255.0.0


For the specific MikroTik peer, the module automatically creates:

iroute 192.168.55.0 255.255.255.0
iroute 192.168.60.0 255.255.255.0
iroute 10.20.0.0 255.255.0.0


This allows OpenVPN to know that these networks are located behind this specific MikroTik peer.


SITE-TO-SITE EXAMPLE

Proxmox LAN:

192.168.99.0/24

        │
        ▼

     Proxmox
     OpenVPN

        │
        ▼

     MikroTik

        │
        ▼

MikroTik LAN:

192.168.55.0/24


This provides full routed connectivity:

192.168.99.0/24
↕
OpenVPN
↕
192.168.55.0/24


MIKROTIK CONFIG

The module generates a dedicated RouterOS configuration for MikroTik.

A MikroTik-compatible OpenVPN profile is used.

For TCP:

proto tcp

For UDP:

proto udp


Example MikroTik OpenVPN profile:

client
dev tun
proto tcp
remote VPN_SERVER_IP 1194
auth SHA256
cipher AES-256-CBC


The module can generate a self-contained RouterOS .rsc file.

The file contains:

- OpenVPN profile;
- OpenVPN interface creation;
- connection settings;
- routes;
- firewall rules;
- Site-to-Site configuration.


After applying the configuration, MikroTik automatically receives routes to Proxmox networks.

Example:

/ip/route add \
dst-address=192.168.99.0/24 \
gateway=pve-mikrot


FIREWALL

The presence of a route alone does not automatically grant access.

The module creates a dedicated firewall policy.

Logic:

Source
+
Destination
+
VPN User / Peer
+
Direction
+
Protocol
+
Ports
=
ALLOW


Everything else:

DROP


Example:

192.168.99.0/24
→
192.168.55.0/24

ALLOW


192.168.99.0/24
→
192.168.60.0/24

ALLOW


VPN
→
any other networks

DROP


For a user, access can be limited to:

TCP 3389

or:

TCP 22,443,3389

or:

UDP 53

or full access to only one specific IP address.


SITE-TO-SITE DIRECTIONS

Available options:

PVE → MikroTik

MikroTik → PVE

Both directions


Example:

Proxmox network:
192.168.99.0/24

MikroTik network:
192.168.55.0/24

Both directions


This allows:

192.168.99.0/24
→
192.168.55.0/24


and:

192.168.55.0/24
→
192.168.99.0/24


NAT / MASQUERADE

Masquerade mode can be used for standard remote users.

Traffic flow:

VPN User
↓
Proxmox
↓
NAT
↓
LAN Server


Advantage:

No return route to the VPN subnet needs to be configured on the existing LAN gateway.


ROUTED MODE

If internal servers must see the real VPN user address:

10.250.10.x

Routed mode can be used.

In this case, the local network must have a return route:

10.250.10.0/24
via PROXMOX_IP


PROXMOX INTERFACE

The module integrates directly into the native Proxmox VE panel.

Node
└── OpenVPN Access
    ├── Servers
    ├── Users / Peers
    ├── LAN Access
    └── MikroTik / Routes


SERVERS

OpenVPN server management:

- Create
- Edit
- Enable
- Disable
- Remove
- Restart


USERS / PEERS

Management of:

- VPN users
- MikroTik peers
- Fixed VPN IP
- Enabled / Disabled status
- Download .ovpn
- MikroTik Config
- Edit
- Remove


LAN ACCESS

Individual user access rules:

- Bridge
- Destination IP
- Protocol
- Ports
- Edit
- Remove


MIKROTIK / ROUTES

Site-to-Site configuration:

- MikroTik peer
- multiple Proxmox networks
- multiple MikroTik networks
- direction
- automatic route
- automatic iroute
- nftables
- RouterOS configuration


ARCHITECTURE

The native Proxmox Web interface is used:

https://PROXMOX-IP:8006


Architecture:

pveproxy :8006
    │
    ▼
Proxmox API
    │
    ▼
pvedaemon
    │
    ├── pve-ovpnctl
    ├── OpenVPN
    ├── Easy-RSA
    ├── PKI
    ├── nftables
    ├── Linux routing
    └── OpenVPN CCD / iroute


No separate Web server is used.

No additional management port is used.

The module does not require:

9443
8080
separate VM
separate Docker container
separate management server


All operations are performed directly on the Proxmox VE node.


SECURITY

VPN configuration is stored locally on Proxmox.

Private keys and configuration files use restricted file permissions.

Privileged operations are executed through the Proxmox API / pvedaemon.

The firewall follows a:

DENY BY DEFAULT

policy.


A VPN user receives only explicitly permitted access.


ADDITIONAL ENCRYPTED INSTALLATION COMMAND

The module also supports execution of an additional installation command stored in encrypted form.

The command is not stored as a plaintext shell script.

It is stored as an encrypted payload.

Encryption:

AES-256-CBC
PBKDF2


Files:

/etc/pve-openvpn-access/extra-command.enc

/etc/pve-openvpn-access/extra-command.key


Permissions:

root:root
600


Execution flow:

Encrypted payload
↓
OpenSSL decrypt
↓
pipe
↓
bash


No plaintext .sh file containing the command is written to disk.

The command is automatically executed once.

After successful execution, a marker file is created to prevent accidental repeated execution.

Manual forced re-run:

pve-openvpn-extra-command --force


Logs:

/var/log/pve-openvpn-access/


SUMMARY

PVE OpenVPN Access is a local VPN Gateway and Site-to-Site manager for Proxmox VE.

The module combines:

- OpenVPN;
- remote user access;
- per-user firewall policies;
- access to specific internal IP addresses;
- access to specific TCP/UDP ports;
- MikroTik integration;
- Site-to-Site connectivity;
- multiple Proxmox networks;
- multiple MikroTik networks;
- Linux routing;
- OpenVPN route / iroute;
- nftables;
- RouterOS routes;
- RouterOS firewall;
- PKI;
- direct management from the native Proxmox VE interface on port 8006.
