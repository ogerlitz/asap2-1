##### Table of Contents
1. [Motivation](#motivation)
2. [mlxsw](#mlxsw)
    1. [Features by Version](#features-by-version)
    2. [Known Issues](#known-issues)
3. [Reporting Issues](#reporting-issues)

Motivation
----------

switchdev is an infrastructure in the Linux kernel which facilitates the
offloading of the kernel's forwarding plane to capable ASICs.

switchdev allows users and developers to utilize current ASICs by using a
standardized and well-known API exposed by the Linux kernel instead of
relying on proprietary APIs implemented in binary user space blobs.

By using the Linux kernel to configure the hardware, users can use the same
familiar tools to configure both their servers and switches. The sole
difference would be the performance gained by offloading the kernel's
forwarding plane to the switch's ASIC.

```
switch$ ip link add name br0 type bridge
switch$ ip link set dev br0 type bridge vlan_filtering 1
switch$ ip link set dev sw1p1 master br0
switch$ ip link set dev sw1p2 master br0
switch$ ip link set dev br0 up

hostA$ iperf –s –i1
hostB$ iperf -c hostA -i1 -P 8
------------------------------------------------------------
Client connecting to 192.168.1.1, TCP port 5001
TCP window size: 85.0 KByte (default)
------------------------------------------------------------
...
[SUM]  1.0- 2.0 sec  10.5 GBytes  90.6 Gbits/sec
```

For more information about switchdev please refer to the kernel's
[Switchdev documentation][1].

mlxsw
-----

Mellanox Technologies is the first hardware vendor to use the switchdev API to
offload the kernel's forwarding plane to a real ASIC. Mellanox's current
switchdev-based solution is focused on the 100Gb/s Spectrum ASIC switches
([SN2000 Series][2]).

This is achieved by using an [upstream driver][3] in the Linux kernel. A user
can simply buy a switch, install Linux on it like any other server and benefit
from the underlying hardware.

#### Features by Version

| Kernel Version |                                                                        |
|:--------------:|:---------------------------------------------------------------------- |
| 4.3            | SwitchX-2 driver submission. Slow path only. Not in active development |
| 4.4            | Spectrum driver submission. VLAN-aware bridge offload                  |
| 4.5            | LAG and VLAN-unaware bridges                                           |
| 4.6            | devlink infrastructure and port splitter                               |
| 4.7            | Quality of Service: DCB and shared buffers                             |
| 4.8            | IPv4 unicast router, port mirroring, extended ethtool statistics       |
| 4.9            | Extended ethtool support, HW stats query via iproute                   |
| 4.10           | SwitchIB support, I2C support, CPU policer                             |
| 4.11           | tc-flower offload, enhanced L3 offload, packet sampling                |
| [4.12](4.12-Release-notes) | tc-vlan offload, VRFs, ACL activity and stats dumping, OVS offload     |
| [4.13](4.13-Release-notes) | Match on TCP flags, firmware flashing, trap action, port module info   |
| [4.14](4.14-Release-notes) | IPv6 unicast router, match on IP TTL and TOS, tc-multichain offload, GRE tunnels |
| [4.15](4.15-Release-notes) | IPv4 multicast router, IPv4 non-equal-cost multi-path, multi-path hash policy, RED queueing discipline |
| [4.16](4.16-Release-notes) | IPv6 non-equal-cost multi-path, PRIO scheduler, flow based mirroring |
| [4.17](4.17-Release-notes) | RED as a child of PRIO, IPv6 multicast router, ERSPAN |
| 4.19           | Virtual Router Redundancy Protocol (VRRP) |

#### Known Issues

The following is a list of known issues that will be resolved in future releases:

1. Order of operations. Network configuration should be done in a
   bottom-up fashion. Teardown should be done top-down.
2. ~~Splitting front panel ports at the lower row to four fails~~ (Fixed in
4.7-rc3).
3. ACL key size. It's not possible to specify rules that match on L2
   fields and a complete IPv6 header.
4. ~~ECMP size. 32 nexthops are currently supported~~ (Added in 4.15).
5. ~~L2 multicast~~ (Fixed in 4.15):
    1. ~~When multicast snooping is disabled:~~
        1. ~~MDB entries are still in use.~~
        2. ~~`mcast_flood` flag is ignored.~~
    2. ~~When multicast snooping is enabled:~~
        1. ~~Packets that hit an MDB entry will not be sent to ports that~~
	   ~~are makred as multicast router ports unless they are also~~
           ~~part of the MDB entry.~~
6. Creation of VLAN netdevice with VID 1 is not allowed.

   VLAN 1 is used for carrying untagged traffic on router interfaces. Starting
   with 4.17, mlxsw actually forbids creation of such VLAN netdevice. It is
   still possible to use VLAN 1 in 802.1Q bridges.

Reporting Issues
----------------

To report issues in the Wiki please send an email to:
`mlxsw [at] mellanox [dot] com`

[1]: https://www.kernel.org/doc/Documentation/networking/switchdev.txt
[2]: http://www.mellanox.com/page/products_dyn?product_family=251&mtag=sn2000
[3]: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/drivers/net/ethernet/mellanox/mlxsw
