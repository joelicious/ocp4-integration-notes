# Homelab Infrastructure

While not required to have a homelab for software development, having one can be a valueable 
learning experience. The intent of this guide is to document the steps to provision an 
OpenShift environment that can host a variety of Open Source integration technologies. In addition,
it will document steps to make your OpenShift cluster accessible from outside your home
network.

## Homelab Architecture

The following is a high level diagram of the hardware for my homelab.

![Architecture][2022-okc-homelab.png] 

In the top left of the diagram is the Unifi Dream Machine Pro (UDM-Pro). This router has the ability
to create multiple networks and handles the routing between each network. For the purposes of this
overview, there are two networks created:

https://store.ui.com/collections/unifi-network-unifi-os-consoles/products/udm-pro


* 192.168.1.x : handles all non-homelab traffic in my home. This would include Network APs and general 
home stuff.
* 10.10.10.x : homelab network traffic, which at this point is an OpenShift Cluster.

The EdgeSwitch PoE+ 24 is the switch that I use for 192.168.1.x network. While there are a number 
devices attached to this network, for the purposes of this guide there is a FreeNas and Work Laptop.
There is a FreeNAS baremetal node that I hope to encorporate into the OpenShift Cluster in the 
future. My Work Laptop exists on the 192.168.1.x network and can route to the 10.10.10.x network as 
needed.

https://store.ui.com/collections/operator-edgemax-switches/products/edgeswitch-24-250w

For the homelab network, a Mikrotik Router Switch was used due to its ability to switch traffic at 
10 gbe speeds. Connected to this Mikrotik Router Switch are three baremetal computers which will
become hosts to the OpenShift Cluster.

(https://mikrotik.com/product/crs317_1g_16s_rm)

### Architectural Decisions

1. Traffic for the homelab should not interfere with other home network traffic.
2. Traffic within the homelab network should switch at 10 gbe speeds.
3. Use of a virtualization platform on the baremetal computers.
4. Hardware should meet or exceed minimum hardware requirements for OpenShift.

## Virtualization Platforms

There are a number of choices that were considered for a homelab virtualization platform.

A. Red Hat Virtualization (https://www.redhat.com/en/technologies/virtualization/enterprise-virtualization): Based on KVM.
B. ProxMox VE (https://www.proxmox.com/en/proxmox-ve): Also based on KVM. 
C. VMWare ESXi ( https://www.vmware.com/products/esxi-and-esx.html ): Costs money / not open source.

VMWare ESXi was evaluated and quickly reached the limit of managing virtual machines across 
multiple hosts was a feature that not included in the free version. ProxMox was the next
to be evaluated and found the overall installation and provisioning of virtual machines to 
be fairly straightforward. I also was able to cluster the three nodes on my network, making 
it easier to manage all VMs in a central location.

Red Hat Virtualization is also a viable option and I hope to evaluate in a future homelab execise.
My experience with ProxMox was so positive that I just decided to go with that platform.

### Minimum Requirements

As mentioned in the documentation for the current OpenShift release (4.10.3), the following
is the minimum hardware requirements:

* Control plane nodes: At least 4 CPU cores, 16.00 GiB RAM, 120 GB disk size for every supervisor.
* Workers: At least 2 CPU cores, 8.00 GiB RAM, 120 GB disk size for each worker
* SNO: One host is required with at least 4 CPU cores, 16.00 GiB of RAM, and 120 GB of disk size storage.
* Also note that each host's disk write speed should meet the minimum requirements to run OpenShift.

### ProxMox Configuration

There are a number of guides that can walk you through the provisioning of ProxMox VE. For a deep 
dive on how to install, the following guide is very helpful: https://forum.proxmox.com/threads/proxmox-beginner-tutorial-how-to-set-up-your-first-virtual-machine-on-a-secondary-hard-disk.59559/

Here is a simplified set of steps that I used:
* Download the latest ISO from: https://www.proxmox.com/en/downloads
* Follow the install steps to provision on bare metal.
* Configure with a static IP (10.10.10.10, 10.10.10.11, 10.10.10.12) and I choose the following 
hostnames (pve1, pve2, pve3).

Upon completing the install across all three nodes, then follow the steps in the GUI to
"Create Cluster". I found the Cluster Manager (https://pve.proxmox.com/wiki/Cluster_Manager) to be quite 
useful as all hosts were presented in a single location. Additional features such as easy
migration of virtual machines and the ability to make cluster-wide configurations are 
just some of the features that make it worth it to cluster.

## OpenShift Assisted Installer

This guide was contructed for an OpenShift Assisted Cluster install, which is best described at the 
following location. https://cloud.redhat.com/blog/how-to-use-the-openshift-assisted-installer. The 
basic premise of this approach is that you create / declare your cluster in the Red Hat Hybrid
Cloud Console ( https://console.redhat.com/openshift ). 

Upon selecting "Create cluster", then select "Datacenter" and then "Bare Metal (x86_64)". At this
point there is an option to select "Assisted Installer (Technology Preview)". In the subsequent
dialog, there is an option to name the Cluster. Just remember that you may find the need to find
your cluster in a Red Hat Hybrid Cloud Console search, so it would be advantageous to name your
cluster something unique (stay away from ocp / playground!).

In my case, I named my cluster after my hometown Oklahoma City (okc). For a base domain, I have 
a registered domain name and also have intentions to access this cluster from outside my homelab, 
so I used my registered domain. Also in this dialog, there is an option to install single node
OpenShift (SNO) or on arm64 CPU architectures. Since my homelab has multiple hosts and is x86
based, I did not select these options.

In the next dialog, the configuration of hosts is covered. Upon selecting "Add Hosts", a pop
up dialog is presented where you can download an Discovery ISO image. I leveraged the Full
Image file with the hopes that this would speed up the OpenShift provisioning process.

Upon downloading the image, it is important to make this image available to each node
in the ProxMox VE cluster. For each node, select the local disk and then the "ISO Images"
button in the configuration table. This is depicted in the following figure (../images/2022-ProxMox-ISOImages.png)

By uploading the ISO, it will then be available to the create VM process dialogs.






## ProxMox Provision


