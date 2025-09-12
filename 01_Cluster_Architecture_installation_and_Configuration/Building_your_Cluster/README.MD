## Building Your Cluster

1. Install and configure packages required for Kubernetes (containerd, kubeadm, kubelet, kubectl)
2. Create your cluster
3. Configure Pod networking
4. Join nodes to your cluster

`kubelet` is a thing that will drive the work on each node and in our cluster.
`kubeadm` is respionsible for creating and managing the cluster up/running and configured. It also helps with upgrades and joining nodes.
`kubectl` is the primary CLI tool to interact with your cluster.

## Topology

For this course, we will be using a simple topology with 1 control plane node and 2 worker nodes.
We are using Ubuntu 22.04 VMs with 1 vCPU, 1GB RAM and 20GB disk each.
Swap is disabled on all nodes. -- Swap space is a designated area on a HDD/SSD that an operating system uses as virtual memory. This can interfere with kubelet operations.
Hostnames and IPs are set and host file on each node.
```
        +---------------------+
        |   Control Plane      |
        |   (kubeadm init).    |
        |.  Hostname: c1-cp1   |
        |   IP: 172.16.94.10   |
        +---------------------+
                   |
                   |
        +---------------------+
        |     Worker Node 1   |
        |   (kubeadm join)    |
        |.   Hostname: c1-node1 |
        |   IP: 172.16.94.11   |
        +---------------------+
                   |
                   |
        +---------------------+
        |     Worker Node 2   |
        |   (kubeadm join)    |
        |.   Hostname: c1-node2 |
        |   IP: 172.16.94.12  |
        +---------------------+
        |   IP:                |
        +---------------------+
```

## Prerequisites and Oracle Setup

1. I created a Virtual Cloud Network with a public subnet in the London region. I added in Internet Gateway and the Default Route Table. 
2. I created 3 VMs in the public subnet using the Ubuntu 22.04 image. I added in a new SSH key pair and downloaded the private key. For storage I used the default 50GB boot volume.

``` bash
# Disable swap with swapoff -a
# Check if it is disabled
sudo swapoff -a
vi /etc/fstab

```