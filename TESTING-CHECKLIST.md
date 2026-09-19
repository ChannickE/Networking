# Testing Checklist

Use this checklist before publishing the project. Capture screenshots of the most important successful tests for the GitHub `images/` folder.

## Layer 1 / Interfaces
- [ ] CORE-SW routed and trunk uplinks are up/up
- [ ] FLOOR1-SW Gi0/1 trunk is up
- [ ] FLOOR2-SW Gi0/1 trunk is up
- [ ] AP access ports are connected
- [ ] Required endpoint ports are connected
- [ ] Unused ports are shutdown

Commands:
```text
show ip interface brief
show interfaces status
```

## VLANs / Trunks
- [ ] VLANs 10,20,30,40,50,99 exist
- [ ] Floor endpoint ports are in the correct VLAN
- [ ] VLAN 999 contains disabled unused ports
- [ ] Core-to-floor trunks carry 10,20,30,40,50,99

Commands:
```text
show vlan brief
show interfaces trunk
```

## DHCP
- [ ] Wired POS receives 10.10.10.x/24
- [ ] Manager PC receives 10.10.30.x/24
- [ ] Wireless POS receives 10.10.40.x/24
- [ ] Correct gateway and DNS are received
- [ ] VLAN 10/30/40 SVIs contain `ip helper-address 10.10.50.20`

## Routing
- [ ] CORE-SW contains connected routes for all VLANs
- [ ] CORE-SW default route points to 10.10.254.2
- [ ] EDGE-RTR default route points to 203.0.113.1
- [ ] EDGE-RTR has routes back to restaurant networks

Command:
```text
show ip route
```

## Internal Connectivity
From a wired POS:
- [ ] Ping 10.10.10.1
- [ ] Ping 10.10.50.10
- [ ] Ping allowed printer address

From a wireless POS:
- [ ] Ping 10.10.40.1
- [ ] Ping 10.10.50.10
- [ ] Ping allowed printer address

From a manager PC:
- [ ] Ping required internal servers
- [ ] SSH to permitted network devices

## DNS / Application
- [ ] `ping pos.restaurant.local` resolves to 10.10.50.10
- [ ] `http://pos.restaurant.local` opens the POS status page

## Internet / NAT
- [ ] Internal client can ping 8.8.8.10
- [ ] EDGE-RTR shows active translations after traffic generation

Commands:
```text
show ip nat translations
show ip nat statistics
```

## ACL Security
- [ ] POS can reach required POS/DNS/printer services
- [ ] Wireless POS can reach required POS/DNS/printer services
- [ ] POS cannot reach manager VLAN
- [ ] POS cannot reach VLAN 99
- [ ] Wireless POS cannot reach manager VLAN
- [ ] Wireless POS cannot reach VLAN 99
- [ ] ACL counters increment during tests

Commands:
```text
show access-lists
show ip interface vlan 10
show ip interface vlan 40
```

## Layer 2 Security
- [ ] Port security enabled on designated POS/manager/printer ports
- [ ] Sticky MAC addresses learned
- [ ] Maximum secure MAC = 1 on single-endpoint ports
- [ ] BPDU Guard enabled on end-device access ports
- [ ] BPDU Guard is NOT applied to normal switch-to-switch trunks
- [ ] AP ports are not limited to one secure MAC

Commands:
```text
show port-security
show port-security address
show spanning-tree summary
```

## Monitoring
- [ ] NTP configured where supported
- [ ] Syslog configured to 10.10.50.30
- [ ] Test interface event appears on Syslog server
- [ ] Log timestamps enabled

## Recommended Portfolio Screenshots
- [ ] Full Packet Tracer topology
- [ ] `show vlan brief`
- [ ] `show interfaces trunk`
- [ ] `show ip route`
- [ ] DHCP-assigned POS address
- [ ] Successful POS-server/DNS test
- [ ] Successful Internet test
- [ ] `show ip nat translations`
- [ ] ACL blocked/allowed test plus `show access-lists`
- [ ] Port-security output
- [ ] Syslog server event
