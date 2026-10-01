# Enterprise Firewall & DMZ Implementation
## Project Overview

This project implements and evaluates two firewall solutions using **VyOS and pfSense** to protect private networks and isolate public-facing services within DMZ networks.

The environment includes public, DMZ, and private network segments. Firewall policies and NAT were configured to control communication between these networks while allowing required HTTP, FTP, and TFTP services.

The project also involved testing connectivity and troubleshooting firewall and NAT issues to ensure that permitted services were accessible while unauthorized traffic remained restricted.

## Network Topology
The environment contains two separate firewall implementations:

- **pfSense** protecting the Private A and pfSense DMZ networks
- **VyOS** protecting the Private B and VyOS DMZ networks
- **Windows 11** workstation representing a host on the public network
- **AlmaLinux** servers providing services within the DMZs
- **Windows Server** systems representing hosts within the private networks

### Network Segments

| Network | Subnet | Purpose |
| Public | 44.104.11.0/16 | Simulated public network |
| pfSense DMZ | 192.168.1.0/24 | Public-facing services behind pfSense |
| Private A | 192.168.3.0/24 | Private network protected by pfSense |
| VyOS DMZ | 172.18.11.0/24 | Public-facing services behind VyOS |
| Private B | 192.168.2.0/24 | Private network protected by VyOS |

### Topology
The netowrk topology is showed in the topology folder

## Firewall Architecture
### VyOS

VyOS was configured as a zone-based firewall separating the **WAN, DMZ, and Private B** networks.

Firewall policies were created to control communication between the zones. A default-deny approach was used so that traffic was blocked unless explicitly permitted by a firewall rule.

The VyOS environment included rules for:

- WAN-to-DMZ communication
- DMZ-to-WAN communication
- Private-to-DMZ communication
- Private-to-WAN communication
- Established and related connections

### pfSense

pfSense was configured with separate **WAN, DMZ, and Private A** interfaces.

Interface firewall rules were used to restrict communication between network segments while allowing required services. NAT was also configured to provide access between public and private addressing.

## NAT Configuration

NAT was used to make internal services accessible from the public network and provide outbound connectivity for protected systems.
The configuration included:

- Destination/static NAT for public-facing DMZ services
- Source NAT/masquerading for private network Internet access
- pfSense 1:1 NAT for the DMZ server
- VyOS DNAT for required DMZ services
- Outbound NAT for protected networks

For example, the pfSense DMZ server used the internal address `192.168.1.10` with a public virtual address of `44.104.11.6`.

## DMZ Services

AlmaLinux servers were configured to provide services for firewall testing.

### HTTP

Apache HTTP Server was configured to provide HTTP service over **TCP port 80**.

Connectivity was tested from the public Windows 11 workstation to verify that the firewall and NAT configuration permitted HTTP traffic to reach the DMZ server.

### FTP

vsftpd was configured to provide FTP service.

FTP required:
- TCP 21 for the FTP control connection
- TCP 30000–30050 for passive FTP data connections

Passive FTP was used because the control connection and data connection operate separately, requiring the firewall to permit the configured passive data-port range.

### TFTP

TFTP was configured to provide lightweight file transfer functionality over UDP.
Testing confirmed that a file could be retrieved successfully through the configured environment.

## Testing and Validation

The environment was tested to verify that firewall and NAT policies operated as expected.

Testing included:
- HTTP connectivity to the DMZ web server
- FTP authentication and directory listing
- Passive FTP data connections
- TFTP file transfers
- Private network outbound connectivity
- Firewall rule validation
- NAT translation validation
- Verification that unauthorized traffic was denied

## Troubleshooting
### Passive FTP Data Connection

During testing, the FTP control connection on TCP port 21 successfully reached the server and authentication succeeded. However, directory listings and data transfers initially failed.

Investigation showed that FTP was attempting to establish a separate passive data connection. A passive port range of **30000–30050** was configured on the vsftpd server, and the corresponding firewall and NAT rules were added.

After the changes, both the FTP control connection and passive data connection completed successfully.

### TFTP Return Traffic

TFTP requests successfully reached the DMZ server, but return traffic initially failed.

Packet inspection showed that TFTP begins communication on UDP port 69 but uses dynamically selected UDP ports during the transfer. The firewall configuration was adjusted to correctly handle the return traffic, allowing the file transfer to complete.

### ICMP Connectivity

Private network systems were able to access web services while ICMP testing initially failed because ICMP traffic was not explicitly permitted.
An ICMP firewall rule was added where required, allowing connectivity tests to complete without broadly permitting unnecessary inbound traffic.

## Security Design

Several security principles were applied throughout the project:

- Separation of public, DMZ, and private networks
- Default-deny firewall policies
- Explicit service-based firewall rules
- DMZ placement for externally accessible services
- NAT to separate public and private addressing
- Stateful handling of established and related connections
- Limited exposure of required service ports

These controls allow required services to operate while reducing unnecessary communication between network segments.

## Technologies Used

- VyOS
- pfSense
- AlmaLinux
- Windows Server
- Windows 11
- Apache HTTP Server
- vsftpd
- TFTP
- TCP/IP
- NAT / PAT
- Zone-based firewall policies
- `curl`
- `tcpdump`

## Key Takeaways

This project provided hands-on experience designing and troubleshooting a segmented firewall environment rather than only creating firewall rules theoretically.

The implementation demonstrated how firewall policies, NAT, DMZ segmentation, stateful connections, and application protocols interact. Troubleshooting FTP and TFTP also demonstrated the importance of understanding the difference between an application's initial connection and its subsequent data traffic.


> **Security Note:** Configuration files and screenshots in this repository are sanitized. Passwords, password hashes, private keys, certificates, and other sensitive configuration data are excluded.