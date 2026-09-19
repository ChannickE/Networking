# Network Documentation

## 1. Business Scenario

A two-floor restaurant requires a network for ten wired POS terminals, wireless POS tablets, six kitchen/service-bar printers, two manager workstations, three internal servers, network-management access, and simulated Internet connectivity.

The design prioritizes segmentation, centralized services, simple operations, and CCNA-level security controls.

## 2. Physical Design

### Core
- CORE-SW: Cisco 3560 multilayer switch
- EDGE-RTR: Cisco 2911 edge router
- Three servers attached to CORE-SW VLAN 50
- ISP-RTR and INTERNET-SERVER simulate an external network

### Floor 1
- FLOOR1-SW: Cisco 2960
- Fa0/1–5: wired POS, VLAN 10
- Fa0/6–7: manager PCs, VLAN 30
- Fa0/8–11: printers, VLAN 20
- Gi0/1: trunk to CORE-SW
- Gi0/2: AP-PT access port, VLAN 40

### Floor 2
- FLOOR2-SW: Cisco 2960
- Fa0/1–5: wired POS, VLAN 10
- Fa0/6–7: printers, VLAN 20
- Gi0/1: trunk to CORE-SW
- Gi0/2: AP-PT access port, VLAN 40

## 3. Logical Design

CORE-SW owns SVIs for VLANs 10, 20, 30, 40, 50, and 99 and performs inter-VLAN routing. Its default route points to EDGE-RTR at 10.10.254.2. EDGE-RTR has static routes back to internal VLANs and performs PAT using its 203.0.113.2 outside interface.

DHCP/DNS is centralized at 10.10.50.20. Client VLAN SVIs use DHCP relay. Servers and printers use static addressing; user/POS endpoints use DHCP.

## 4. Wireless Design

Two AP-PT autonomous access points use:
- SSID: Restaurant-POS
- WPA2-PSK / AES
- PSK: RestaurantPOS123! (lab credential; replace before publishing screenshots/configs if desired)
- Floor 1 channel: 6
- Floor 2 channel: 11

The AP-PT model used in this lab acts as a Layer 2 bridge and does not expose an IP-management configuration. Each AP switch port is therefore an access port in VLAN 40 rather than an 802.1Q trunk.

## 5. Server Services

### POS/Application — 10.10.50.10
HTTP hosts a simple restaurant POS status page.

### DHCP/DNS — 10.10.50.20
DHCP pools:
- POS: 10.10.10.100+, gateway 10.10.10.1
- MANAGEMENT: 10.10.30.100+, gateway 10.10.30.1
- WIRELESS-POS: 10.10.40.100+, gateway 10.10.40.1

DNS records:
- pos.restaurant.local → 10.10.50.10
- dhcp.restaurant.local → 10.10.50.20
- backup.restaurant.local → 10.10.50.30

### File/Backup — 10.10.50.30
Used for FTP lab services plus centralized Syslog and NTP where supported by Packet Tracer.

## 6. Routing

CORE-SW directly routes all restaurant VLANs and uses:
`0.0.0.0/0 via 10.10.254.2`

EDGE-RTR uses:
`0.0.0.0/0 via 203.0.113.1`

EDGE-RTR also contains static routes for restaurant subnets via 10.10.254.1.

## 7. NAT/PAT

EDGE-RTR marks Gi0/0 as NAT inside and Gi0/1 as NAT outside. ACL 1 identifies 10.10.0.0/16 and overloads traffic on Gi0/1.

## 8. Security Controls

- Extended ACLs applied inbound near POS/Wireless-POS source VLANs
- POS networks denied access to VLAN 30 and VLAN 99
- SSH-only VTY access
- VTY access-class permits management source network
- Sticky port security on single-endpoint access ports
- BPDU Guard + PortFast on end-device ports
- Unused ports assigned to VLAN 999 and shut down
- AP ports do not use maximum-one-MAC port security because multiple wireless clients may be learned behind the AP

## 9. Monitoring

Network devices send informational Syslog messages to 10.10.50.30 and use that server for NTP where supported. Log timestamps are enabled.

## 10. Design Decisions

**Why VLANs?** To reduce broadcast domains and separate systems by role.

**Why a multilayer core?** To provide efficient centralized inter-VLAN routing without router-on-a-stick.

**Why DHCP relay?** Client broadcasts cannot cross VLAN boundaries, while the centralized DHCP server resides in VLAN 50.

**Why static printer/server addresses?** Infrastructure and service endpoints should have predictable addresses.

**Why PAT?** Many private internal endpoints can share the edge router's simulated public address.

**Why ACLs?** VLAN separation alone does not prevent routed communication; ACLs enforce policy between segments.

## 11. Production Considerations

This lab intentionally focuses on CCNA concepts. A real restaurant/payment network would require stronger identity/AAA, firewalling, PCI DSS controls, resilient switching/routing, secure secret handling, backups, monitoring/alerting, patching, and vendor-specific POS requirements.
