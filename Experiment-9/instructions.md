# VLAN Configuration and Management – Cisco Packet Tracer Lab

## Objective
To configure and verify VLANs, trunking, and management interfaces on Cisco switches (S1, S2, and S3) to enable logical network segmentation and secure management access.

---

## Network Topology
- **Switches:** S1, S2, S3 (Cisco 2960 or equivalent)
- **PCs:** PC1–PC6 connected across S2 and S3
- **Trunks:** 
  - S1 Fa0/1 ↔ S2 Fa0/1  
  - S1 Fa0/2 ↔ S3 Fa0/2
- **VLANs Configured:**
  | VLAN ID | Name           | Purpose         |
  |----------|----------------|-----------------|
  | 10       | Faculty_Staff  | VLAN for staff  |
  | 20       | Students       | VLAN for students |
  | 30       | Guest          | VLAN for guests |
  | 99       | Management     | For switch management |

---

## Configuration Summary

### 1. Basic Switch Setup
Performed on **S1**, **S2**, and **S3**:
```bash
enable
configure terminal
hostname S1        # (S2 / S3 accordingly)
no ip domain-lookup
service password-encryption
banner motd #VLAN lab - authorized use only#
line con 0
 exec-timeout 0 0
 logging synchronous
 password cisco
 login
line vty 0 4
 password cisco
 login
enable secret class
end
```
2. Clear and Initialize Switch Ports
```
configure terminal
interface range fa0/1-24
 shutdown
interface range gi0/1-2
 shutdown
```
Re-enable only required user ports:
```
interface range fa0/6, fa0/11, fa0/18
 switchport mode access
 no shutdown
```
3. VLAN Creation (on All Switches)
```
vlan 10
 name faculty/staff
vlan 20
 name students
vlan 30
 name guest
vlan 99
 name management
end
```
Verify VLANs:
```
show vlan brief
```
4. Assign VLANs to Ports

Example for S2:
```
interface fa0/11
 switchport access vlan 10
 no shutdown
interface fa0/18
 switchport access vlan 20
 no shutdown
interface fa0/6
 switchport access vlan 30
 no shutdown
```
Example for S3: (same pattern)

5. Configure Management VLAN (VLAN 99)
```
interface vlan 99
 ip address 172.17.99.11 255.255.255.0    # S1
 no shutdown
interface vlan 99
 ip address 172.17.99.12 255.255.255.0    # S2
 no shutdown
interface vlan 99
 ip address 172.17.99.13 255.255.255.0    # S3
 no shutdown
```
6. Configure Trunking and Native VLAN
S1.
```
interface range fa0/1-2
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
```
S2.
```
interface fa0/1
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
```
S3.
```
interface fa0/2
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
```
Verify
```
show interfaces trunk
```
7. Save Configuration
```
end
copy running-config startup-config

```
Verification & Testing
| Test                               | Command                              | Expected Result                    |
| ---------------------------------- | ------------------------------------ | ---------------------------------- |
| Check VLANs                        | `show vlan brief`                    | VLANs 10, 20, 30, 99 listed        |
| Check Trunks                       | `show interfaces trunk`              | Trunk ports active, native VLAN 99 |
| Check SVI                          | `show ip interface brief`            | VLAN99 interfaces up/up            |
| Ping between switches              | `ping 172.17.99.12` / `172.17.99.13` | Success                            |
| Ping between PCs (same VLAN)       | e.g., PC2 → PC5                      | Success                            |
| Ping between PCs (different VLANs) | e.g., PC1 → PC2                      | Fail (no L3 routing)               |

PC IP Configuration
| PC  | VLAN | IP Address   | Subnet Mask   | Default Gateway |
| --- | ---- | ------------ | ------------- | --------------- |
| PC1 | 10   | 172.17.10.21 | 255.255.255.0 | 172.17.10.1     |
| PC2 | 20   | 172.17.20.22 | 255.255.255.0 | 172.17.20.1     |
| PC3 | 30   | 172.17.30.23 | 255.255.255.0 | 172.17.30.1     |
| PC4 | 10   | 172.17.10.24 | 255.255.255.0 | 172.17.10.1     |
| PC5 | 20   | 172.17.20.25 | 255.255.255.0 | 172.17.20.1     |
| PC6 | 30   | 172.17.30.26 | 255.255.255.0 | 172.17.30.1     |
