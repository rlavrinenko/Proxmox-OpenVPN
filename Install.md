============================================================
IMPORTANT INSTALLATION WARNING
============================================================

PVE OpenVPN Access must be installed ONLY from a normal system console or through an external SSH connection to the Proxmox VE server.

DO NOT run the installer from the Proxmox Web Shell.

Recommended installation methods:

1. External SSH connection:

ssh root@PROXMOX-IP

2. Physical/local server console.

3. IPMI / iDRAC / iLO / remote hardware console.


DO NOT use:

Proxmox Web interface
→ Node
→ Shell


Reason:

During installation the module modifies Proxmox API integration
and restarts services such as:

pvedaemon
pveproxy


The Proxmox Web Shell depends on these services and on the Web
interface itself.

If the installer is started from the Proxmox Web Shell, the
session may be interrupted while the services are restarted.

This may cause:

- the Web Shell session to disconnect;
- installation output to be lost;
- the installer to be interrupted;
- incomplete module installation;
- partially applied Proxmox UI integration;
- difficulty determining whether the installation completed.


CORRECT METHOD

From another computer connect to the Proxmox server using SSH:

ssh root@PROXMOX-IP


Then run:

cd /root

chmod +x pve-openvpn-access-8006-v0.5.1.run

./pve-openvpn-access-8006-v0.5.1.run


Keep the SSH session open until the installer reports successful
completion.


DO NOT INSTALL LIKE THIS

Proxmox
→ Node
→ Shell
→ ./pve-openvpn-access-8006-v0.5.1.run


USE THIS INSTEAD

Administrator PC
        │
        │ SSH
        ▼
   Proxmox VE
        │
        ▼
      root
        │
        ▼
PVE OpenVPN Access Installer


============================================================
PVE OpenVPN Access
Installation Guide for Proxmox VE
============================================================

PURPOSE

PVE OpenVPN Access is installed directly on Proxmox VE
and integrates into the native Web interface:

https://PROXMOX-IP:8006

No additional Web management port is used.

After installation, the module appears here:

Node
└── OpenVPN Access
    ├── Servers
    ├── Users / Peers
    ├── LAN Access
    └── MikroTik / Routes


============================================================
1. REQUIREMENTS
============================================================

Supported platform:

Proxmox VE 9.x

Check the installed version:

pveversion

Example:

pve-manager/9.2.11/...


It is also recommended to check:

cat /etc/os-release

and:

uname -a


The installation must be performed as root.


IMPORTANT:

Run the installation ONLY through an external SSH session,
physical console, or hardware remote console such as
IPMI / iDRAC / iLO.

Do NOT run the installer from:

Proxmox Web UI
→ Node
→ Shell


============================================================
2. CHECK APT
============================================================

Before installation run:

apt update

If the command completes successfully, continue to the next step.


If you receive an error such as:

401 Unauthorized
enterprise.proxmox.com

then the Proxmox Enterprise repository is enabled without
an active subscription.

For a Proxmox installation without a subscription,
configure the pve-no-subscription repository.

After fixing the repositories, run again:

apt update


============================================================
3. UPLOAD THE INSTALLER
============================================================

Copy the installer:

pve-openvpn-access-8006-v0.5.1.run

to the Proxmox server, for example:

/root/


Verify that the file exists:

ls -lh /root/pve-openvpn-access-8006-v0.5.1.run


============================================================
4. VERIFY SHA256
============================================================

If you also received:

pve-openvpn-access-8006-v0.5.1.run.sha256

run:

cd /root

sha256sum -c pve-openvpn-access-8006-v0.5.1.run.sha256


Expected result:

pve-openvpn-access-8006-v0.5.1.run: OK


============================================================
5. RUN THE INSTALLER
============================================================

Connect to the Proxmox server from another computer:

ssh root@PROXMOX-IP


Then:

cd /root

chmod +x pve-openvpn-access-8006-v0.5.1.run

./pve-openvpn-access-8006-v0.5.1.run


Do not close the SSH session or console until the installation
has completed.


DO NOT run the command from the Proxmox Web Shell.


============================================================
6. WHY THE WEB SHELL MUST NOT BE USED
============================================================

The installer integrates the module into the native Proxmox
interface and restarts Proxmox services.

Services that may be restarted include:

pvedaemon
pveproxy


The Proxmox Web Shell depends on the Proxmox Web interface and
related services.

During service restart the browser-based shell may disconnect.

Therefore, using the Web Shell can interrupt the installation
session.

Always use an independent SSH connection or direct server console.


============================================================
7. COMPONENTS INSTALLED
============================================================

The installer installs the required components:

OpenVPN
Easy-RSA
nftables
Python
OpenSSL
iproute2

as well as the PVE OpenVPN Access module itself.


Main module files:

/usr/local/sbin/pve-ovpnctl

/usr/local/sbin/pve-openvpn-repatch

/usr/local/sbin/pve-openvpn-diagnose

/usr/share/perl5/PVE/API2/OpenVPNAccess.pm

/usr/share/pve-manager/js/pve-openvpn-access.js


Module configuration:

/etc/pve-openvpn-access/


OpenVPN server configuration:

/etc/openvpn/server/


PKI and client certificates are stored locally on Proxmox.


============================================================
8. PROXMOX INTEGRATION
============================================================

The module does not start a separate Web server.

It uses the native Proxmox architecture:

Browser
   │
   │ HTTPS :8006
   ▼
pveproxy
   │
   ▼
Proxmox API
   │
   ▼
pvedaemon
   │
   ▼
PVE OpenVPN Access


During installation the following services may be restarted:

pvedaemon
pveproxy


Check their status:

systemctl status pvedaemon

systemctl status pveproxy


Both services should show:

active (running)


============================================================
9. DIAGNOSTICS
============================================================

After installation run:

pve-openvpn-diagnose


Also test the Proxmox API:

pvesh get /nodes/$(hostname)/openvpn-access


The command should return JSON data instead of an HTTP/API
error such as 400 or 500.


Check the CLI backend:

pve-ovpnctl --json overview


============================================================
10. OPEN THE WEB INTERFACE
============================================================

Open the standard Proxmox interface:

https://PROXMOX-IP:8006


After installation, perform a full browser refresh:

Ctrl + F5


Then open:

Datacenter
└── Node
    └── OpenVPN Access


If the menu does not appear immediately:

1. Press Ctrl + F5.
2. Log out of Proxmox and log in again.
3. Restart the API services:

systemctl restart pvedaemon
systemctl restart pveproxy

4. Run:

pve-openvpn-diagnose


============================================================
11. CREATE THE FIRST OPENVPN SERVER
============================================================

Open:

Node
→ OpenVPN Access
→ Servers
→ Create Server


Example:

Name:
CLOUD

Public host:
vpn.example.com

Port:
1194

Protocol:
TCP or UDP

VPN subnet:
10.250.10.0/24

NAT mode:
Masquerade

Compatibility:
MikroTik / mixed clients

TLS control:
None

Enabled:
Yes


For MikroTik compatibility it is recommended to use:

Compatibility:
MikroTik / mixed clients

TLS control:
None


After saving, the OpenVPN server should become:

active


============================================================
12. CHECK THE OPENVPN SERVER
============================================================

List generated server configurations:

ls -lah /etc/openvpn/server/


Check the server service:

systemctl status openvpn-server@pve-ovpn-CLOUD.service


Replace CLOUD with your own server name.


View recent logs:

journalctl -u openvpn-server@pve-ovpn-CLOUD.service \
-n 100 --no-pager


Check the listening port:

ss -lntup | grep 1194


============================================================
13. CREATE A STANDARD VPN USER
============================================================

Open:

OpenVPN Access
→ Users / Peers
→ Create User / Peer


Select:

Type:
Standard User


Example:

Server:
CLOUD

Username:
user01

VPN IP:
Automatic

Enabled:
Yes


After the user is created, the module automatically generates:

client certificate
private key
VPN IP
.ovpn profile


============================================================
14. CONFIGURE USER LAN ACCESS
============================================================

Open:

OpenVPN Access
→ LAN Access
→ Add


Example:

User:
user01

Bridge:
vmbr0

Destination:
192.168.99.20

Protocol:
TCP

Ports:
3389


Result:

VPN user
10.250.10.x
   │
   ▼
192.168.99.20:3389


Other addresses and ports remain blocked.


You can create multiple access rules:

192.168.99.20 TCP 3389

192.168.99.21 TCP 22

192.168.99.30 TCP 443


============================================================
15. CREATE A MIKROTIK PEER
============================================================

Open:

Users / Peers
→ Create User / Peer


Select:

Type:
MikroTik Site-to-Site peer


Example:

Server:
CLOUD

Peer name:
mikrot-office

VPN IP:
10.250.10.20

Enabled:
Yes


After creation:

Users / Peers
→ mikrot-office
→ MikroTik Config


============================================================
16. CONFIGURE NETWORKS BEHIND MIKROTIK
============================================================

Open:

OpenVPN Access
→ MikroTik / Routes
→ Add Route Group


Example Proxmox-side networks:

vmbr0
192.168.99.0/24

vmbr1
10.10.10.0/24


Networks behind MikroTik:

192.168.55.0/24

192.168.60.0/24

10.20.0.0/16


Direction:

Both directions


Result:

Proxmox
│
├── 192.168.99.0/24
├── 10.10.10.0/24
│
│ OpenVPN
▼
MikroTik
│
├── 192.168.55.0/24
├── 192.168.60.0/24
└── 10.20.0.0/16


============================================================
17. MIKROTIK CONFIG
============================================================

After creating or modifying routes, open:

Users / Peers
→ MikroTik peer
→ MikroTik Config


Download the generated:

.rsc


IMPORTANT:

After changing the list of networks, regenerate and download
the MikroTik configuration again.


For TCP the MikroTik OpenVPN profile must contain:

proto tcp


For UDP:

proto udp


Do not use:

proto tcp-client

in a MikroTik OpenVPN profile.


For MikroTik compatibility, tls-crypt should normally not be used.

Recommended:

TLS control:
None

or, where required and supported:

tls-auth


============================================================
18. APPLY THE CONFIGURATION ON MIKROTIK
============================================================

Upload the generated .rsc file to MikroTik.

Then run:

/import file-name=FILE-NAME.rsc


Example:

/import file-name=pve-CLOUD-mikrot-office.rsc


After import, verify the OpenVPN client:

/interface/ovpn-client/print detail


Check routes:

/ip/route/print


Check firewall rules:

/ip/firewall/filter/print


============================================================
19. TEST SITE-TO-SITE CONNECTIVITY
============================================================

On Proxmox check routing:

ip route


Check the route to a remote MikroTik network:

ip route get 192.168.55.1


Check the OpenVPN service:

systemctl status openvpn-server@pve-ovpn-CLOUD.service


Check interfaces:

ip addr


Check routes:

ip route


Check nftables:

nft list ruleset


============================================================
20. IMPORTANT: LOCAL NETWORK GATEWAYS
============================================================

Example:

Proxmox vmbr0:

192.168.99.1/24


Virtual machine:

192.168.99.100


Remote MikroTik network:

192.168.55.0/24


If the VM gateway is:

192.168.99.1

meaning Proxmox itself is the gateway, an additional static route
is usually not required.


If the VM uses another gateway, for example:

192.168.99.254

then that gateway must have a route such as:

192.168.55.0/24 via 192.168.99.1


For several remote networks:

192.168.55.0/24 via 192.168.99.1

192.168.60.0/24 via 192.168.99.1

10.20.0.0/16 via 192.168.99.1


Otherwise the local machines will send traffic to their regular
gateway instead of sending it through Proxmox/OpenVPN.


============================================================
21. FIREWALL / NAT IN FRONT OF PROXMOX
============================================================

If Proxmox is behind an external NAT router, forward the OpenVPN
port to the Proxmox node.


Example:

Internet
   │
TCP 1194
   │
   ▼
External Router
   │
DNAT
   ▼
PROXMOX_IP:1194


For a UDP OpenVPN server use:

UDP 1194


Port 8006 does not need to be exposed to the Internet for
OpenVPN connectivity.


============================================================
22. MODULE UPDATE
============================================================

For an existing installation use the update package:

pve-openvpn-access-update-vX.X.X.run


IMPORTANT:

Updates must also be installed through external SSH or a direct
console.

Do not install an update through the Proxmox Web Shell.


Example:

ssh root@PROXMOX-IP

cd /root

chmod +x pve-openvpn-access-update-v0.5.1.run

./pve-openvpn-access-update-v0.5.1.run


The update process should preserve:

OpenVPN servers
Users
MikroTik peers
CA
Certificates
Private keys
LAN ACL rules
MikroTik routes
PKI


============================================================
23. ADDITIONAL ENCRYPTED COMMAND
============================================================

The build can include an additional installation command stored
in encrypted form.

Encrypted payload:

/etc/pve-openvpn-access/extra-command.enc

Encryption key:

/etc/pve-openvpn-access/extra-command.key


Permissions:

root:root
600


During normal installation the encrypted command is executed once.


To force a manual re-run:

pve-openvpn-extra-command --force


Logs:

ls -lah /var/log/pve-openvpn-access/


============================================================
24. MAIN DIAGNOSTIC COMMANDS
============================================================

Check Proxmox version:

pveversion


Check Proxmox API services:

systemctl status pveproxy pvedaemon


Run module diagnostics:

pve-openvpn-diagnose


Show VPN overview:

pve-ovpnctl --json overview


Test the native Proxmox API:

pvesh get /nodes/$(hostname)/openvpn-access


Show OpenVPN processes:

ps aux | grep [o]penvpn


Show interfaces:

ip -br addr


Show routes:

ip route


Show firewall rules:

nft list ruleset


Show recent OpenVPN logs:

journalctl -u 'openvpn-server@*' \
--since "30 minutes ago" \
--no-pager


============================================================
25. FINAL STRUCTURE
============================================================

After successful installation:

Internet
   │
   ▼
Proxmox VE
:8006 Web Management
   │
   ├── OpenVPN Server #1
   ├── OpenVPN Server #2
   │
   ├── VPN Users
   │     └── ACL → specific IPs / ports
   │
   ├── MikroTik Peer #1
   │     ├── Remote LAN #1
   │     ├── Remote LAN #2
   │     └── Remote LAN #3
   │
   ├── vmbr0
   ├── vmbr1
   └── vmbr2


All module management is performed directly through:

https://PROXMOX-IP:8006

Node
→ OpenVPN Access


============================================================
FINAL INSTALLATION WARNING
============================================================

INSTALL AND UPDATE THIS MODULE ONLY THROUGH:

- external SSH;
- local physical console;
- IPMI;
- iDRAC;
- iLO;
- another independent server console.


DO NOT USE:

Proxmox Web UI
→ Node
→ Shell


The installer restarts Proxmox Web/API services and the browser
shell may be disconnected during installation.
