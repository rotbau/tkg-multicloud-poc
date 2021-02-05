# TKG POC Guide
This guide covers some basic POC pre-requisites and general setup for Tanzu Kubernetes Grid proof of concept testing.

Assumptions:

1. TKG 1.2+
3. Internet connectivity from jumpbox and vsphere environment
4. Perimeter firewall is not doing SSL cracking / Cert spoofing for public URLs.  If your firewall is doing this you will need to run an airgap install instead https://gist.github.com/rotbau/a90f79473326a7bd3aeb3afa05a01ab3

Because we have internet connectivity from our vsphere and jumpbox environments images will be pulled from the public VMware repository at registry.tkg.vmware.run

## Documentation

This POC guide is meant to supplement the official VMware documentation.  Always consult the latest VMware documentation.

https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.2/vmware-tanzu-kubernetes-grid-12/GUID-index.html

## Jumpbox Requirements

Its best practice to have a management jumpbox to use for day to day operation of TKG.  This jumpbox is used as the intial bootstrap environment when configuring your TKG management clusters on-premise or in the cloud.  TKG creates a temporary management cluster using Kubernetes in Docker (KIND) on the jumpbox.  It will use the temporary management cluster running in KIND to deploy a permanent management cluster on the cloud infrastructure (vSphere, AWS, Azure, VMC, etc).  It also holds the configuration files for your managment clusters, the TKG cli as well as the admin kube config file.

I strongly recommend a Linux VM or MacOS for your jumpbox, however Windows can be used if that is your primary OS of choice.  Installation of the management cluster can be completed using the CLI or UI.  For first deployments it is strongly recommended to use the UI installer interface. 

- Linux, Windows or MacOS 
- Minimum 6 GB of RAM
- Minimum 2 vCPU (or 2 cores)
- 200 GB of Disk
- Desktop Environment recommended for Linux VMs but not required (Gnome, Unity, KDE, MATE, etc)
- Docker CE or Docker Desktop depending on OS (Note RHEL doesn't support Docker CE) - https://docs.docker.com/engine/install/
- System time synchronized with NTP
- Git Tools - https://git-scm.com/downloads
- jq - https://stedolan.github.io/jq/download/
- Kind Binary - https://github.com/kubernetes-sigs/kind/releases
- TKG CLI - https://www.vmware.com/go/get-tkg 
    - Click **Go to Downloads**
    - Make Sure **Select Version** pull down is 1.2.1
    - Download the **VMware Tanzu Kubernetes Grid CLI** for your Jumpbox OS
    - Log in with your My VMware credentials
- Kubectl 
    - Linux `curl -LO https://dl.k8s.io/release/v1.19.0/bin/linux/amd64/kubectl`
    - MacOS `curl -LO https://dl.k8s.io/release/v1.19.0/bin/darwin/amd64/kubectl`
    - Windows `curl -LO https://dl.k8s.io/release/v1.19.0/bin/windows/amd64/kubectl.exe`
- AWS or Azure CLI if you are deploying to public cloud

Download the binarys above (git, kind, tkg cli, kubectl etc) to your jumpbox and place them in a location that is in your path so they can be ls
executed from the command line.

Example Commands:
```
gunzip tkg-linux-amd64-v1.2.1-vmware.1.tar.gz && tar -xvf tkg-linux-amd64-v1.2.1-vmware.1.tar
cd tkg
chmod +x *
sudo mv tkg* /usr/local/share/tkg
sudo mv kbld* /usr/local/share/kbld
sudo mv kapp* /usr/local/share/kapp
sudo mv imgpkg* /usr/local/share/imgpkg
sudo mv ytt* /usr/local/share/ytt
```
## vSphere Infrastructure Prepration

- vCenter >= 6.7u3 or 7.0
- DHCP subnet presented as vCenter Portgroup (VSS/VDS) for Kubernetes Management and Workload Clusters
    - Recommend at least a /26
    - Set aside at least 10 IPs that are part of this subnet (excluded from DHCP scope) that can be manually assigned for Kubernetes API VIP or Application VIP
    - Example (Subnet 192.168.50.0/24, DHCP Scope 192.168.50.10-192.168.50.200, Usable IPs for VIP or LB 192.168.50.201-254)
- Create Resource Group in vSphere Cluster for TKG VMs - leave settings at default
- Create VM Folder for TKG VMs and Templates 
- Download Photon v3 Kubernetes v1.19.3 OVA - https://www.vmware.com/go/get-tkg
- Optional: Download v1.18.x and v1.17.x OVAs if you want different K8s versions
- Import OVA into vCenter - leave name as is
- Convert VM to Template

## AWS EC2 Infrastructure Perparation

- Access key and access key secret for an active AWS account
- Your AWS account must have Administrator privileges
- AWS Account with suffcient resources for:
    - Elastic IP (EIP) addresses. The default EIP quota is 5 EIP addresses per region, per account.
    - Management cluster deployment creates one VPC and one or three NAT gateways. The default NAT gateway quota is 5 instances per availability zone, per account
- AWS CLI installed locally on  Jumpbox
- Traffic allowed on port TCP 6443 from local bootstrap (jumpbox) and AWS
- Traffic allowed on port TCP 443 from local bootstrap and VMare registry (registry.tkg.vmware.run)

## Azure Infrastructure Requirements

- Microsoft Azure account
    - permissions to deploy and app
    - Sufficient VM core quotas for clusters.  Standard Azure account has a 10 vCPU quota per region.  TKG clusters require 2 vCPU per node.  Examples of sizing for deployments
        - Management Cluster **Prod Plan** 8 vCPU for 3 control plane nodes and 1 worker node.  **Dev Plan** 4 vCPU for 1 control plane node and 1 worker node.
        - Workload Cluster **Prod Plane** 12 vCPU (3 control, 3 worker).  **Dev Plan** 4 vCPU (1 control, 1 worker)
- Traffic allowed on port TCP 6443 from local bootstrap (jumpbox) and Azure
- Traffic allowed on port TCP 443 from local bootstrap and VMare registry (registry.tkg.vmware.run)
- VNET will be created or optionally you can choose existing provided there is suffcient capacity

## VMC on AWS or Azure VMware Solution

- Jumpbox is required to be located in VMC or AVS environment.  Prepare using same requirements as listed [Jumpbox Requirements](#jumpbox-requirements) section.
- Follow official documentation for additional preparation steps https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.2/vmware-tanzu-kubernetes-grid-12/GUID-prepare-maas.html#prep-vmc

## Management Cluster Installation

Follow official documentation [linked here](#documentation)

## Working with the TKG Management cluster

After you install the management cluster(s) for the desired environments.  You can use the TKG CLI to view TKG management clusters and create and view TKG workload clusters. All comand executed from jumpbox  

- View TKG management clusters
    `tkg get mc` to view management clusters
    You will see all TKG management clusters you've installed from this jumpbox

- Select TKG mangement cluster
    `tkg set mc tkg-mgmt` will set the mc context for the TKG cli.  Any further TKG commands you run will be executed by the selected TKG management cluster

- View TKG workload clusters managed by selected TKG management cluster
    `tkg get clusters` or `tkg get clusters --include-management-cluster`
