# PN configuration
## Router Porto

```
conf t
ip cef
!
!
mpls traffic-eng tunnels
!
interface Loopback0
 ip address 10.0.1.236 255.255.255.255
 ip ospf 1 area 0
!
interface Tunnel1
 ip unnumbered Loopback0
 tunnel mode mpls traffic-eng
 tunnel destination 10.0.1.237
 tunnel mpls traffic-eng priority 7 7
 tunnel mpls traffic-eng bandwidth 150
 tunnel mpls traffic-eng path-option 1 explicit name path1
!
interface FastEthernet0/0
 ip address 10.0.1.205 255.255.255.252
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
 mpls traffic-eng tunnels
 ip rsvp bandwidth 5000 5000
!
interface FastEthernet0/1
 ip address 10.0.0.65 255.255.255.192
 ip policy route-map L101
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
 mpls traffic-eng tunnels
!
interface FastEthernet1/0
 ip address 10.0.0.1 255.255.255.192
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
!
interface FastEthernet1/1
 ip address 10.0.1.201 255.255.255.252
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
!
router ospf 1
 router-id 1.1.1.1
 mpls traffic-eng router-id Loopback0
 mpls traffic-eng area 0
!
!
ip explicit-path name path1 enable
 next-address 10.0.1.206
 next-address 10.0.1.194
!
ip access-list extended L101
 permit udp any eq 8472 any eq 8472
!
!
route-map VXLAN-UDP permit 10
 match ip address L101
 set interface Tunnel1
!
!
!
end
```


## Router Aveiro
```
conf t
mpls ip
ip cef

router ospf 1
router-id 1.1.1.2

int f1/1 ! connection to Porto
ip address 10.0.1.202 255.255.255.252
mpls ip
no shut
ip ospf 1 area 0 ! enable ospf inside PN

int f1/0 ! connection to Coimbra
ip address 10.0.1.198 255.255.255.252
!ip rsvp bandwidth 5000 5000
mpls ip
no shut
ip ospf 1 area 0

int f0/0 ! connection to DC.A1
ip address 10.0.1.129 255.255.255.192
ip ospf 1 area 0
mpls ip
no shut

int f0/1 ! connection to DC.A2
ip address 10.0.0.194 255.255.255.192
mpls ip
ip ospf 1 area 0
no shut


```



## Router Coimbra
```
conf t
ip cef
!
!
mpls traffic-eng tunnels
!
!
!
!
interface Loopback0
 ip address 10.0.1.253 255.255.255.255
 ip ospf 1 area 0
!
interface FastEthernet0/0
 ip address 10.0.1.206 255.255.255.252
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
 mpls traffic-eng tunnels
 ip rsvp bandwidth 5000 5000
!
interface FastEthernet0/1
 ip address 10.0.1.193 255.255.255.252
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
 mpls traffic-eng tunnels
 ip rsvp bandwidth 5000 5000
!
interface FastEthernet1/0
 ip address 10.0.1.197 255.255.255.252
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
 mpls traffic-eng tunnels
 ip rsvp bandwidth 5000 5000
!
interface FastEthernet1/1
 ip address 10.0.1.1 255.255.255.192
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip

router ospf 1
 router-id 1.1.1.3
 mpls traffic-eng router-id Loopback0
 mpls traffic-eng area 0
!
!
end

```

## Router Lisboa
```
ip cef
!
!
!
!
!
!
no ip domain lookup
no ipv6 cef
!
!
mpls traffic-eng tunnels
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
ip tcp synwait-time 5
!
!
!
!
!
!
interface Loopback0
 ip address 10.0.1.237 255.255.255.255
 ip ospf 1 area 0
!
interface Tunnel1
 ip unnumbered Loopback0
 tunnel mode mpls traffic-eng
 tunnel destination 10.0.1.236
 tunnel mpls traffic-eng priority 7 7
 tunnel mpls traffic-eng bandwidth 5000
 tunnel mpls traffic-eng path-option 1 explicit name path1
!
interface FastEthernet0/0
 ip address 10.0.1.65 255.255.255.192
 ip policy route-map VXLAN-UDP
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
!
interface FastEthernet0/1
 ip address 10.0.1.194 255.255.255.252
 ip ospf 1 area 0
 speed auto
 duplex auto
 mpls ip
 mpls traffic-eng tunnels
 ip rsvp bandwidth 5000 5000
!
!
router ospf 1
 router-id 1.1.1.4
 mpls traffic-eng router-id Loopback0
 mpls traffic-eng area 0
!
ip explicit-path name path1 enable
 next-address 10.0.1.193
 next-address 10.0.1.205
!
ip access-list extended L101
!
!
route-map VXLAN-UDP permit 10
 match ip address L101
 set interface Tunnel1
!
!
!
```


# Lisboa DC
## VyOS DC.L1

```
sudo cp /opt/vyatta/etc/config.boot.default /config/config.boot
reboot
configure

set interfaces bridge br120 member interface eth2.2
set interfaces bridge br120 member interface eth2.20
set interfaces bridge br120 member interface vxlan102

set interfaces bridge br130 member interface eth2.3
set interfaces bridge br130 member interface eth2.30
set interfaces bridge br130 member interface vxlan103

set interfaces dummy dum0 address '10.0.1.208/32'
set interfaces ethernet eth0 address '10.0.1.66/26'

set interfaces ethernet eth2 vif 2
set interfaces ethernet eth2 vif 3
set interfaces ethernet eth2 vif 20
set interfaces ethernet eth2 vif 30

set interfaces vxlan vxlan120 mtu '1500'
set interfaces vxlan vxlan120 remote '10.0.0.66'
set interfaces vxlan vxlan120 vni '120'

set interfaces vxlan vxlan130 mtu '1500'
set interfaces vxlan vxlan130 remote '10.0.0.66'
set interfaces vxlan vxlan130 vni '130'

set protocols ospf area 0 network '10.0.1.64/26'
set protocols ospf area 0 network '10.0.1.208/32'

set protocols bgp system-as 43100
set protocols bgp address-family l2vpn-evpn advertise-all-vni
set protocols bgp parameters router-id 10.0.1.208
set protocols bgp neighbor 10.0.1.211 peer-group evpn
set protocols bgp neighbor 10.0.1.212 peer-group evpn
set protocols bgp peer-group evpn update-source dum0
set protocols bgp peer-group evpn remote-as 102
set protocols bgp peer-group evpn address-family l2vpn-evpn nexthop-self
set protocols bgp peer-group evpn address-family l2vpn-evpn route-reflector-client


set system host-name DCL1

set interfaces vxlan vxlan101 source-address 10.0.1.208
set interfaces vxlan vxlan101 vni 101
set interfaces vxlan vxlan101 mtu 1500
s
set interfaces bridge br101 address 10.2.1.1/22
set interfaces bridge br101 description 'client x1'
set interfaces bridge br101 member interface eth1
set interfaces bridge br101 member interface vxlan101



commit
save


```

# Porto DC
## VyOS DC.P2

```
sudo cp /opt/vyatta/etc/config.boot.default /config/config.boot
reboot

configure
set interfaces bridge br120 member interface eth2.2
set interfaces bridge br120 member interface eth2.20
set interfaces bridge br120 member interface vxlan102
set interfaces bridge br130 member interface eth2.3
set interfaces bridge br130 member interface eth2.30
set interfaces bridge br130 member interface vxlan103
set interfaces dummy dum0 address '10.0.1.211/32'
set interfaces ethernet eth0 address '10.0.0.66/26'
set interfaces ethernet eth2 vif 2
set interfaces ethernet eth2 vif 3
set interfaces ethernet eth2 vif 20
set interfaces ethernet eth2 vif 30
set interfaces loopback lo
set interfaces vxlan vxlan120 mtu '1500'
set interfaces vxlan vxlan120 remote '10.0.1.66'
set interfaces vxlan vxlan120 vni '120'
set interfaces vxlan vxlan130 mtu '1500'
set interfaces vxlan vxlan130 remote '10.0.1.66'
set interfaces vxlan vxlan130 vni '130'
set protocols ospf area 0 network '10.0.0.64/26'
set protocols ospf area 0 network '10.0.1.211/32'
set system host-name DCP2

set protocols bgp system-as 43100
set protocols bgp address-family l2vpn-evpn advertise-all-vni
set protocols bgp parameters router-id 10.0.1.211
set protocols bgp neighbor 10.0.1.208 peer-group evpn
set protocols bgp peer-group evpn update-source dum0
set protocols bgp peer-group evpn remote-as 102
set protocols bgp peer-group evpn address-family l2vpn-evpn nexthop-self



set interfaces vxlan vxlan101 source-address 10.0.1.211
set interfaces vxlan vxlan101 vni 101
set interfaces vxlan vxlan101 mtu 1500

set interfaces bridge br101 address 10.2.2.1/22
set interfaces bridge br101 description 'client x2'
set interfaces bridge br101 member interface eth1
set interfaces bridge br101 member interface vxlan101


commit 

save
```

## Router DC.P1
```
conf t
mpls ip
ip cef
router ospf 1
router-id 2.2.2.2
router bgp 43100
bgp router-id 10.10.10.10
neighbor 10.0.1.232 remote-as 43100 ! C1
neighbor 10.0.1.232 update-source lo0

neighbor 10.0.1.228 remote-as 43100 ! A2
neighbor 10.0.1.228 update-source lo0

address-family vpnv4

neighbor 10.0.1.232 activate
neighbor 10.0.1.232 send-community both

neighbor 10.0.1.228 activate
neighbor 10.0.1.228 send-community both

address-family ipv4 vrf VPN-1
redistribute connected

exit

ip vrf VPN-1
rd 200:1
route-target import 200:1
route-target export 200:1

interface f0/1
ip address 10.0.0.2 255.255.255.192
ip ospf 1 area 0
mpls ip
no shut

interface f0/0
ip address 10.0.2.1 255.255.255.0
ip vrf forwarding VPN-1
ip address 10.0.2.1 255.255.255.0
no shut

int lo0
ip address 10.0.1.208 255.255.255.255
ip ospf 1 area 0




```
# Aveiro DC
## VyOS DC.AC1
```
sudo cp /opt/vyatta/etc/config.boot.default /config/config.boot
reboot

configure

set interfaces ethernet eth0 address 10.0.0.130/26
set interfaces dummy dum0 address 10.0.1.212/32
set protocols ospf area 0 network 10.0.0.128/26
set protocols ospf area 0 network 10.0.1.212/32
set system host-name DCA1

set protocols bgp system-as 43100
set protocols bgp address-family l2vpn-evpn advertise-all-vni
set protocols bgp parameters router-id 10.0.1.212
set protocols bgp neighbor 10.0.1.208 peer-group evpn
set protocols bgp peer-group evpn update-source dum0
set protocols bgp peer-group evpn remote-as 102
set protocols bgp peer-group evpn address-family l2vpn-evpn nexthop-self

set interfaces vxlan vxlan101 source-address 10.0.1.212
set interfaces vxlan vxlan101 vni 101
set interfaces vxlan vxlan101 mtu 1500

set interfaces bridge br101 address 10.2.3.1/22
set interfaces bridge br101 description 'client x3'
set interfaces bridge br101 member interface eth1
set interfaces bridge br101 member interface vxlan101

commit
save

```

## Router DC.A2
```
conf t

router ospf 1
router-id 8.3.3.3
mpls ip
ip cef

router bgp 43100
bgp router-id 10.10.10.12
neighbor 10.0.1.232 remote-as 43100 ! C1
neighbor 10.0.1.232 update-source lo0


neighbor 10.0.1.208 remote-as 43100 ! P1
neighbor 10.0.1.208 update-source lo0

address-family vpnv4

neighbor 10.0.1.232 activate
neighbor 10.0.1.232 send-community both

neighbor 10.0.1.208 activate
neighbor 10.0.1.208 send-community both

address-family ipv4 vrf VPN-1
redistribute connected

exit

ip vrf VPN-1
rd 200:1
route-target import 200:1
route-target export 200:1



interface f0/1
ip address 10.0.0.193 255.255.255.192
ip ospf 1 area 0
mpls ip
no shut

interface f0/0
ip address 10.0.3.1 255.255.255.0
ip vrf forwarding VPN-1
ip address 10.0.3.1 255.255.255.0
no shut

int lo0
ip address 10.0.1.228 255.255.255.255
ip ospf 1 area 0


```

# Coimbra DC

## Router DC.C1
```
conf t
mpls ip
ip cef
router ospf 1
router-id 4.4.3.3
router bgp 43100
bgp router-id 10.10.10.14
neighbor 10.0.1.228 remote-as 43100 ! A2
neighbor 10.0.1.228 update-source Lo0


neighbor 10.0.1.208 remote-as 43100 ! P1
neighbor 10.0.1.208 update-source Lo0

address-family vpnv4

neighbor 10.0.1.228 activate
neighbor 10.0.1.228 send-community both

neighbor 10.0.1.208 activate
neighbor 10.0.1.208 send-community both

address-family ipv4 vrf VPN-1
redistribute connected

exit

ip vrf VPN-1
rd 200:1
route-target import 200:1
route-target export 200:1

interface f0/1
ip address 10.0.1.2 255.255.255.192
mpls ip
ip ospf 1 area 0
no shut

interface f0/0
ip address 10.0.1.1 255.255.255.0
ip vrf forwarding VPN-1
ip address 10.0.1.1 255.255.255.0
no shut

int lo0
ip address 10.0.1.232 255.255.255.255
ip ospf 1 area 0

```
