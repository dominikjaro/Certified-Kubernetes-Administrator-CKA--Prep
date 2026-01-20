## What is `containerd`?

1. Kubelet (Kubernetes agent on the node) interacts with `containerd` via Container Runtime Interface (CRI)
2. `containerd` pulls container images, creates containers and manages their lifecycle
3. Containers are executed using `runc`

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
