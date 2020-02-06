# Basic L2 VPC Design #

![Topology](basic_vpc_topology.jpg?raw=true "Topology")

In this example we use vxlan to extend bridge domains vlan 101 and 102 for a
single tenant.

#### Caveats and Notes ####

Due to the limitations of LACP support in these virtual environments all
port-channels are mode on.

#### Output Notes ####

__HP_Server_A__

```
root@ubuntu:/var/log# ifup bond0 --force
[ 4023.390596] bond0: option mode: unable to set because the bond device has slaves
sh: echo: I/O error
Waiting for a slave to join bond0 (will timeout after 60s)
ifquery: recursion detected for interface bond0 in pre-up phase
Internet Systems Consortium DHCP Client 4.3.3
Copyright 2004-2015 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/bond0/00:50:00:00:02:00
Sending on   LPF/bond0/00:50:00:00:02:00
Sending on   Socket/fallback
DHCPDISCOVER on bond0 to 255.255.255.255 port 67 interval 3 (xid=0xa9360049)
DHCPREQUEST of 10.0.100.51 on bond0 to 255.255.255.255 port 67 (xid=0x490036a9)
DHCPOFFER of 10.0.100.51 from 10.0.100.1
DHCPACK of 10.0.100.51 from 10.0.100.1
bound to 10.0.100.51 -- renewal in 42977 seconds.
root@ubuntu:/var/log# ping 10.0.200.42
PING 10.0.200.42 (10.0.200.42) 56(84) bytes of data.
^C
--- 10.0.200.42 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

root@ubuntu:/var/log# ping 10.0.200.52
PING 10.0.200.52 (10.0.200.52) 56(84) bytes of data.
64 bytes from 10.0.200.52: icmp_seq=1 ttl=63 time=9.95 ms
64 bytes from 10.0.200.52: icmp_seq=2 ttl=63 time=10.9 ms
64 bytes from 10.0.200.52: icmp_seq=3 ttl=63 time=11.0 ms
64 bytes from 10.0.200.52: icmp_seq=4 ttl=63 time=11.6 ms
64 bytes from 10.0.200.52: icmp_seq=5 ttl=63 time=11.1 ms
64 bytes from 10.0.200.52: icmp_seq=6 ttl=63 time=11.4 ms
^C
--- 10.0.200.52 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5008ms
rtt min/avg/max/mdev = 9.957/11.025/11.630/0.541 ms
root@ubuntu:/var/log#
```

__HP_Server_B__
```
ifup: interface bond0 already configured
root@ubuntu:~# ifup bond0 --force
Internet Systems Consortium DHCP Client 4.3.3
Copyright 2004-2015 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/bond0/00:50:00:00:03:00
Sending on   LPF/bond0/00:50:00:00:03:00
Sending on   Socket/fallback
DHCPDISCOVER on bond0 to 255.255.255.255 port 67 interval 3 (xid=0xa50f905e)
DHCPREQUEST of 10.0.200.52 on bond0 to 255.255.255.255 port 67 (xid=0x5e900fa5)
DHCPOFFER of 10.0.200.52 from 10.0.200.1
DHCPACK of 10.0.200.52 from 10.0.200.1
bound to 10.0.200.52 -- renewal in 33699 seconds.
root@ubuntu:~# ping 10.0.100.51
PING 10.0.100.51 (10.0.100.51) 56(84) bytes of data.
64 bytes from 10.0.100.51: icmp_seq=1 ttl=63 time=17.7 ms
64 bytes from 10.0.100.51: icmp_seq=2 ttl=63 time=10.3 ms
64 bytes from 10.0.100.51: icmp_seq=3 ttl=63 time=10.5 ms
64 bytes from 10.0.100.51: icmp_seq=4 ttl=63 time=10.6 ms
64 bytes from 10.0.100.51: icmp_seq=5 ttl=63 time=10.5 ms
64 bytes from 10.0.100.51: icmp_seq=6 ttl=63 time=10.1 ms
64 bytes from 10.0.100.51: icmp_seq=7 ttl=63 time=11.3 ms
64 bytes from 10.0.100.51: icmp_seq=8 ttl=63 time=10.9 ms
64 bytes from 10.0.100.51: icmp_seq=9 ttl=63 time=10.2 ms
64 bytes from 10.0.100.51: icmp_seq=10 ttl=63 time=9.54 ms
64 bytes from 10.0.100.51: icmp_seq=11 ttl=63 time=10.1 ms
64 bytes from 10.0.100.51: icmp_seq=12 ttl=63 time=9.77 ms
64 bytes from 10.0.100.51: icmp_seq=13 ttl=63 time=9.45 ms
64 bytes from 10.0.100.51: icmp_seq=14 ttl=63 time=11.6 ms
64 bytes from 10.0.100.51: icmp_seq=15 ttl=63 time=10.2 ms
64 bytes from 10.0.100.51: icmp_seq=16 ttl=63 time=10.2 ms
64 bytes from 10.0.100.51: icmp_seq=17 ttl=63 time=11.4 ms
64 bytes from 10.0.100.51: icmp_seq=18 ttl=63 time=11.4 ms
^C
--- 10.0.100.51 ping statistics ---
18 packets transmitted, 18 received, 0% packet loss, time 17024ms
rtt min/avg/max/mdev = 9.456/10.919/17.704/1.763 ms
```
