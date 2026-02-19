## 🛠️ The Control Plane Cheatsheet

1.  **The "Static Pods" Manifests**
    If you need to fix a broken API Server, Scheduler, or Controller Manager, this is the most important folder in the exam.

    **Path:** `/etc/kubernetes/manifests/`

    **What's inside:** YAML files for `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, and `etcd`.

    **Pro-Tip:** If you move a file out of this folder, the pod stops. Move it back, and Kubelet restarts it automatically.

2.  **Certificates & Keys**
    When you get a "Certificate Expired" or "Connection Refused" error between components.

    **Path:** `/etc/kubernetes/pki/`

    **What's inside:** `ca.crt`, `apiserver.crt`, and the `etcd/` subfolder.

    **Pro-Tip:** Use `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout` to check expiration dates.

3.  **Kubeconfig Files (Authentication)**
    These files tell components how to talk to the API.

    **Path:** `/etc/kubernetes/`

    **What's inside:** `admin.conf`, `kubelet.conf`, `controller-manager.conf`, and `scheduler.conf`.

---

## 🏗️ The Worker Node Cheatsheet

4.  **Kubelet Configuration**
    If the node is `NotReady`, check here.

    **Config Path:** `/var/lib/kubelet/config.yaml`

    **Service Path:** `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`

    **Pro-Tip:** If you edit the `config.yaml`, you must run `systemctl restart kubelet`.

5.  **CNI Networking (Plugins & Config)**
    **Binary Path:** `/opt/cni/bin/` (Where the flannel, calico, or bridge executables live).

    **Config Path:** `/etc/cni/net.d/` (The JSON files that define how the network behaves).

---

## 📦 Data & Runtime Cheatsheet

6.  **The "Brain" (ETCD Data)**
    **Path:** `/var/lib/etcd/`

    **Pro-Tip:** When doing an ETCD Backup, this is the directory you are backing up.

7.  **Container Secrets (Inside Pods)**
    The path you just used for your curl challenge.

    **Path:** `/var/run/secrets/kubernetes.io/serviceaccount/`

    **What's inside:** `token`, `ca.crt`, and `namespace`.

8.  **Logs**
    **System Logs:** `journalctl -u kubelet`

    **Container Logs:** `/var/log/pods/` or `/var/log/containers/`

---

Here's a quick reference for common issues and where to look:

*   **If you need to fix...** API Server not starting, **Go to...** `/etc/kubernetes/manifests/`
*   **If you need to fix...** Node joining issues, **Go to...** `/etc/kubernetes/kubelet.conf`
*   **If you need to fix...** ETCD Snapshots (for backup/restore), **Go to...** `/var/lib/etcd/`
*   **If you need to fix...** Expired certs, **Go to...** `/etc/kubernetes/pki/`
*   **If you need to fix...** Static Pod path (to verify its location), **Go to...** Check `staticPodPath` in `/var/lib/kubelet/config.yaml`


---

🏁 **Final Exam Tip: The "Find" Command**
If you are in the exam and forget a path, use this:

```bash
find /etc -name "*kubelet*"

