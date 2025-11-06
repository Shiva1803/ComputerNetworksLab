# Cisco Packet Tracer — DHCP Relay, NAT & OSPF Configuration (CLI Only)

This **Packet Tracer activity** was **fully configured via CLI**, demonstrating end-to-end setup of:
- **DHCP Relay** between routers  
- **NAT (PAT & Static NAT)** for internal-external communication  
- **OSPF Routing** for dynamic network connectivity  

---

## Network Topology Overview

| Device | Role | Key Interfaces | Description |
|---------|------|----------------|--------------|
| **R1** | Branch Router | G0/0, G0/1, S0/3/0 | Connects PCs via switches, relays DHCP |
| **R2** | Main Router | G0/0, S0/3/0, S0/3/1 | DHCP Server + NAT Router |
| **ISP** | Internet Router | S0/3/1 | External network |
| **PC0** | Client | Connected to Switch0 → R1 G0/0 | DHCP Client |
| **PC1** | Client | Connected to Switch1 → R1 G0/1 | DHCP Client |
| **Server** | DHCP/NAT Backend | Connected to R2 G0/0 | Internal Server |

---

## Step 1 — Setup Devices & Connections
1. Add the following devices to the workspace:
   - **3 Routers (R1, R2, ISP)**  
   - **2 Switches (Switch0, Switch1)**  
   - **2 PCs (PC0, PC1)**  
   - **1 Server**
2. Connect them using suitable cables:
   - PCs ↔ Switches ↔ R1  
   - R1 ↔ R2 (Serial)  
   - R2 ↔ ISP (Serial)  
   - Server ↔ R2 (GigabitEthernet)

---

## Step 2 — Configure **Router R1**
Paste the following commands into **R1 CLI**:

```bash
enable
configure terminal
hostname R1

! Interfaces
interface GigabitEthernet0/0
 description To-Switch0 (PC0)
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

interface GigabitEthernet0/1
 description To-Switch1 (PC1)
 ip address 192.168.11.1 255.255.255.0
 no shutdown
 exit

interface Serial0/3/0
 description To-R2
 ip address 10.1.1.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! DHCP Relay
interface GigabitEthernet0/0
 ip helper-address 192.168.20.254
 exit
interface GigabitEthernet0/1
 ip helper-address 192.168.20.254
 exit

! Routing - OSPF
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.11.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.3 area 0
 exit

end
write memory
enable
configure terminal
hostname R2

! Interfaces
interface GigabitEthernet0/0
 description To-Server
 ip address 192.168.20.254 255.255.255.0
 no shutdown
 exit

interface Serial0/3/0
 description To-R1
 ip address 10.1.1.2 255.255.255.252
 ip nat inside
 no shutdown
 exit

interface Serial0/3/1
 description To-ISP
 ip address 209.165.200.225 255.255.255.252
 ip nat outside
 clock rate 64000
 no shutdown
 exit

! DHCP Exclusions and Pools
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.11.1 192.168.11.10

ip dhcp pool LAN1
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 lease 1
 exit

ip dhcp pool LAN2
 network 192.168.11.0 255.255.255.0
 default-router 192.168.11.1
 dns-server 8.8.8.8
 lease 1
 exit

! NAT Configuration
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.11.0 0.0.0.255
ip nat inside source list 1 interface Serial0/3/1 overload

! Static NAT for Server
ip nat inside source static 192.168.20.253 209.165.200.254

! Routing - OSPF
router ospf 1
 network 10.1.1.0 0.0.0.3 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 209.165.200.224 0.0.0.3 area 0
 exit

end
write memory
```
For R2
```
enable
configure terminal
hostname R2

! Interfaces
interface GigabitEthernet0/0
 description To-Server
 ip address 192.168.20.254 255.255.255.0
 no shutdown
 exit

interface Serial0/3/0
 description To-R1
 ip address 10.1.1.2 255.255.255.252
 ip nat inside
 no shutdown
 exit

interface Serial0/3/1
 description To-ISP
 ip address 209.165.200.225 255.255.255.252
 ip nat outside
 clock rate 64000
 no shutdown
 exit

! DHCP Exclusions and Pools
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.11.1 192.168.11.10

ip dhcp pool LAN1
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 lease 1
 exit

ip dhcp pool LAN2
 network 192.168.11.0 255.255.255.0
 default-router 192.168.11.1
 dns-server 8.8.8.8
 lease 1
 exit

! NAT Configuration
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.11.0 0.0.0.255
ip nat inside source list 1 interface Serial0/3/1 overload

! Static NAT for Server
ip nat inside source static 192.168.20.253 209.165.200.254

! Routing - OSPF
router ospf 1
 network 10.1.1.0 0.0.0.3 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 209.165.200.224 0.0.0.3 area 0
 exit

end
write memory
```
For ISP (R3)
```
enable
configure terminal
hostname ISP

interface Serial0/3/1
 description To-R2
 ip address 209.165.200.226 255.255.255.252
 no shutdown
 exit

! Routing - OSPF
router ospf 1
 network 209.165.200.224 0.0.0.3 area 0
 exit

end
write memory
```
Step 5 — Verify DHCP Operation
   Step 1: Set PCs to DHCP
      
   PC0 → Desktop → IP Configuration → Select DHCP
      
   PC1 → Desktop → IP Configuration → Select DHCP
      
   R1 will forward the DHCP requests to R2, and R2 will assign IPs.
   
   Step 2: Verify DHCP Assignment
   
   On each PC → Desktop → Command Prompt → Type:
   
      
      ipconfig

      
   Expected Output:
   
   ```
   
   PC0
   IP Address .......... 192.168.10.x
   Subnet Mask ......... 255.255.255.0
   Default Gateway ..... 192.168.10.1
   
   PC1
   IP Address .......... 192.168.11.x
   Subnet Mask ......... 255.255.255.0
   Default Gateway ..... 192.168.11.1
   ```

   If both PCs receive IPs — DHCP Relay and Pools are working!
