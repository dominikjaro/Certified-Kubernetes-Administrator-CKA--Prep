## Building Your Cluster

1. Install and configure packages required for Kubernetes (containerd, kubeadm, kubelet, kubectl)
2. Create your cluster
3. Configure Pod networking
4. Join nodes to your cluster

`kubelet` is a thing that will drive the work on each node and in our cluster.
`kubeadm` is respionsible for creating and managing the cluster up/running and configured. It also helps with upgrades and joining nodes.
`kubectl` is the primary CLI tool to interact with your cluster.
`containerd` pulls images from registries. Stores and organizes them on the server. It starts, stops and deletes containers.

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
        |   IP: 10.0.0.9      |
        +---------------------+
                   |
                   |
        +---------------------+
        |     Worker Node 1   |
        |   (kubeadm join)    |
        |.   Hostname: c1-node1 |
        |   IP: 10.0.0.10     | 
        +---------------------+
```

## Prerequisites and Oracle Setup

**Oracle:**

1. I created a Virtual Cloud Network with a public subnet in the London region. I added in Internet Gateway and the Default Route Table.
2. Also, I needed a security List allowing SSH (22) and all traffic and Network Security Group allowing SSH (22) and all traffic.
3. I created 2 VMs in the public subnet using the Ubuntu 24.04 image.

``` bash
# Disabling `swap` ensures that the kubelet has a clear, real-time view of a node's memory resources, leading to more predictable performance and more reliable scheduling.
# Disable swap with swapoff -a
# Permanently disable swap by editing /etc/fstab and commenting out any swap entries.
# Check if it is disabled
sudo swapoff -a
vi /etc/fstab
free -h
```

Configure the containerd prerequisite, and load two modules and configure them to load on boot.

`overlay`: is a filesystem driver that allows different layers of a container image to be stacked on top of each other efficiently. This is how container runtimes like `containerd` manage images.
`br_netfilter`: is required to enable communication between pods by allowing `iptables` rules to filter and route bridge network traffic. Without it your pods wouldn't be able to talk to each other. 

![overlay_and_br_netfilter.png](overlay_and_br_netfilter.png)

```bash
# By creating the file and adding the kernel modules, we ensure that they load automatically on system boot, which is essential for the proper functioning of containerd and Kubernetes networking.
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
overlay
EOF

# Load the modules for the current session, so no need to reboot the server
sudo modprobe br_netfilter
sudo modprobe overlay
```

Need the `sysctl` params to enable networking routing for the Kubernetes cluster.

`net.bridge.*`: commands needed to enable `iptables` rules to filter and route bridge network traffic. In a Kubernetes cluster, pods communicate over a virtual network bridge (e.g. cbr0). Wihtout these settings, `iptables` rules, which are used for Kubernetes network policies and service routing, would not function correctly.

The `net.ipv4.ip_forward` setting is crucial for enabling IP forwarding on the node. This allows the node to route packets between different network interfaces, which is essential for pod-to-pod communication across nodes in a Kubernetes cluster.

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Load them into memory by running:
sudo sysctl --system
```

---

**Install Packages:**

1. containerd
2. kubelet
3. kubeadm
4. kubectl

Install `containerd`...
`sudo apt-get update && sudo apt-get install -y containerd`

Create the default configuration file for `containerd`... - to make sure container runtime functional out of the box. 

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

