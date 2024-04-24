# Nokia SR Linux Generic Lab

Welcome to the generic SR Linux lab, where you will learn and experience Nokia's SR Linux network operating system.

In this lab you can:

- Deploy the pre-configured SR Linux fabric with workloads using [containerlab](https://containerlab.dev).
- Explore the underlay routing with eBGP and a typical BGP EVPN overlay setup leveraging Route Reflectors.
- Examine the IP-VRF and MAC-VRF network instances, VXLAN tunnels, and IRB interfaces.
- Verify VXLAN datapath by pinging between the hosts.
- And discover everything you want to know about SR Linux!

**Difficulty:** Beginner

## Topology

The lab represents a 2-Tier Clos data center fabric topology with four hosts.

The Leaf and Spine nodes are running containerized Nokia SR Linux (7220 IXR D3L) NOS and the hosts are based on Alpine Linux.

Topology diagram:

![pic](https://gitlab.com/rdodin/pics/-/wikis/uploads/720785fa42f2e377c4319a9ba56819e9/image.png)

Every node in the data center fabric is configured with eBGP as an underlay routing protocol. iBGP is used to setup EVPN-based overlay layer 2 and layer 3 services.

Hosts are connected to the respective overlay services as per the diagram below:

![image](https://gitlab.com/rdodin/pics/-/wikis/uploads/aaf85bdaf5fe9a9a378446eea2c05e4f/image.png)

## Deploying the lab

Deploy the topology with containerlab.

```bash
cd $HOME/DCFWorkshop/srl-generic-lab
sudo clab deploy -c -t srl-generic.clab.yml
```
### Configure from scratch or Explore a ready-made fabric

In this lab, you may choose to build out the fabric from scratch or explore a fabric that will fully configured.

If you choose to build from scratch, you will find step-by-step SRLinux configuration examples on this page. The hosts are auto configured.

The default setting is to build from scratch. If you choose to explore a configured fabric, please uncomment the line for the startup-config in the containerlab topology file `srl-generic.clab.yml` for all 6 nodes.

```
#startup-config: configs/fabric/leaf1.cfg
```

## Credentials & Access

Once the lab is running, you can access the network elements from inside the VM through **ssh**, just passing the node name:
```
ssh admin@clab-srl-generic-leaf1
# password: NokiaSrl1!
```
To access host h1:
```
# password: srllabs@123
sudo docker exec -it clab-srl-generic-h1 bash
```

## Configuring & Exploring SR Linux

### Configure underlay

Underlay Connectivity:

![image](/underlay.jpg)

#### Configure System IP
```
/interface system0 admin-state enable subinterface 0 ipv4 admin-state enable address 10.0.0.1/32
```

#### Configure Loopback
```
/interface lo1 admin-state enable subinterface 1 ipv4 admin-state enable address 100.100.100.100/32
```

#### Configure Leaf-Spine links
```
/interface ethernet-1/31 admin-state enable vlan-tagging true
/interface ethernet-1/31 subinterface 1 ipv4 admin-state enable address 100.64.1.0/31
/interface ethernet-1/31 subinterface 1 vlan encap single-tagged vlan-id 1
```

#### Configure Host L2 link
```
/interface ethernet-1/11 admin-state enable ethernet port-speed 10G
/interface ethernet-1/11 subinterface 1 type bridged
```

#### Configure Host L3 link
```

```

#### Configure BFD on Leaf-Spine links
```
/bfd subinterface ethernet-1/31.1 admin-state enable
/bfd subinterface ethernet-1/31.1 desired-minimum-transmit-interval 100000
/bfd subinterface ethernet-1/31.1 required-minimum-receive 100000
/bfd subinterface ethernet-1/31.1 detection-multiplier 3
/bfd subinterface ethernet-1/31.1 minimum-echo-receive-interval 0
```

#### Configure irb

```
/interface irb1 admin-state enable
/interface irb1 subinterface 1 admin-state enable ip-mtu 9000
/interface irb1 subinterface 1 anycast-gw virtual-router-id 1
/interface irb1 subinterface 1 ipv4 admin-state enable address 100.101.1.1/24 anycast-gw true primary
/interface irb1 subinterface 1 ipv4 arp learn-unsolicited true
/interface irb1 subinterface 1 ipv4 arp host-route populate static
/interface irb1 subinterface 1 ipv4 arp host-route populate dynamic
/interface irb1 subinterface 1 ipv4 arp evpn advertise static
/interface irb1 subinterface 1 ipv4 arp evpn advertise dynamic
```

#### Configure underlay VRF

```
/network-instance default type default
/network-instance default admin-state enable
/network-instance default interface ethernet-1/31.1
/network-instance default interface ethernet-1/32.1
/network-instance default interface system0.0
```

#### Configure underlay BGP

```
/network-instance default protocols bgp admin-state enable
/network-instance default protocols bgp autonomous-system 64601
/network-instance default protocols bgp router-id 10.0.0.1
/network-instance default protocols bgp afi-safi evpn evpn rapid-update true
/network-instance default protocols bgp afi-safi ipv4-unicast admin-state enable
/network-instance default protocols bgp afi-safi ipv4-unicast multipath allow-multiple-as true
/network-instance default protocols bgp afi-safi ipv4-unicast multipath max-paths-level-1 64

/network-instance default protocols bgp group ebgp-underlay failure-detection enable-bfd true
/network-instance default protocols bgp group ebgp-underlay failure-detection fast-failover true

/network-instance default protocols bgp neighbor 100.64.1.1 admin-state enable
/network-instance default protocols bgp neighbor 100.64.1.1 peer-as 65177
/network-instance default protocols bgp neighbor 100.64.1.1 peer-group ebgp-underlay
/network-instance default protocols bgp neighbor 100.64.1.5 admin-state enable
/network-instance default protocols bgp neighbor 100.64.1.5 peer-as 65177
/network-instance default protocols bgp neighbor 100.64.1.5 peer-group ebgp-underlay
```

#### Configure BGP Policy for underlay

```
/routing-policy prefix-set loopbacks prefix 10.0.0.0/24 mask-length-range 32..32

/routing-policy policy export-to-underlay default-action policy-result reject
/routing-policy policy export-to-underlay statement 20 match prefix-set loopbacks
/routing-policy policy export-to-underlay statement 20 action policy-result accept
/routing-policy policy export-to-underlay statement 30 match protocol bgp
/routing-policy policy export-to-underlay statement 30 match family [ ipv4-unicast ]
/routing-policy policy export-to-underlay statement 30 action policy-result accept

/routing-policy policy import-from-underlay default-action policy-result reject
/routing-policy policy import-from-underlay statement 20 match prefix-set loopbacks
/routing-policy policy import-from-underlay statement 20 action policy-result accept
/routing-policy policy import-from-underlay statement 30 match protocol bgp
/routing-policy policy import-from-underlay statement 30 match family [ ipv4-unicast ]
/routing-policy policy import-from-underlay statement 30 action policy-result accept

/network-instance default protocols bgp group ebgp-underlay export-policy export-to-underlay
/network-instance default protocols bgp group ebgp-underlay import-policy import-from-underlay
```

### Explore the underlay configuration

The role of the underlay routing is to distribute loopback prefixes across the leaf switches of our fabric. In this lab, we are using eBGP as the underlay routing protocol.

![pic](https://gitlab.com/rdodin/pics/-/wikis/uploads/2dc87c8921b2c42f4fa94ad141a5845f/image.png)

You can connect to nodes and check the interface configs, eBGP sessions state, AS numbers, and underlay network reachability between nodes by pinging the system IP addresses.

A list of helpful show commands:

- `show interface brief`
- `show interface ethernet-1/3*`
- `show network-instance summary`
- `show network-instance default route-table all`
- `show network-instance default protocols bgp neighbor`
- `show network-instance default protocols bgp neighbor 100.64.1.1 advertised-routes ipv4`
- `show network-instance default protocols bgp neighbor 100.64.1.1 received-routes ipv4`

`info` commands allow the retrieval of the configuration and state information from the respective datastores and provide full access to the system internals.

Getting information from the configuration datastore:

- `info interface ethernet-1/*`
- `info network-instance default`

And from the state datastore:
- `info from state interface ethernet-1/31 subinterface 1 statistics`
- `info from state network-instance default protocols bgp neighbor * received-messages`

The output can be retrieved in different formats:

As JSON:
- `info from state network-instance default protocols bgp neighbor * | filter fields admin-state session-state | as json`

As Table:
- `info from state network-instance default protocols bgp neighbor * | filter fields admin-state session-state | as table`

These commands can be associated to CLI Aliases

-`environment alias mybgp "info from state network-instance default protocols bgp neighbor * | filter fields admin-state session-state | as table"`
-`mybgp`

These CLI Aliases can receive inputs so that the neighbor can be dynamically selected (dynamic aliases)

-`environment alias mybgp "info from state network-instance default protocols bgp neighbor {} | filter fields admin-state session-state | as table"`
-`mybgp mybgp 10.0.0.5`

### iBGP EVPN as Network Virtualization Overlay

EVPN uses MP-BGP as a control plane protocol between the tunnel endpoints. Typically, route-reflectors (RRs) are configured for the scalability. In this lab, spine routers are configured as RRs. All the leaf routers are peering with the spine routers for iBGP EVPN routes.

![pic](https://gitlab.com/rdodin/pics/-/wikis/uploads/0afc55fdadc2c0e5522e3503b70d0cc2/image.png)

#### Configure BGP for overlay

Use AS65501 as AS on both leaf and spine

```
/network-instance default protocols bgp group ibgp-evpn export-policy export-all
/network-instance default protocols bgp group ibgp-evpn import-policy import-all
/network-instance default protocols bgp group ibgp-evpn peer-as 65501
/network-instance default protocols bgp group ibgp-evpn failure-detection enable-bfd true
/network-instance default protocols bgp group ibgp-evpn failure-detection fast-failover true
/network-instance default protocols bgp group ibgp-evpn afi-safi evpn admin-state enable
/network-instance default protocols bgp group ibgp-evpn afi-safi ipv4-unicast admin-state disable
/network-instance default protocols bgp group ibgp-evpn local-as as-number 65501

/network-instance default protocols bgp neighbor 10.0.0.5 admin-state enable
/network-instance default protocols bgp neighbor 10.0.0.5 peer-group ibgp-evpn
/network-instance default protocols bgp neighbor 10.0.0.5 transport local-address 10.0.0.1
/network-instance default protocols bgp neighbor 10.0.0.6 admin-state enable
/network-instance default protocols bgp neighbor 10.0.0.6 peer-group ibgp-evpn
/network-instance default protocols bgp neighbor 10.0.0.6 transport local-address 10.0.0.1
```

#### Configure BGP policy for overlay

```
/routing-policy policy export-all default-action
/routing-policy policy export-all default-action policy-result accept

/routing-policy policy import-all default-action
/routing-policy policy import-all default-action policy-result accept

/network-instance default protocols bgp group ibgp-evpn export-policy export-all
/network-instance default protocols bgp group ibgp-evpn import-policy import-all
```

Connect to the spine nodes and check RR configuration and iBGP EVPN sessions across the data center fabric.

#### Configure RR on Spines

```
/network-instance default protocols bgp group ibgp-evpn next-hop-self false
/network-instance default protocols bgp group ibgp-evpn route-reflector client true
/network-instance default protocols bgp group ibgp-evpn route-reflector cluster-id 10.0.0.5
```

Some useful commands:

- `show network-instance default protocols bgp neighbor 10.0.0.5 advertised-routes evpn`
- `show network-instance default protocols bgp neighbor 10.0.0.5  received-routes evpn`
- `info network-instance default protocols bgp group ibgp-evpn`
- `info from state network-instance default protocols bgp neighbor 10.0.0.5 peer-type`

### MAC-VRFs & IP-VRFs with EVPN

MAC-VRFs are layer 2 EVPN instances usually mapped to a single broadcast domain and forward based on the mac-table.
IP-VRFs are layer 3 instances with IP routing tables. MAC-VRFs can bind to IP-VRFs using IRB interfaces.

Connect to the SR Linux nodes to check the EVPN service building blocks; IP-VRF and MAC-VRF configurations, VXLAN tunnel and IRB interfaces.

Have a look at the overlay topology and how different VRF types are mapped:

![image](https://gitlab.com/rdodin/pics/-/wikis/uploads/b60a887995e199a4ca373657628ac486/image.png)

#### Configure VxLAN Interface

```
/tunnel-interface vxlan1 vxlan-interface 1000 type routed
/tunnel-interface vxlan1 vxlan-interface 1000 ingress
/tunnel-interface vxlan1 vxlan-interface 1000 ingress vni 1000

/tunnel-interface vxlan1 vxlan-interface 1001 type bridged
/tunnel-interface vxlan1 vxlan-interface 1001 ingress
/tunnel-interface vxlan1 vxlan-interface 1001 ingress vni 1001
```

#### Configure MAC-VRF (L2 EVPN)

```
/network-instance mac-vrf-1 type mac-vrf
/network-instance mac-vrf-1 admin-state enable
/network-instance mac-vrf-1 interface ethernet-1/11.1
/network-instance mac-vrf-1 interface irb1.1
/network-instance mac-vrf-1 vxlan-interface vxlan1.1001
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 admin-state enable
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 vxlan-interface vxlan1.1001
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 evi 1001
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 ecmp 4
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 routes bridge-table next-hop use-system-ipv4-address
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 routes bridge-table mac-ip
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 routes bridge-table mac-ip advertise true
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 routes bridge-table inclusive-mcast
/network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 routes bridge-table inclusive-mcast advertise true
/network-instance mac-vrf-1 protocols bgp-vpn bgp-instance 1 route-target export-rt target:65501:1001
/network-instance mac-vrf-1 protocols bgp-vpn bgp-instance 1 route-target import-rt target:65501:1001
```

#### Configure IP-VRF (L3 EVPN)

```
/network-instance ip-vrf-1 type ip-vrf
/network-instance ip-vrf-1 admin-state enable
/network-instance ip-vrf-1 interface irb1.1
/network-instance ip-vrf-1 interface lo1.1
/network-instance ip-vrf-1 vxlan-interface vxlan1.1000
/network-instance ip-vrf-1 protocols bgp-evpn bgp-instance 1 admin-state enable
/network-instance ip-vrf-1 protocols bgp-evpn bgp-instance 1 vxlan-interface vxlan1.1000
/network-instance ip-vrf-1 protocols bgp-evpn bgp-instance 1 evi 1000
/network-instance ip-vrf-1 protocols bgp-evpn bgp-instance 1 ecmp 4
/network-instance ip-vrf-1 protocols bgp-vpn bgp-instance 1 route-target export-rt target:65501:1000
/network-instance ip-vrf-1 protocols bgp-vpn bgp-instance 1 route-target import-rt target:65501:1000
```

Some useful commands:

- `info network-instance ip-vrf-1`
- `info from state network-instance mac-vrf-1 protocols`
- `show network-instance mac-vrf-1 bridge-table mac-table all`
- `show network-instance ip-vrf-1 route-table all`
- `info tunnel-interface vxlan1 vxlan-interface *`
- `show tunnel-interface vxlan1 vxlan-interface * detail`
- `info interface irb1`
- `info interface ethernet-1/1*`
- `show network-instance default protocols bgp routes evpn route-type summary`

### Ping between hosts

With overlay services deployed, you can successfully test network reachability between the hosts.

Connect to the hosts and `ping` from:

- **h1 to h4** *(ping goes over mac-vrf-1)*

```
sudo docker exec -it clab-srl-generic-h1 ping 100.101.1.14
```

- **h1 to eth1 of h3** *(ping goes over ip-vrf-1)*

```
sudo docker exec -it clab-srl-generic-h1 ping 100.101.3.13
```

- **h2 to eth2 of h3** *(ping goes over ip-vrf-2)*

```
sudo docker exec -it clab-srl-generic-h2 ping 100.102.4.13
```

- **h3 to 100.100.100.100** *(ping an address on ip-vrf-1 and ip-vrf-2)*

```
sudo docker exec -it clab-srl-generic-h3 ping 100.100.100.100
```

Check the mac/ip table entries and EVPN route advertisements on the related SR Linux nodes with:

- `show network-instance mac-vrf-1 bridge-table mac-table all`
- `show network-instance ip-vrf-1 route-table all`
- `show network-instance default protocols bgp neighbor 10.0.0.5  received-routes evpn`
- `show network-instance default protocols bgp routes evpn route-type summary`
- `show network-instance default protocols bgp routes evpn route-type 2 detail`
- `show network-instance default tunnel-table ipv4`
- `show tunnel-interface vxlan1 vxlan-interface * detail`
- `info from state tunnel vxlan-tunnel vtep * statistics`

## References

- [SR Linux documentation](https://documentation.nokia.com/srlinux/)
- [Learn SR Linux](https://learn.srlinux.dev/)
