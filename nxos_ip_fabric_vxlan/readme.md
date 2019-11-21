# NXOS Based IP Fabric with VXLan Example #

![Topology](nxos_ip_fabric.jpg?raw=true "Topology")

In this example we use vxlan to extend bridge domains vlan 101 and 102 for a
single tenant over an ip fabric. We also have DHCP and border-leaf functionality
off leaf2.

Devices are members of tenant-1. Leaf2 is a border leaf which runs OSPF inside
its own VRF. Egress_router is the gateway for tenant-1. It lives outside of the
fabric. For the purpose of the lab it routes and nats out to the internet. In
this place you might have a pair of border leaves with core routers/switches
above the IP fabric.

#### Caveats and Notes ####
For examples on how to roll out a configuration like this with ansible please
review the following github repository.

[Github Ansible IP Fabric Example](https://github.com/crosson/nxos_ip_fabric)

#### Output Notes ####

__Host Pings__

hosta_net1
```
My traceroute  [v0.86]
ubuntu (0.0.0.0)                                       Fri Nov 22 00:28:56 2019
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
          Packets               Pings
Host                                Loss%   Snt   Last   Avg  Best  Wrst StDev
1. 192.168.0.1                       0.0%    84    3.8   4.0   2.7  40.9   4.2
2. 192.168.0.1                       0.0%    84    9.4   9.9   7.9  23.4   2.3
3. 192.168.100.1                     0.0%    84   13.2  14.5  10.3  76.9   7.8
4. 10.254.254.1                      0.0%    84   15.9  18.4  13.9  73.5   9.0
5. 96.120.100.233                    0.0%    84   27.5  27.2  22.6  65.1   5.4
6. 68.87.205.49                      0.0%    83   27.6  31.7  21.9 376.6  38.8
7. po-2-rur202.seattle.wa.seattle.c  0.0%    83   26.9  29.2  22.8 156.9  15.2
8. be-220-ar01.seattle.wa.seattle.c  0.0%    83   26.5  33.1  24.1  76.3   9.9
9. lag-39.ear2.Seattle1.Level3.net  58.5%    83   27.0  27.6  22.8  50.6   5.7
10. 4.69.202.241                     87.8%    83   44.1  44.7  40.8  50.9   3.9
11. a.resolvers.level3.net            0.0%    83   53.5  45.9  39.2 133.1  12.2

root@ubuntu:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:50:00:00:07:01 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:50:00:00:07:01 brd ff:ff:ff:ff:ff:ff
4: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:50:00:00:07:01 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.201/24 brd 192.168.0.255 scope global bond0
       valid_lft forever preferred_lft forever
    inet6 fe80::250:ff:fe00:701/64 scope link
       valid_lft forever preferred_lft forever

```
