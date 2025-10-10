# Upgrade the Cluster

<u>First you need to upgrade the control plane node, then the worker nodes.</u>
![cluster_upgrade_overview](cluster_upgrade_overview.png)

## 1. Upgrade the Control Plane Node

![control_plane_node_upgrade](control_plane_node_upgrade.png)

### Order:

1. **Upgrade kubeadm** 	`sudo apt install kubeadm=<new-version>`	Updates the utility responsible for  the upgrade process.
2. **Apply Upgrade**  `sudo kubeadm upgrade apply <new-version>`	Renews certificates and updates the static Pod manifests (API Server, Scheduler, etc.).
3. **Drain Node**  `kubectl drain c1-cp1 --ignore-daemonsets`	Now drain the node to evict any non-control plane workloads before touching the kubelet.
4. **Upgrade Binaries**  `sudo apt install kubelet=<new-version> kubectl=<new-version>`	Updates the local kubelet agent and the kubectl utility.
5. **Restart Kubelet**  `sudo systemctl daemon-reload && sudo systemctl restart kubelet`	Restarts the agent to load the newly upgraded binary.
6. **Uncordon Node**  `kubectl uncordon c1-cp1`	Marks the Control Plane as available for scheduling.

**Upgrade kubeadm**

Find the version you want to upgrade the `kubeadm` to and upgrade it.

```bash
sudo apt-get update && apt-cache policy kubeadm
TARGET_VERSION=<VERSION_STRING>

sudo apt-mark unhold kubeadm && sudo apt-get install -y kubdeadm=$TARGET_VERSION
sudo apt-mark hold kubeadm
```

**Verify the upgrade plan**

Run the upgrade plan to see what will be upgraded. and check the pre-flight checks.

```bash
sudo kubeadm upgrade plan
```

**Run the upgrade**

1. It runs pre-flight checks - API availability, Node status Ready and control plane health.
2. Checks to ensure you are upgrading along the correct upgrade path. 
3. Prepulls container images to reduce downtime of control plane components.
    3.1 Updates the certificates used for authentication
    3.2 Creates a new static pod manifest in `/etc/kubernetes/manifests` and saves the old one to `/etc/kubernetes/tmp`
4. Updates the Control Plane Node's kubelet configuration and also updates CoreDNS and kube-proxy.

```bash
sudo kubeadm upgrade apply $TARGET_VERSION
```

**Drain any workload from the Control Plane Node**

`kubectl drain <control-plane-node-name> --ignore-daemonsets`

**Upgrade kubelet and kubectl**

```bash
sudo apt-mark unhold kubelet kubectl && sudo apt-get install -y kubelet=$TARGET_VERSION kubectl=$TARGET_VERSION
sudo apt-mark hold kubelet kubectl
```

**Reload and restart the systemd unit**

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
sudo systemctl status kubelet
```

**Uncordon the Control Plane Node**
This will allow scheduling of workloads on the control plane node.

`kubectl uncordon <control-plane-node-name>`

---

## 2. Upgrade the Worker Nodes

![worker_node_upgrade](worker_node_upgrade.png)

1. **Upgrade kubeadm**		`sudo apt install kubeadm=<new-version>`	Updates the utility responsible for the configuration changes.
2. **Drain Node**		`kubectl drain c1-node1 --ignore-daemonsets`	First, safely evict all application Pods to prevent disruption.
3. **Upgrade Config**		`sudo kubeadm upgrade node`	Downloads and applies the new Kubelet configuration from the cluster.
4. **Upgrade Binaries**		`sudo apt install kubelet=<new-version> kubectl=<new-version>`	Updates the local kubelet agent and the kubectl utility.
5. **Restart Kubelet**		`sudo systemctl daemon-reload && sudo systemctl restart kubelet`	Restarts the agent to load the upgraded binary and config.
6. **Uncordon Node**		`kubectl uncordon c1-node1`	Marks the worker node as available for scheduling new workloads.

**Drain the Worker Node**

This will evict all the workloads from the worker node and mark it unschedulable.
`kubectl drain <worker-node-name> --ignore-daemonsets`

**Upgrade kubeadm**

```bash
sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm=$TARGET_VERSION
sudo apt-mark hold kubeadm
```

**Run the upgrade**

`sudo kubeadm upgrade node`
This will update the kubelet configuration and also update the kube-proxy DaemonSet.

**Upgrade kubelet and kubectl**

```bash
sudo apt-mark unhold kubelet kubectl && sudo apt-get install -y kubelet=$TARGET_VERSION kubectl=$TARGET_VERSION
sudo apt-mark hold kubelet kubectl
```

**Reload and restart the systemd unit**

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
sudo systemctl status kubelet
```

**Uncordon the Worker Node**

This will allow scheduling of workloads on the worker node.

`kubectl uncordon <worker-node-name>`

---

## Key Difference:

**Control Plane:** You apply the upgrade first (kubeadm upgrade apply) because you need the cluster's brain to be upgraded before anything else.

**Worker Node:** You drain the node first (kubectl drain) because your primary concern is minimizing application downtime.

---

## 3. Issues I faced

- **CNI plugin (Calico)** interfering with the host's DNS resolution setup after the **drained** the worker node. 
  - It caused issues with `kubeadm upgrade node` command as it couldn't resolve the API server's DNS name.
  - **Solution**: I had to temporarily edit the `/etc/resolv.conf` file to use a public DNS server like `nameserver 8.8.8.8`.
    - I upgraded the worker node successfully and the packages (kubelet, kubectl)then a reboot restored the original `/etc/resolv.conf` file.