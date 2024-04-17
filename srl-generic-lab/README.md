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
cd $HOME/DCFPartnerHackathon/srl-generic-lab
sudo clab deploy -c -t srl-generic.clab.yml
```
