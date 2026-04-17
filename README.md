## Initial Device Access & Security
R1> enable                                    ! Enter privileged EXEC mode
R1# configure terminal                           ! Enter global configuration mode
R1(config)# hostname R1                          ! Set device hostname
R1(config)# ip domain-name cisco.com             ! Set DNS domain name
R1(config)# crypto key generate rsa              ! Generate RSA keys for SSH (e.g., 2048)
R1(config)# username admin privilege 15 secret pass ! Create local admin user
R1(config)# enable secret class                  ! Set encrypted privileged mode password
R1(config)# line con 0                           ! Enter console line configuration
R1(config-line)# password cisco                  ! Set console password
R1(config-line)# login                           ! Require login on console
R1(config-line)# logging synchronous             ! Prevent log messages from interrupting typing
R1(config-line)# exit                            ! Return to global config mode
R1(config)# line vty 0 4                         ! Enter VTY (SSH/Telnet) line configuration
R1(config-line)# transport input ssh             ! Restrict access to SSH only
R1(config-line)# login local                     ! Use local username database for login
R1(config-line)# exit                            ! Return to global config mode
R1(config)# service password-encryption          ! Encrypt plaintext passwords in config
R1(config)# banner motd # UNAUTHORIZED ACCESS #  ! Set login warning banner

## Basic Interface Configuration
R1(config)# interface GigabitEthernet0/0         ! Enter interface configuration mode
R1(config-if)# description Link to Core          ! Add a description to the interface
R1(config-if)# ip address 192.168.1.1 255.255.255.0 ! Assign IPv4 address and subnet mask
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64   ! Assign IPv6 global unicast address
R1(config-if)# ipv6 address fe80::1 link-local   ! Assign IPv6 link-local address
R1(config-if)# no shutdown                       ! Administratively enable the interface
R1(config-if)# exit                              ! Return to global config mode

## Creating VLANs
SW1(config)# vlan 10                             ! Create VLAN 10 and enter VLAN config mode
SW1(config-vlan)# name Sales                     ! Assign a name to VLAN 10
SW1(config-vlan)# vlan 20                        ! Create VLAN 20
SW1(config-vlan)# name Engineering               ! Assign a name to VLAN 20
SW1(config-vlan)# vlan 99                        ! Create Management VLAN 99
SW1(config-vlan)# name Management                ! Assign a name to VLAN 99
SW1(config-vlan)# exit                           ! Return to global config mode

## Assigning Ports to VLANs
SW1(config)# interface FastEthernet0/1           ! Enter interface config mode for port 1
SW1(config-if)# switchport mode access           ! Hardcode port as an access port
SW1(config-if)# switchport access vlan 10        ! Assign port to VLAN 10
SW1(config-if)# exit                             ! Return to global config mode
SW1(config)# interface range FastEthernet0/2 - 10 ! Select multiple interfaces
SW1(config-if-range)# switchport mode access     ! Hardcode ports as access ports
SW1(config-if-range)# switchport access vlan 20  ! Assign ports to VLAN 20
SW1(config-if-range)# exit                       ! Return to global config mode

## Configuring 802.1Q Trunking
SW1(config)# interface GigabitEthernet0/1        ! Select the uplink interface
SW1(config-if)# switchport trunk encapsulation dot1q ! Set encapsulation (required on older switches)
SW1(config-if)# switchport mode trunk            ! Hardcode port as a trunk
SW1(config-if)# switchport trunk native vlan 99  ! Set the native (untagged) VLAN to 99
SW1(config-if)# switchport trunk allowed vlan 10,20,99 ! Restrict allowed VLANs on the trunk
SW1(config-if)# exit                             ! Return to global config mode

## EtherChannel (Port-Channel) Configuration
SW1(config)# interface range FastEthernet0/1 - 2 ! Select ports to bundle
SW1(config-if-range)# channel-protocol lacp      ! Set negotiation protocol to LACP
SW1(config-if-range)# channel-group 1 mode active ! Bundle into Port-Channel 1 and actively negotiate
SW1(config-if-range)# exit                       ! Return to global config mode
SW1(config)# interface port-channel 1            ! Enter the logical Port-Channel interface
SW1(config-if)# switchport mode trunk            ! Make the bundled link a trunk
SW1(config-if)# switchport trunk allowed vlan 10,20,99 ! Allow specific VLANs
SW1(config-if-range)# exit                       ! Return to global config mode

## Spanning Tree Protocol (STP)
SW1(config)# spanning-tree mode rapid-pvst       ! Enable Rapid PVST+ (default on modern Catalyst)
SW1(config)# spanning-tree vlan 10,20 root primary ! Make switch the Primary Root Bridge for VLANs 10, 20
SW1(config)# spanning-tree vlan 99 root secondary  ! Make switch the Backup Root Bridge for VLAN 99
SW1(config)# interface FastEthernet0/10          ! Select a port connected to an end device (PC)
SW1(config-if)# spanning-tree portfast           ! Bypass STP listening/learning states
SW1(config-if)# spanning-tree bpduguard enable   ! Disable port if a BPDU is unexpectedly received
SW1(config-if)# exit                             ! Return to global config mode

## Inter-VLAN Routing (Router-on-a-Stick)
R1(config)# interface GigabitEthernet0/1         ! Enter the physical interface connected to the switch
R1(config-if)# no ip address                     ! Ensure no IP is assigned to the physical interface
R1(config-if)# no shutdown                       ! Turn on the physical interface
R1(config-if)# exit                              ! Return to global config mode
R1(config)# interface GigabitEthernet0/1.10      ! Create subinterface for VLAN 10
R1(config-subif)# encapsulation dot1Q 10         ! Tag traffic for VLAN 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0 ! Set gateway IP for VLAN 10
R1(config-subif)# exit                           ! Return to global config mode
R1(config)# interface GigabitEthernet0/1.99      ! Create subinterface for Native VLAN 99
R1(config-subif)# encapsulation dot1Q 99 native  ! Tag traffic for VLAN 99 and specify it as native
R1(config-subif)# ip address 192.168.99.1 255.255.255.0 ! Set gateway IP for VLAN 99
R1(config-subif)# exit                           ! Return to global config mode

## Switch Virtual Interfaces (SVI) for Layer 3 Switching
SW1(config)# ip routing                          ! Enable routing capabilities on a Layer 3 switch
SW1(config)# interface vlan 10                   ! Create the SVI for VLAN 10
SW1(config-if)# ip address 192.168.10.1 255.255.255.0 ! Set gateway IP for VLAN 10
SW1(config-if)# no shutdown                      ! Enable the SVI
SW1(config-if)# exit                             ! Return to global config mode

## Static Routing
R1(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.226 ! Configure IPv4 Default Route
R1(config)# ipv6 route ::/0 2001:db8:acad:2::2   ! Configure IPv6 Default Route
R1(config)# ip route 10.1.1.0 255.255.255.0 192.168.2.2 ! Configure Standard Static Route
R1(config)# ip route 10.1.1.0 255.255.255.0 192.168.3.2 10 ! Configure Floating Static Route (Backup, AD 10)

## Dynamic Routing (OSPFv2 - Single Area)
R1(config)# router ospf 1                        ! Start OSPF process 1
R1(config-router)# router-id 1.1.1.1             ! Set OSPF Router ID manually
R1(config-router)# network 192.168.10.0 0.0.0.255 area 0 ! Advertise network into Area 0 (Wildcard mask)
R1(config-router)# network 192.168.20.0 0.0.0.255 area 0 ! Advertise network into Area 0
R1(config-router)# passive-interface default     ! Stop OSPF hellos out of all interfaces by default
R1(config-router)# no passive-interface GigabitEthernet0/0 ! Allow hellos out of the uplink interface
R1(config-router)# default-information originate ! Advertise the default route to OSPF neighbors
R1(config-router)# exit                          ! Return to global config mode

## DHCP Server Configuration
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10 ! Exclude static IPs (like gateways)
R1(config)# ip dhcp pool VLAN10_POOL             ! Create and name the DHCP pool
R1(dhcp-config)# network 192.168.10.0 255.255.255.0 ! Define the subnet to lease IPs from
R1(dhcp-config)# default-router 192.168.10.1     ! Provide the default gateway to clients
R1(dhcp-config)# dns-server 8.8.8.8              ! Provide DNS server to clients
R1(dhcp-config)# domain-name cisco.com           ! Provide domain name to clients
R1(dhcp-config)# exit                            ! Return to global config mode

## Network Address Translation (NAT)
R1(config)# ip nat inside source static 192.168.10.50 209.165.200.228 ! Static NAT (1-to-1) mapping
R1(config)# access-list 1 permit 192.168.10.0 0.0.0.255 ! Define internal network eligible for NAT
R1(config)# ip nat inside source list 1 interface GigabitEthernet0/1 overload ! Configure PAT (NAT Overload)
R1(config)# interface GigabitEthernet0/0         ! Enter internal-facing interface
R1(config-if)# ip nat inside                     ! Designate as NAT inside interface
R1(config-if)# exit                              ! Return to global config mode
R1(config)# interface GigabitEthernet0/1         ! Enter external-facing interface
R1(config-if)# ip nat outside                    ! Designate as NAT outside interface
R1(config-if)# exit                              ! Return to global config mode

## First Hop Redundancy Protocol (HSRP)
R1(config)# interface GigabitEthernet0/1.10      ! Enter interface serving the LAN
R1(config-subif)# standby version 2              ! Enable HSRP version 2
R1(config-subif)# standby 10 ip 192.168.10.254   ! Set the shared Virtual IP (VIP) for group 10
R1(config-subif)# standby 10 priority 110        ! Set priority higher than default (100) to become Active
R1(config-subif)# standby 10 preempt             ! Allow router to reclaim Active status if it recovers
R1(config-subif)# exit                           ! Return to global config mode

## Creating Standard Access Control Lists (ACLs)
R1(config)# access-list 10 remark BLOCK_HOST     ! Add description to the ACL
R1(config)# access-list 10 deny host 192.168.10.50 ! Block a specific IP address
R1(config)# access-list 10 permit 192.168.10.0 0.0.0.255 ! Allow the rest of the subnet
R1(config)# access-list 10 permit any            ! Override implicit deny at the end
R1(config)# interface GigabitEthernet0/1         ! Enter interface (Apply standard ACLs close to destination)
R1(config-if)# ip access-group 10 out            ! Apply ACL outbound on the interface
R1(config-if)# exit                              ! Return to global config mode

## Creating Extended Access Control Lists (ACLs)
R1(config)# ip access-list extended BLOCK_HTTP   ! Create named extended ACL
R1(config-ext-nacl)# remark Deny web traffic     ! Add description
R1(config-ext-nacl)# deny tcp 192.168.10.0 0.0.0.255 host 10.1.1.50 eq 80 ! Block port 80 to host
R1(config-ext-nacl)# deny tcp 192.168.10.0 0.0.0.255 host 10.1.1.50 eq 443 ! Block port 443 to host
R1(config-ext-nacl)# permit ip any any           ! Allow all other traffic
R1(config-ext-nacl)# exit                        ! Return to global config mode
R1(config)# interface GigabitEthernet0/0         ! Enter interface (Apply extended ACLs close to source)
R1(config-if)# ip access-group BLOCK_HTTP in     ! Apply ACL inbound on the interface
R1(config-if)# exit                              ! Return to global config mode

## Layer 2 Security (Port Security & DHCP Snooping)
SW1(config)# interface FastEthernet0/1           ! Enter edge port
SW1(config-if)# switchport mode access           ! Hardcode access mode (Required for port security)
SW1(config-if)# switchport port-security         ! Enable port security feature
SW1(config-if)# switchport port-security maximum 2 ! Allow max 2 MAC addresses
SW1(config-if)# switchport port-security mac-address sticky ! Dynamically learn and save MAC addresses
SW1(config-if)# switchport port-security violation restrict ! Drop bad frames & log without shutting down port
SW1(config-if)# exit                             ! Return to global config mode
SW1(config)# ip dhcp snooping                    ! Enable DHCP Snooping globally
SW1(config)# ip dhcp snooping vlan 10,20         ! Enable DHCP Snooping on specific VLANs
SW1(config)# interface GigabitEthernet0/1        ! Enter uplink port facing the trusted DHCP server
SW1(config-if)# ip dhcp snooping trust           ! Trust DHCP offers/acks on this port
SW1(config-if)# exit                             ! Return to global config mode

## Device Management & Saving Configurations
R1(config)# ntp server 192.168.1.100             ! Sync time with external NTP server
R1(config)# ntp master 3                         ! Act as an NTP server for others (Stratum 3)
R1(config)# no cdp run                           ! Disable Cisco Discovery Protocol globally
R1(config)# lldp run                             ! Enable vendor-neutral LLDP globally
R1(config)# end                                  ! Exit directly to privileged EXEC mode from anywhere
R1# copy running-config startup-config           ! Save active config to NVRAM
