# Welcome to SRLinux DC Fabric Workshop
This README is your starting point into the hands on section, it should get you familiar with the lab environment provided by Nokia, and provide an overview of the suggested sample labs.

During the hands on section of the Workshop, you may work individually or in groups on any of the sample labs listed below or create your own network topology and configuration.

Pre-requisite: A laptop with SSH client

If you need help, please raise your hand and a Nokia team member will be happy to assist.

## Lab Environment

A Nokia team member will provide you with a sheet of paper that contains:
- your group and VM ID
- SSH credentials to the VM instance
- URL of this repo

> <p style="color:red">!!! Make sure to backup any code, config, ... <u> offline (e.g on your laptop)</u>. 
> The VM instances might be destroyed once the Workshop is concluded.</p>

## The projects


### Overview of pre-provided projects
During this session you can work on any problem/project you are inspired to tackle or on one of the pre-provided projects of varying difficulty.
Below you can find a table with links towards those pre-provided project which you can use as a baseline for the problem/project you might want to tackle or perform the tasks we've set up for you.

If you have your own project in mind then we would suggest to use the [Standard SR Linux lab](./srl-generic-lab/).

Each pre-provided project comes with a README of it's own, please click the pre-provided projects for more information.

| Link to pre-provided project | Difficulty |
| --- | --- |
| [Standard SR Linux](./srl-generic-lab/) | # |
| [SR Linux Streaming Telemetry & NDK](./srl-telemetry-ndk-lab/) | ## |
| [SR Linux JSON-RPC with Ansible](./srl-ansible-lab/) | ## |

### Deploying a project
When accessing your Workshop VM instance you'll see the following bootstrapped environment.
The DCFPartnerHackathon directory is a git clone of this repository.

``` 
~$ ls
DCFPartnerHackathon

~$ cd DCFPartnerHackathon/

~/DCFPartnerHackathon$ ls -1
README.md
srl-ansible-lab
srl-generic-lab
srl-k8s-anycast-lab
srl-sros-gnmi-config-lab
srl-telemetry-ndk-lab
~/DCFPartnerHackathon$
```

For explanatory purposes, suppose we want to deploy the srl-generic-lab:

Change directories
```
cd $HOME/DCFPartnerHackathon/srl-generic-lab
```
Execute the `containerlab deploy` command
```
sudo containerlab deploy --reconfigure
```

> As CPU cores and memory are a finite resource, please destroy the deployed labs once your objective has been concluded by executing the `containerlab destroy` command.
```
sudo containerlab destroy --cleanup
```
### Credentials & Access
#### Accessing the lab from within the VM
To access the lab nodes from within the VM, users should identify the names of the deployed nodes using the `sudo containerlab inspect` command:

```
sudo containerlab inspect
INFO[0000] Parsing & checking topology file: sros-generic-lab.clab.yml
INFO[0000] Parsing & checking topology file: srl-generic.clab.yml 
+----+-------------------------+--------------+------------------------------+---------------+---------+----------------+--------------+
| #  |          Name           | Container ID |            Image             |     Kind      |  State  |  IPv4 Address  | IPv6 Address |
+----+-------------------------+--------------+------------------------------+---------------+---------+----------------+--------------+
|  1 | clab-srl-generic-h1     | 2be75127cd0f | ghcr.io/srl-labs/alpine      | linux         | running | 172.20.0.31/24 | N/A          |
|  2 | clab-srl-generic-h2     | 8d96d3078599 | ghcr.io/srl-labs/alpine      | linux         | running | 172.20.0.32/24 | N/A          |
|  3 | clab-srl-generic-h3     | 7d8c71ad7434 | ghcr.io/srl-labs/alpine      | linux         | running | 172.20.0.33/24 | N/A          |
|  4 | clab-srl-generic-h4     | 180a2abb7958 | ghcr.io/srl-labs/alpine      | linux         | running | 172.20.0.34/24 | N/A          |
|  5 | clab-srl-generic-leaf1  | 085498a8fba0 | ghcr.io/nokia/srlinux:23.7.1 | nokia_srlinux | running | 172.20.0.11/24 | N/A          |
|  6 | clab-srl-generic-leaf2  | 0d6afcf5cd72 | ghcr.io/nokia/srlinux:23.7.1 | nokia_srlinux | running | 172.20.0.12/24 | N/A          |
|  7 | clab-srl-generic-leaf3  | d5794eb6b404 | ghcr.io/nokia/srlinux:23.7.1 | nokia_srlinux | running | 172.20.0.13/24 | N/A          |
|  8 | clab-srl-generic-leaf4  | 6b3bf9ba8949 | ghcr.io/nokia/srlinux:23.7.1 | nokia_srlinux | running | 172.20.0.14/24 | N/A          |
|  9 | clab-srl-generic-spine1 | c92c00cb95f0 | ghcr.io/nokia/srlinux:23.7.1 | nokia_srlinux | running | 172.20.0.21/24 | N/A          |
| 10 | clab-srl-generic-spine2 | bfcdb9f31e22 | ghcr.io/nokia/srlinux:23.7.1 | nokia_srlinux | running | 172.20.0.22/24 | N/A          |
+----+-------------------------+--------------+------------------------------+---------------+---------+----------------+--------------+
```
Using the names from the above output, we can login to the a node using the following command:

For example to access node `clab-srl-generic-leaf1` via ssh simply type:
```
ssh admin@clab-srl-generic-leaf1
```

## Useful links

* [Network Developer Portal](https://network.developer.nokia.com/)
* [containerlab](https://containerlab.dev/)
* [gNMIc](https://gnmic.openconfig.net/)

### SR Linux
* [Learn SR Linux](https://learn.srlinux.dev/)
* [YANG Browser](https://yang.srlinux.dev/)
* [gNxI Browser](https://gnxi.srlinux.dev/)
* [Ansible Collection](https://learn.srlinux.dev/ansible/collection/)

