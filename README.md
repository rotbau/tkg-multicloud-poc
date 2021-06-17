# TKG POC Guide
This guide covers some basic POC pre-requisites and general setup for Tanzu Kubernetes Grid proof of concept testing.

Assumptions:

1. TKG 1.3.1
3. Internet connectivity from jumpbox and vsphere environment
4. Perimeter firewall is not doing SSL cracking / Cert spoofing for public URLs.  If your firewall is doing this you will need to run an airgap install instead https://gist.github.com/rotbau/a90f79473326a7bd3aeb3afa05a01ab3. You can test this by running `openssl s_client -connect registry.tkg.vmware.run:443` .  This should show issuer as digicert and subject *.bintray.com.

Because we have internet connectivity from our vsphere and jumpbox environments images will be pulled from the public VMware repository at registry.tkg.vmware.run


## Quicklinks


1. [Working with Tanzu CLI](#working-with-tkg-cli)
2. [Working with the TKG Management cluster](#working-with-the-tkg-management-cluster)
3. [Creating TKG Workload clusters](#creating-tkg-workload-clusters)
4. [Working with TKG Workload clusters](#working-with-tkg-workload-clusters)
5. [Deploying test applications]()


## Documentation


This POC guide is meant to supplement the official VMware documentation.  Always consult the latest VMware documentation.

https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.3/vmware-tanzu-kubernetes-grid-13/GUID-index.html


## Jumpbox Requirements


Its best practice to have a management jumpbox to use for day to day operation of TKG.  This jumpbox is used as the intial bootstrap environment when configuring your TKG management clusters on-premise or in the cloud.  TKG creates a temporary management cluster using Kubernetes in Docker (KIND) on the jumpbox.  It will use the temporary management cluster running in KIND to deploy a permanent management cluster on the cloud infrastructure (vSphere, AWS, Azure, VMC, etc).  It also holds the configuration files for your managment clusters, the TKG cli as well as the admin kube config file.

I strongly recommend a Linux VM or MacOS for your jumpbox, however Windows can be used if that is your primary OS of choice.  Installation of the management cluster can be completed using the CLI or UI.  For first deployments it is strongly recommended to use the UI installer interface. 

- Linux, Windows or MacOS 
- Minimum 6 GB of RAM
- Minimum 2 vCPU (or 2 cores)
- 200 GB of Disk
- Desktop Environment strongly recommended for Linux VMs but not required (Gnome, Unity, KDE, MATE, etc)
- Docker CE or Docker Desktop depending on OS (Note RHEL doesn't support Docker CE) - https://docs.docker.com/engine/install/
- System time synchronized with NTP
- Git Tools - https://git-scm.com/downloads
- jq - https://stedolan.github.io/jq/download/
- Kind Binary - https://github.com/kubernetes-sigs/kind/releases
- TKG CLI - https://www.vmware.com/go/get-tkg 
    - Click **Go to Downloads**
    - Make Sure **Select Version** pull down is 1.3.x
    - Download the **VMware Tanzu Kubernetes Grid CLI** for your Jumpbox OS
    - Log in with your My VMware credentials
- Kubectl 
    - Linux `curl -LO https://dl.k8s.io/release/v1.19.0/bin/linux/amd64/kubectl`
    - MacOS `curl -LO https://dl.k8s.io/release/v1.19.0/bin/darwin/amd64/kubectl`
    - Windows `curl -LO https://dl.k8s.io/release/v1.19.0/bin/windows/amd64/kubectl.exe`
- AWS or Azure CLI if you are deploying to public cloud
- Code editing tool like Visual Studio Code or Notepad++ as examples

Download the binarys above (git, kind, tkg cli, kubectl etc) to your jumpbox and place them in a location that is in your path so they can be ls
executed from the command line.

Installing Tanzu CLI and Plugins:
```

```

## vSphere Infrastructure Prepration


- vCenter >= 6.7u3 or 7.0
- DHCP subnet presented as vCenter Portgroup (VSS/VDS) for Kubernetes Management and Workload Clusters (Workload Cluster Network)
    - Recommend a /24 for a POC
    - Set aside at least 20  (Recommend more) IPs that are part of this subnet (excluded from DHCP scope) that can be manually assigned for Kubernetes API VIP or Application VIP
    - Example (Subnet 192.168.50.0/24, DHCP Scope 192.168.50.10-192.168.50.200, Usable IPs for VIP or LB 192.168.50.201-254)
- Create Resource Group in vSphere Cluster for TKG VMs - leave settings at default
- Create VM Folder for TKG VMs and Templates 
- Download Photon v3 or Ubuntu Kubernetes for the version of tanzu we are installing (ex 1.3.1 using v1.20.5) - https://www.vmware.com/go/get-tkg
```
Kubernetes v1.20.5: Ubuntu v20.04 Kubernetes v1.20.5 OVA
Kubernetes v1.20.5: Photon v3 Kubernetes v1.20.5 OVA
```
- Optional: Download v1.18.x and v1.17.x OVAs if you want different K8s versions
- Import OVA into vCenter - leave name as is
- Convert VM to Template
- Traffic allowed out to vCenter Server 443 from the Workload Cluster network
- Traffic allowed on port TCP 6443 from local bootstrap (jumpbox) and Workload Cluster network
- Time in sync on all ESXi hosts
- Generate SSH Key to use for TKG nodes `ssh-keygen -t rsa -b 4096 -C "email@example.com"`


## AWS EC2 Infrastructure Perparation


- Access key and access key secret for an active AWS account
- Your AWS account must have Administrator privileges
- AWS Account with suffcient resources for:
    - Elastic IP (EIP) addresses. The default EIP quota is 5 EIP addresses per region, per account.
    - Management cluster deployment creates one VPC and one or three NAT gateways. The default NAT gateway quota is 5 instances per availability zone, per account
- AWS CLI installed locally on  Jumpbox
- Traffic allowed on port TCP 6443 from local bootstrap (jumpbox) and AWS
- Traffic allowed on port TCP 443 from local bootstrap and VMare registry (registry.tkg.vmware.run)
- Generate SSH Key to use for TKG nodes `ssh-keygen -t rsa -b 4096 -C "email@example.com"`

## Azure Infrastructure Requirements


- Microsoft Azure account
    - permissions to deploy and app
    - Sufficient VM core quotas for clusters.  Standard Azure account has a 10 vCPU quota per region.  TKG clusters require 2 vCPU per node.  Examples of sizing for deployments
        - Management Cluster **Prod Plan** 8 vCPU for 3 control plane nodes and 1 worker node.  **Dev Plan** 4 vCPU for 1 control plane node and 1 worker node.
        - Workload Cluster **Prod Plane** 12 vCPU (3 control, 3 worker).  **Dev Plan** 4 vCPU (1 control, 1 worker)
- Traffic allowed on port TCP 6443 from local bootstrap (jumpbox) and Azure
- Traffic allowed on port TCP 443 from local bootstrap and VMare registry (registry.tkg.vmware.run)
- VNET will be created or optionally you can choose existing provided there is suffcient capacity
- Generate SSH Key to use for TKG nodes `ssh-keygen -t rsa -b 4096 -C "email@example.com"`

## VMC on AWS or Azure VMware Solution

- Jumpbox is required to be located in VMC or AVS environment.  Prepare using same requirements as listed [Jumpbox Requirements](#jumpbox-requirements) section.
- Follow official documentation for additional preparation steps https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.3/vmware-tanzu-kubernetes-grid-13/GUID-mgmt-clusters-prepare-maas.html


## Management Cluster Installation


Follow official documentation [linked here](#documentation)

Example using UI:
`tanzu management-cluster create my-mgmt-cluster --ui`

Headless UI Install:
`tanzu management-cluster create my-mgmt-cluster --ui --bind [jumpbox IP when command is run]:8181 --browser none`


## Working with Tanzu CLI


From your jumpbox issue `tanzu` from the command line.  This will show you all of the commands available from the TKG CLI.

Reference Documentation: https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.3/vmware-tanzu-kubernetes-grid-13/GUID-tanzu-cli-reference.html

## Working with the TKG Management cluster

After you install the management cluster(s) for the desired environments.  You can use the Tanzu cli to view TKG management clusters and create and view TKG workload clusters. 

All comands executed from jumpbox where tanzu cli is installed

### View and Select TKG management clusters using TKG cli
- View TKG management clusters
    `tkg get mc` to view management clusters
    You will see all TKG management clusters you've installed from this jumpbox.  The management cluster with the asterisk is the currently set cluster.

    ![alt text](/assets/tkg-get-mc.png)

- Set TKG mangement cluster 
    `tkg set mc tkg-mgmt` will set the mc context for the TKG cli.  Any further TKG commands you run will be executed by the selected TKG management cluster.

    ![alt text](/assets/tkg-set-mc.png)

- View TKG workload clusters managed by selected TKG management cluster
    `tkg get clusters` or `tkg get clusters --include-management-cluster`

    ![alt text](/assets/tkg-set-mc.png)

### Accessing TKG managemnt cluster nodes using kubectl

By default during installation of the Management cluster, the credentials are added to the jumpbox kubeconfig file (~/.kube/config).  You can access the management cluster nodes using kubectl.

- Set kubectl cli context
    `kubectl config use-context [mgmt-cluster-name]`

example for tkg-mgmt management cluster name

    `kubectl config use-context tkg-mgmt-admin@tkg-mgmt`

    ![alt text](/assets/kubectl-config.png)

- View nodes
    `kubectl get nodes`

    ![alt text](/assets/kubectl-get-nodes.png)

- View namespaces
    `kubectl get ns`

    ![alt text](/assets/kubectl-get-ns.png)

- View all pods
    `kubectl get pods -A`

- View pods in a specific namespace
    `kubectl get pods -n kube-system`

## Creating TKG Workload clusters

Once you have your TKG management cluster created you can create TKG workload clusters for your applications.  You can leverage the TKG cli to create clusters.

- Set TKG management cluster if you have multiple management clusters.  Ignore if you only have a single TKG management cluster
- Create TKG cluster from dev plan (single control plane node and single worker node)

    `tkg create cluster test-cluster -p dev --vsphere-controlplane-endpoint 172.31.3.80`

    *note the controlplane IP is an IP address from the same dhcp network that your nodes are deployed on but outside the dhcp scope.  This IP is used by kube-vip to provide a reliable IP to the workload clusters kubernetes API*

    ![alt text](/assets/tkg-create-cluster.png)

- Create a TKG cluster from prod plan with custom node size and number of worker nodes (3 control plane nodes and workers based on command line input)

    `tkg create cluster test-cluster -p prod --controlplane-size large -w 10 --worker-size extra-large --vsphere-control-endpoint 172.31.3.80`

- Pre-configured node sizes
    - small = Cpus: 2, Memory: 2048, Disk: 20
    - medium = Cpus: 2, Memory: 4096, Disk: 40
    - large = Cpus: 2, Memory: 8192, Disk: 40
    - extra-large = Cpus: 4, Memory: 16384, Disk: 80

### Generating Cluster YAML for use with CICD or Kubectl

In addition to using the TKG cli to create the cluster, you can also use the --config string to create the YAML for cluster creation and then use it in a CIDC system or pipeline instead of the CLI

   `tkg config create cluster test-cluster -p prod --controlplane-size large -w 10 --worker-size extra-large --vsphere-control-endpoint 172.31.3.80 > test-cluster.yaml`

You can then use a CICD system or kubectl to deploy the cluster
- Set you kubectl context the TKG management cluster in the desired environment `kubectl config use-context tkg-mgmt`
- Apply cluster yaml `kubectl apply -f test-cluster.yaml`

Note: you will still be able to view, scale and upgrade clusters using the TKG cli when they are created using kubectl or CICD tools.


## Working with TKG Workload clusters

### Exporting TKG workload cluster credentials

1. Get credentials to workload cluster for the first time.  You can either have the credentials merged into the default kubeconfig file in ~/.kube/config or have it exported to a separate file.

**Merge credentials to existing ~/.kube/config**

`tkg get credentials test-cluster`

**Export credentials to separate kubeconfig file**

`tkg get credentials test-cluster --kubeconfig tkgkubeconfig`  
**Note:** if you export to separate file you need to specify the kubeconfig file on all kubectl commands `kubecctl --kubeconfig tkgkubeconfig {command}`

### Accessing TKG workload cluster using kubectl

1. Set kubectl context to workload cluster
`kubectl config set-context test-cluster`
2. View nodes
`kubectl get nodes`
3. View namespaces
`kubectl get ns`
4. View Pods
`kubectl get pods` or `kubectl get pods -A` or `kubectl get pods -n {namespace}`
