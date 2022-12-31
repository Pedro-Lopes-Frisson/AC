# PN configuration
## Router Porto

```
conf t
router ospf 1
router-id 1.1.1.1

mpls ip
ip cef

interface f0/1 ! connection to DC.P2
ip address 10.0.0.65 255.255.255.192
ip ospf 1 area 0
mpls traffic-eng tunnels
no shut

interface f1/0 ! connectiion to DC.P1
ip address 10.0.0.1 255.255.255.192
ip ospf 1 area 0
mpls ip
no shut

interface f6/1 ! Connection to the internet
no shut

interface f1/1 ! connection to Aveiro
ip address 10.0.1.201 255.255.255.252
mpls ip
ip ospf 1 area 0 ! enable ospf inside PN
no shut

interface f0/0 ! connection to Coimbra
ip address 10.0.1.205 255.255.255.252
mpls ip
ip ospf 1 area 0 ! enable ospf inside PN
no shut

!interface lo0 ! bgp connections
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
mpls ip
no shut
ip ospf 1 area 0

int f0/0 ! connection to DC.A1
ip address 10.0.1.129 255.255.255.192
ip ospf 1 area 0
mpls traffic-eng tunnels
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
router ospf 1
router-id 1.1.1.3
mpls ip 
ip cef

int f1/1 ! connection to DC.C1
ip address 10.0.1.1  255.255.255.192
ip ospf 1 area 0
mpls ip
no shut

int f1/0 ! connection to Aveiro
ip address 10.0.1.197  255.255.255.252
ip ospf 1 area 0
mpls ip
no shut

int f0/0 ! connection to Porto
ip address 10.0.1.206  255.255.255.252
ip ospf 1 area 0
mpls ip
no shut

int f0/1 ! connection to Lisboa
ip address 10.0.1.193  255.255.255.252
mpls ip
ip ospf 1 area 0
no shut

```

## Router Lisboa
```
conf t
mpls ip
router ospf 1
router-id 1.1.1.4
ip cef


int f0/0 ! connection to DC.L1
ip address 10.0.1.65  255.255.255.192
mpls ip
ip ospf 1 area 0
no shut

int f0/1 ! connection to Coimbra
ip address 10.0.1.194  255.255.255.252
ip ospf 1 area 0
mpls traffic-eng tunnels
no shut

int f6/1  ! connection to internet
no shut

```


# Lisboa DC
## VyOS DC.L1

```
sudo cp /opt/vyatta/etc/config.boot.default /config/config.boot
reboot
```

# Porto DC
## VyOS DC.P2

```
sudo cp /opt/vyatta/etc/config.boot.default /config/config.boot
reboot


configure
set interfaces ethernet eth0 address 10.0.0.66/26
set protocols ospf area 0 network 10.0.0.64/26
commit
save


set interfaces ethernet eth1 vif 2
set interfaces ethernet eth1 vif 3
commit
save

set interfaces vxlan vxlan102 vni 101
set interfaces vxlan vxlan102 mtu 1500
set interface vxlan vxlan102 remote 10.0.1.130

set interfaces vxlan vxlan103 vni 102
set interfaces vxlan vxlan103 mtu 1500
set interface vxlan vxlan103 remote 10.0.1.130
commit
save

set interfaces bridge br102 member interface 'eth1.2'
set interfaces bridge br102 member interface 'vxlan102'
set interfaces bridge br103 member interface 'eth1.3'
set interfaces bridge br103 member interface 'vxlan103'
commit




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
set interfaces ethernet eth0 address 10.0.1.130/26
set protocols ospf area 0 network 10.0.1.128/26
commit
save


set interfaces ethernet eth1 vif 2
set interfaces ethernet eth1 vif 3
commit
save

set interfaces vxlan vxlan102 vni 101
set interfaces vxlan vxlan102 mtu 1500
set interface vxlan vxlan102 remote 10.0.0.66

set interfaces vxlan vxlan103 vni 102
set interfaces vxlan vxlan103 mtu 1500
set interface vxlan vxlan103 remote 10.0.0.66
commit
save

set interfaces bridge br102 member interface 'eth1.2'
set interfaces bridge br102 member interface 'vxlan102'
set interfaces bridge br103 member interface 'eth1.3'
set interfaces bridge br103 member interface 'vxlan103'
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
neighbor 10.0.1.228 update-source lo0


neighbor 10.0.1.208 remote-as 43100 ! P1
neighbor 10.0.1.208 update-source lo0

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
