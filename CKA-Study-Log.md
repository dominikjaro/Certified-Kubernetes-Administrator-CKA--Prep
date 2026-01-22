## What is `containerd`?

1. Kubelet (Kubernetes agent on the node) interacts with `containerd` via Container Runtime Interface (CRI)
2. `containerd` pulls container images, creates containers and manages their lifecycle
3. Containers are executed using `runc`

---

## The `kubctl` is broken - what to do?

1. The Golden Rule: When to use which?
    - `journalctl` - **The System Service** (The Kubelet) -- The Node is "NotReady" or kubectl won't connect at all. You need to check if the Kubelet is alive.
    - `crictl` - **The Containers** (Docker replacement) -- The Kubelet is fine, but a specific Static Pod (like Etcd or API Server) keeps crashing and you can't use `kubectl logs`.

---

2. How to use `journalctl` (The Node Doctor)

Use this to fix the **Kubelet**. In a kubeadm cluster, the Kubelet is a native Systemd service, not a container.

Core Commands:

```bash
systemctl status kubelet # Check if Kubelet is running
journalctl -u kubelet -n 20 # View thr last 20 lines of Kubelet logs

# Look for: "config.yaml not found", "certificate invalid", "executable not found".
```

---

3. How to use `crictl` (The Container Detective)

Use this to debug **Static Pods** (API Server, Etcd, Scheduler) when kubectl logs is unavailable.

Core Commands:

```bash
sudo crictl ps -a | grep apiserver # List all containers and filter for API Server
sudo crictl logs <container_id> # View logs of the specific container
sudo crictl inspect <container_id> | grep args -A 5 # Inspect container details and view args
```

---

## Backup and Restore `etcd` Data

Assume you are on the etcd server.

1. Back up the etcd data:

```bash
ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd_backup.db \
--endpoints=https://etcd1:2379 \      
# The above -- endpoint can be found in the etcd static pod manifest /etc/kubernetes/manifests/etcd.yaml 
--cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \ 
--cert=/home/cloud_user/etcd-certs/etcd-server.crt \
--key=/home/cloud_user/etcd-certs/etcd-server.key
```

2. Restore the etcd Data from the Backup
```bash
# 1. Stop the etcd server
sudo systemctl stop etcd

# 2. Delete the existing data directory
sudo rm -rf /var/lib/etcd

# 3. Restore from the snapshot
sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd_backup.db \
--initial-cluster etcd-restore=https://etcd1:2380 \
# The above --  tells the new etcd instance exactly who is in this new cluster."This new cluster has one member. Its name is etcd-restore, and its address is https://etcd1:2380."
--initial-advertise-peer-urls https://etcd1:2380 \
# The above -- tells the other members (if any) how to contact this specific node. "If anyone wants to talk to me for data replication (Peering), call me at etcd1 on port 2380."
--name etcd-restore \
# The above -- assigns a human-readable name to this specific node.
--data-dir /var/lib/etcd
```

**NOTE:** 

Port 2379: The "Customer Service" Door (Backup)
 - Who uses it? You, kubectl, the API Server, and etcdctl.
 - Purpose: To read or write data (like taking a snapshot).

Port 2380: The "Staff Only" Door (Restore/Clustering)
 - Who uses it? Only other etcd members (peers).
 - Purpose: To sync data between nodes (replication) and check heartbeats.