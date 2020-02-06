# Multi Datacenter VXLan Example #

![Topology](multi_dc_vxlan.jpg?raw=true "Topology")

In this example we use vxlan to extend bridge domains vlan 101 and 102 for a
single tenant.

#### Caveats and Notes ####
It appears DHCP relay is not fully implemented in these nxosv yet. There is a
weird iosl2v bug that prevents forwarding of NAT traffic from the inside
interfaces. This is somehow resolved by adding a simple ACL with logging on to
the inbound, inside, interfaces. I have two ACL's with identical configs but
names corresponding to their interface. In this way I could see traffic move
asynchronously through active links successfully. You would not normally need to
do anything like this.

I have also run into issues with Eve and nxosv with packet loss and dup packets.
Historically I have not been able to tell if this was eve/lab related or config
error. So far I have no been able to reproduce these issues in hardware and when
using 9k switches many of these issues resolved themselves.

#### Output Notes ####

__Host Pings__

```
root@ubuntu:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:00:00:0d:00 brd ff:ff:ff:ff:ff:ff
    inet 172.16.101.49/24 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::250:ff:fe00:d00/64 scope link
       valid_lft forever preferred_lft forever
root@ubuntu:~# ping 172.16.101.47
PING 172.16.101.47 (172.16.101.47) 56(84) bytes of data.
64 bytes from 172.16.101.47: icmp_seq=1 ttl=64 time=13.0 ms
64 bytes from 172.16.101.47: icmp_seq=2 ttl=64 time=11.4 ms
64 bytes from 172.16.101.47: icmp_seq=3 ttl=64 time=13.4 ms
^C
--- 172.16.101.47 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 11.429/12.632/13.462/0.870 ms
root@ubuntu:~# ping 172.16.101.1
PING 172.16.101.1 (172.16.101.1) 56(84) bytes of data.
64 bytes from 172.16.101.1: icmp_seq=1 ttl=255 time=6.37 ms
64 bytes from 172.16.101.1: icmp_seq=2 ttl=255 time=4.16 ms
64 bytes from 172.16.101.1: icmp_seq=3 ttl=255 time=4.80 ms
^C
--- 172.16.101.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 4.167/5.114/6.370/0.927 ms
root@ubuntu:~# ping 172.16.101.48
PING 172.16.101.48 (172.16.101.48) 56(84) bytes of data.
64 bytes from 172.16.101.48: icmp_seq=1 ttl=64 time=9.44 ms
64 bytes from 172.16.101.48: icmp_seq=2 ttl=64 time=11.0 ms
^C
--- 172.16.101.48 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 9.441/10.270/11.099/0.829 ms
root@ubuntu:~# ping 172.16.101.49
PING 172.16.101.49 (172.16.101.49) 56(84) bytes of data.
64 bytes from 172.16.101.49: icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from 172.16.101.49: icmp_seq=2 ttl=64 time=0.027 ms
^C
--- 172.16.101.49 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.027/0.035/0.044/0.010 ms
root@ubuntu:~# ping 4.2.2.1
PING 4.2.2.1 (4.2.2.1) 56(84) bytes of data.
64 bytes from 4.2.2.1: icmp_seq=1 ttl=55 time=46.5 ms
64 bytes from 4.2.2.1: icmp_seq=2 ttl=55 time=41.8 ms
64 bytes from 4.2.2.1: icmp_seq=3 ttl=55 time=42.9 ms
^C
--- 4.2.2.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 41.847/43.785/46.583/2.026 ms
root@ubuntu:~# ping 172.16.102.48
PING 172.16.102.48 (172.16.102.48) 56(84) bytes of data.
64 bytes from 172.16.102.48: icmp_seq=1 ttl=62 time=15.4 ms
64 bytes from 172.16.102.48: icmp_seq=2 ttl=62 time=13.4 ms
64 bytes from 172.16.102.48: icmp_seq=3 ttl=62 time=12.3 ms
64 bytes from 172.16.102.48: icmp_seq=4 ttl=62 time=15.6 ms
^C
--- 172.16.102.48 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 12.373/14.213/15.669/1.381 ms

```

__dc2_a output__
```
dc2_a# show l2route mac-ip all
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated
Topology    Mac Address    Prod   Flags         Seq No     Host IP         Next-
Hops
----------- -------------- ------ ---------- --------------- ---------------
101         5000.0006.0000 BGP    --            0          172.16.101.10  10.100
.1.1
101         0050.0000.0900 BGP    --            0          172.16.101.47  10.100
.1.1
101         0050.0000.0700 HMM    --            0          172.16.101.48  Local

101         0050.0000.0d00 HMM    --            0          172.16.101.49  Local

102         0050.0000.0a00 BGP    --            0          172.16.102.48  10.100
.1.1
102         0050.0000.0800 HMM    --            0          172.16.102.49  Local

dc2_a#
dc2_a# show ip route vrf TENANT-1
IP Route Table for VRF "TENANT-1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

0.0.0.0/0, ubest/mbest: 1/0
    *via 10.100.1.1%default, [200/0], 03:57:46, bgp-65001, internal, tag 65001 (
evpn) segid: 16100 tunnelid: 0xa640101 encap: VXLAN

172.16.0.3/32, ubest/mbest: 2/0, attached
    *via 172.16.0.3, Lo1, [0/0], 05:28:09, local
    *via 172.16.0.3, Lo1, [0/0], 05:28:09, direct
172.16.0.4/32, ubest/mbest: 1/0
    *via 10.100.0.4%default, [200/0], 02:05:44, bgp-65001, internal, tag 65001 (
evpn) segid: 16100 tunnelid: 0xa640004 encap: VXLAN

172.16.101.0/24, ubest/mbest: 1/0, attached
    *via 172.16.101.1, Vlan101, [0/0], 05:27:08, direct
172.16.101.1/32, ubest/mbest: 1/0, attached
    *via 172.16.101.1, Vlan101, [0/0], 05:27:08, local
172.16.101.10/32, ubest/mbest: 1/0
    *via 10.100.1.1%default, [200/0], 02:42:10, bgp-65001, internal, tag 65001 (
evpn) segid: 16100 tunnelid: 0xa640101 encap: VXLAN

172.16.101.47/32, ubest/mbest: 1/0
    *via 10.100.1.1%default, [200/0], 01:18:29, bgp-65001, internal, tag 65001 (
evpn) segid: 16100 tunnelid: 0xa640101 encap: VXLAN

172.16.101.48/32, ubest/mbest: 1/0, attached
    *via 172.16.101.48, Vlan101, [190/0], 01:43:48, hmm
172.16.101.49/32, ubest/mbest: 1/0, attached
    *via 172.16.101.49, Vlan101, [190/0], 03:27:15, hmm
172.16.102.0/24, ubest/mbest: 1/0, attached
    *via 172.16.102.1, Vlan102, [0/0], 05:27:08, direct
172.16.102.1/32, ubest/mbest: 1/0, attached
    *via 172.16.102.1, Vlan102, [0/0], 05:27:08, local
172.16.102.48/32, ubest/mbest: 1/0
    *via 10.100.1.1%default, [200/0], 01:17:15, bgp-65001, internal, tag 65001 (
evpn) segid: 16100 tunnelid: 0xa640101 encap: VXLAN

172.16.102.49/32, ubest/mbest: 1/0, attached
    *via 172.16.102.49, Vlan102, [190/0], 01:20:48, hmm
172.16.204.20/32, ubest/mbest: 1/0
    *via 10.100.1.1%default, [200/0], 04:17:48, bgp-65001, internal, tag 65001 (
evpn) segid: 16100 tunnelid: 0xa640101 encap: VXLAN

dc2_a# show bgp l2vpn evpn vrf TENANT-1
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 384, Local Router ID is 10.100.0.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.100.0.1:3
*>i[5]:[0]:[0]:[0]:[0.0.0.0]/224
                      10.100.1.1                        100          0 i

Route Distinguisher: 10.100.0.1:32868
*>i[2]:[0]:[0]:[48]:[0000.2222.3333]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0900]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[5000.0006.0000]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0900]:[32]:[172.16.101.47]/272
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[5000.0006.0000]:[32]:[172.16.101.10]/272
                      10.100.1.1                        100          0 i
*>i[3]:[0]:[32]:[10.100.1.1]/88
                      10.100.1.1                        100          0 i

Route Distinguisher: 10.100.0.1:32869
*>i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[172.16.102.48]/272
                      10.100.1.1                        100          0 i
*>i[3]:[0]:[32]:[10.100.1.1]/88
                      10.100.1.1                        100          0 i

Route Distinguisher: 10.100.0.2:3
*>i[5]:[0]:[0]:[0]:[0.0.0.0]/224
                      10.100.1.1                        100          0 i
*>i[5]:[0]:[0]:[32]:[172.16.204.20]/224
                      10.100.1.1                        100          0 i

Route Distinguisher: 10.100.0.2:32868
*>i[2]:[0]:[0]:[48]:[0000.2222.3333]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0900]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[5000.0006.0000]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0900]:[32]:[172.16.101.47]/272
                      10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[5000.0006.0000]:[32]:[172.16.101.10]/272
                      10.100.1.1                        100          0 i
*>i[3]:[0]:[32]:[10.100.1.1]/88
                      10.100.1.1                        100          0 i

Route Distinguisher: 10.100.0.2:32869
*>i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[172.16.102.48]/272
                      10.100.1.1                        100          0 i
*>i[3]:[0]:[32]:[10.100.1.1]/88
                      10.100.1.1                        100          0 i

Route Distinguisher: 10.100.0.3:32868    (L2VNI 16101)
* i[2]:[0]:[0]:[48]:[0000.2222.3333]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>i                   10.100.1.1                        100          0 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0700]:[0]:[0.0.0.0]/216
                      10.100.1.2                        100      32768 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0900]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0d00]:[0]:[0.0.0.0]/216
                      10.100.1.2                        100      32768 i
*>i[2]:[0]:[0]:[48]:[5000.0006.0000]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[172.16.101.48]/272
                      10.100.1.2                        100      32768 i
* i[2]:[0]:[0]:[48]:[0050.0000.0900]:[32]:[172.16.101.47]/272
                      10.100.1.1                        100          0 i
*>i                   10.100.1.1                        100          0 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0d00]:[32]:[172.16.101.49]/272
                      10.100.1.2                        100      32768 i
*>i[2]:[0]:[0]:[48]:[5000.0006.0000]:[32]:[172.16.101.10]/272
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>i[3]:[0]:[32]:[10.100.1.1]/88
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>l[3]:[0]:[32]:[10.100.1.2]/88
                      10.100.1.2                        100      32768 i

Route Distinguisher: 10.100.0.3:32869    (L2VNI 16102)
*>l[2]:[0]:[0]:[48]:[0000.2222.3333]:[0]:[0.0.0.0]/216
                      10.100.1.2                        100      32768 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[0]:[0.0.0.0]/216
                      10.100.1.1                        100          0 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0800]:[32]:[172.16.102.49]/272
                      10.100.1.2                        100      32768 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[172.16.102.48]/272
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>i[3]:[0]:[32]:[10.100.1.1]/88
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>l[3]:[0]:[32]:[10.100.1.2]/88
                      10.100.1.2                        100      32768 i

Route Distinguisher: 10.100.0.4:3
*>i[5]:[0]:[0]:[32]:[172.16.0.4]/224
                      10.100.0.4                        100          0 i

Route Distinguisher: 10.100.0.3:3    (L3VNI 16100)
*>l[2]:[0]:[0]:[48]:[5000.0002.0007]:[0]:[0.0.0.0]/216
                      10.100.1.2                        100      32768 i
* i[2]:[0]:[0]:[48]:[0050.0000.0900]:[32]:[172.16.101.47]/272
                      10.100.1.1                        100          0 i
*>i                   10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[172.16.102.48]/272
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>i[2]:[0]:[0]:[48]:[5000.0006.0000]:[32]:[172.16.101.10]/272
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>i[5]:[0]:[0]:[0]:[0.0.0.0]/224
                      10.100.1.1                        100          0 i
* i                   10.100.1.1                        100          0 i
*>l[5]:[0]:[0]:[32]:[172.16.0.3]/224
                      10.100.0.3                        100      32768 i
*>i[5]:[0]:[0]:[32]:[172.16.0.4]/224
                      10.100.0.4                        100          0 i
*>i[5]:[0]:[0]:[32]:[172.16.204.20]/224
                      10.100.1.1                        100          0 i

dc2_a# show nve vni
Codes: CP - Control Plane        DP - Data Plane
       UC - Unconfigured         SA - Suppress ARP
       SU - Suppress Unknown Unicast

Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      16100    n/a               Up    CP   L3 [TENANT-1]
nve1      16101    UnicastBGP        Up    CP   L2 [101]
nve1      16102    UnicastBGP        Up    CP   L2 [102]

dc2_a# show vxlan
Vlan            VN-Segment
====            ==========
100             16100
101             16101
102             16102

dc2_a# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.100.0.3, local AS number 65001
BGP table version is 6, IPv4 Unicast config peers 3, capable peers 3
0 network entries and 0 paths using 0 bytes of memory
BGP attribute entries [0/0], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.0.1      4 65001     358     373        6    0    0 04:05:14 0
10.100.0.2      4 65001     390     373        6    0    0 04:46:54 0
10.100.0.4      4 65001     420     369        6    0    0 05:28:27 0

show vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : secondary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,100-102


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               101-102




Please check "show vpc consistency-parameters vpc <vpc-num>" for the
consistency reason of down vpc and for type-2 consistency reasons for
any vpc.

```
