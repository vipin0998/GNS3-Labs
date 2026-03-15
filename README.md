prepare a project explanation based on configuration from r1 to r5

r1 config
Current configuration : 1978 bytes
!
! Last configuration change at 00:44:10 UTC Mon Mar 16 2026
upgrade fpd auto
version 15.3
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R1
!
boot-start-marker
boot-end-marker
!
aqm-register-fnf
!
!
no aaa new-model
no ip icmp rate-limit unreachable
!
!
!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
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
!
redundancy
!
!
ip tcp synwait-time 5
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 1.1.1.1 255.255.255.255
!
interface Loopback100
 ip address 100.0.0.1 255.255.255.255
!
interface FastEthernet0/0
 no ip address
 shutdown
 duplex half
!
interface FastEthernet1/0
 ip address 21.0.0.1 255.255.255.0
 duplex auto
 speed auto
!
interface FastEthernet1/1
 ip address 13.0.0.1 255.255.255.0
 duplex auto
 speed auto
!
!
router eigrp 100
 network 100.0.0.1 0.0.0.0
 redistribute ospf 123 metric 10000 100 255 1 1500
!
router ospf 123
 router-id 1.1.1.1
 redistribute eigrp 100 subnets
 network 1.1.1.1 0.0.0.0 area 0
 network 13.0.0.0 0.0.0.255 area 0
 network 21.0.0.0 0.0.0.255 area 0
 network 100.0.0.0 0.0.0.0 area 0
!
router bgp 123
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 neighbor 2.2.2.2 remote-as 123
 neighbor 2.2.2.2 update-source Loopback1
 neighbor 2.2.2.2 next-hop-self
 neighbor 3.3.3.3 remote-as 123
 neighbor 3.3.3.3 update-source Loopback1
 neighbor 3.3.3.3 next-hop-self
!
ip forward-protocol nd
no ip http server
no ip http secure-server
!
!
!
no cdp log mismatch duplex
!
!
!
control-plane
!
!
mgcp behavior rsip-range tgcp-only
mgcp behavior comedia-role none
mgcp behavior comedia-check-media-src disable
mgcp behavior comedia-sdp-force disable
!
mgcp profile default
!
!
!
gatekeeper
 shutdown
!
!
line con 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
 stopbits 1
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
 stopbits 1
line vty 0 4
 login
 transport input all
!
!
end
