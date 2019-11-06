# Multi Datacenter VXLan Example #

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
