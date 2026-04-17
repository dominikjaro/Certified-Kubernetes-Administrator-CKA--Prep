# 🚀 CKA Troubleshooting & Operations Cheatsheet

## Table of Contents

1. [The Control Plane & Worker Nodes](#1-the-control-plane--worker-nodes)
2. [The Tool Belt: journalctl vs crictl](#2-the-tool-belt-journalctl-vs-crictl)
3. [Backup and Restore etcd Data](#3-backup-and-restore-etcd-data)
4. [Accessing the API from Inside a Pod](#4-accessing-the-api-from-inside-a-pod)
5. [DNS & CoreDNS Troubleshooting](#5-dns--coredns-troubleshooting)
6. [CNI (Container Network Interface)](#6-cni-container-network-interface)
7. [containerd (The Runtime)](#7-containerd-the-runtime)
8. [Copying Files Between Nodes and Pods](#8-copying-files-between-nodes-and-pods)
9. [Finding & Formatting Output Programmatically](#9-finding--formatting-output-programmatically)
10. [Docker vs. Kubernetes Command Mapping](#10-docker-vs-kubernetes-command-mapping)
11. [K8s Connectivity Cheat Sheet](#11-k8s-connectivity-cheat-sheet)
12. [Kustomize](#12-kustomize)
13. [CustomResourceDefinitions (CRDs)](#13-customresourcedefinitions-crds)

---

## 1. The Control Plane & Worker Nodes

| Component | Path | What to Look For / Action |
|---|---|---|
| Static Pods | `/etc/kubernetes/manifests/` | Typos in image, wrong command flags. Moving a file out and back in restarts the pod. |
| Certificates | `/etc/kubernetes/pki/` | Expired certs. Check with: `openssl x509 -in <file> -text -noout` |
| Kubeconfigs | `/etc/kubernetes/*.conf` | Missing or corrupted auth files (admin, kubelet, scheduler). |
| Kubelet Config | `/var/lib/kubelet/config.yaml` | `staticPodPath` and DNS settings. Must run `systemctl restart kubelet` if edited. |
| CNI (Network) | `/etc/cni/net.d/` | Missing or broken JSON network configs causing `NotReady` nodes. |
| ETCD Data | `/var/lib/etcd/` | The actual database files. This is the directory you back up/restore. |

---

## 2. The Tool Belt: journalctl vs crictl

> **The Golden Rule:** Is the problem with the **System (Node)** or the **Container (Pod)?**

### A. `journalctl` — The Node Doctor

Use this when the Node is `NotReady`, `kubectl` won't connect, or the Kubelet itself is crashing.

```bash
systemctl status kubelet              # Check if running
journalctl -u kubelet -n 20           # View last 20 lines
```

Look for: `"config.yaml not found"`, `"certificate invalid"`, `"executable not found"`.

### B. `crictl` — The Container Detective

Use this to debug Static Pods (API Server, Etcd) when the API is down and `kubectl logs` is impossible.

```bash
crictl ps -a | grep apiserver                      # Find the container ID
crictl logs <container_id>                         # Read why it crashed
crictl inspect <container_id> | grep args -A 5    # Check the exact startup command
```

---

## 3. Backup and Restore etcd Data

### Key Ports

| Port | Name | Purpose |
|---|---|---|
| `2379` | Client API | The "Customer Service" door. Used to read/write data and take backups. |
| `2380` | Peer API | The "Staff Only" door. Used only by other etcd nodes for cluster replication. |

### Part 1: Backup (Snapshot)

> Grab `--endpoints` and cert paths from `/etc/kubernetes/manifests/etcd.yaml`.

```bash
ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd_backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### Part 2: Restore

```bash
# 1. Stop the API Server and Etcd
#    If running as static pods: temporarily move them out of /etc/kubernetes/manifests/
#    If running as a systemd service:
sudo systemctl stop etcd

# 2. Delete the existing corrupted data
sudo rm -rf /var/lib/etcd

# 3. Restore from the snapshot
sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd_backup.db \
  --initial-cluster etcd-restore=https://127.0.0.1:2380 \
  --initial-advertise-peer-urls https://127.0.0.1:2380 \
  --name etcd-restore \
  --data-dir /var/lib/etcd

# 4. Restart Etcd (move static pod YAML back, or start systemd service)
```

---

## 4. Accessing the API from Inside a Pod

If you need to query the Kubernetes API from inside a pod (without `kubectl`):

```bash
# 1. Get the service account token
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

# 2. Query the API securely
curl -k https://kubernetes.default/api/v1/namespaces/default/secrets \
  -H "Authorization: Bearer $TOKEN"
```

---

## 5. DNS & CoreDNS Troubleshooting

> If services cannot talk to each other by name, the problem is usually here.

### A. Check CoreDNS Pods (The Cluster "Phonebook")

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns    # Check status
kubectl logs -n kube-system -l k8s-app=kube-dns        # Check logs
```

> **Pro-Tip:** If CoreDNS pods are `Pending`, your CNI (network plugin) is probably broken.

### B. Check the Broken Pod's DNS Config

```bash
kubectl exec -it <pod> -- cat /etc/resolv.conf           # Check DNS config
kubectl exec -it <pod> -- nslookup kubernetes.default    # Test resolution
```

> **What to look for:** You should see `nameserver 10.96.0.10` (your cluster's DNS IP). If it says `8.8.8.8`, the pod is bypassing cluster DNS!

---

## 6. CNI (Container Network Interface)

The CNI (Flannel, Calico, Weave, etc.) gives pods their IP addresses.

### Symptoms of a Broken CNI

- Nodes stuck in `NotReady`
- CoreDNS pods stuck in `Pending`
- Pods show `NetworkPluginNotReady` or stuck in `ContainerCreating`

### Where the CNI Lives

| Location | Purpose |
|---|---|
| `/etc/cni/net.d/` | Config directory. If completely empty, the CNI was never installed. |
| `/opt/cni/bin/` | CNI binary plugins. |

### Fix (Exam)

```bash
kubectl apply -f <URL_PROVIDED_IN_QUESTION>
```

---

## 7. containerd (The Runtime)

`containerd` is the engine that pulls images and runs containers. The Kubelet is the manager that talks to it.

### A. Troubleshoot the Engine

```bash
systemctl status containerd      # Check if running
systemctl restart containerd     # Restart the engine
journalctl -u containerd         # View logs
```

### B. Configuration & Socket

| Item | Path |
|---|---|
| containerd config | `/etc/containerd/config.toml` |
| CRI socket | `unix:///var/run/containerd/containerd.sock` |

> **Exam Tip:** If `crictl` returns an endpoint error, specify the socket explicitly:
> ```bash
> crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock ps
> ```

---

## 8. Copying Files Between Nodes and Pods

### `scp` — Best for Files and Backups

```bash
# Syntax
scp <source-file> <destination-node>:<destination-path>

# Example
scp /root/my-pod.yaml node01:/root/
```

### Pipe Command Output to Another Node

```bash
# Generic pattern
echo "Here is my secret data" | ssh node01 "cat > /root/secret.txt"

# Real exam example: save kubelet status to another node
systemctl status kubelet | ssh node01 "cat > /root/kubelet-status.txt"
```

### `kubectl cp` — Best for Pod Files and Folders

```bash
# Pod → Local
kubectl cp default/my-pod:/var/log/nginx/access.log /root/nginx-access.log

# Syntax
kubectl cp <namespace>/<pod-name>:<path-inside-pod> <local-path>
```

### Pod → Node via `hostPath` Volume

> If the exam asks a pod to write its output directly to a worker node's disk *while it runs*, you must use a Volume — not commands.

```yaml
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "echo 'Node data!' > /output/data.txt; sleep 3600"]
    volumeMounts:
    - name: node-disk
      mountPath: /output         # Path inside the pod
  volumes:
  - name: node-disk
    hostPath:
      path: /var/log/myapp       # Path on the worker node
      type: DirectoryOrCreate
```

---

## 9. Finding & Formatting Output Programmatically

### A. `--field-selector` — The Bouncer

Filter resources by field value before they are returned.

```bash
# Filter by node
kubectl get pods -A --field-selector spec.nodeName=node01

# Filter by pod phase (Running, Pending, Failed)
kubectl get pods -A --field-selector status.phase=Running
```

### B. `-o custom-columns` — The Editor

Strip away noise and format exactly the columns you need.

```bash
# Syntax
-o custom-columns=TITLE:.json.path

# Print only pod names
kubectl get pods -o custom-columns=NAME:.metadata.name --no-headers

# Save all pod names on node02 to a file
kubectl get pods -A \
  --field-selector spec.nodeName=node02 \
  -o custom-columns=NAME:.metadata.name \
  --no-headers > /root/pods.txt
```

> **Exam Trick:** Always add `--no-headers` when saving to a file so the column title isn't included.

---

## 10. Docker vs. Kubernetes Command Mapping

| Docker Field | Kubernetes Field | Purpose |
|---|---|---|
| `ENTRYPOINT` | `command` | The actual executable (e.g., `/bin/sh`) |
| `CMD` | `args` | The flags or scripts passed to that executable |

### `kubectl run` with a Command (The `--` Syntax)

Everything after `--` is treated as the container's command and args.

```bash
kubectl run test --image=busybox -- sh -c 'echo "do this" && sleep 3600'
# sh        → command (the entrypoint)
# -c '...'  → args
```

### Equivalent YAML

```yaml
spec:
  containers:
  - name: test
    image: busybox
    command: ["sh"]
    args:
    - "-c"
    - "echo 'do this' && sleep 3600"
```

---

## 11. K8s Connectivity Cheat Sheet

### Phase 1 — Internal Plumbing (Pod → Service)

Before testing traffic, ensure the "pipes" are connected.

```bash
# Check endpoints — you MUST see IP addresses in the ENDPOINTS column
kubectl get ep <svc-name> -n <namespace>
# If <none>: Service selector does not match Pod labels

# Verify the app is actually listening on the expected port
kubectl exec -it <pod-name> -n <namespace> -- netstat -tuln
```

### Phase 2 — Internal DNS Test (Service Level)

```bash
# Standard DNS format
<svc-name>.<namespace>.svc.cluster.local

# Throwaway test pod (use wget for busybox, curl for other images)
kubectl run test-pod --image=busybox:1.36 -i --rm --restart=Never \
  -- wget -qO- http://<svc-name>.<namespace>:<port>
```

### Phase 3 — Workstation Test (Port-Forward)

```bash
# Open the tunnel (runs in background)
kubectl port-forward svc/<svc-name> -n <namespace> 8888:<svc-port> &

# Test the request
curl -v http://localhost:8888

# Cleanup: bring tunnel to foreground, then Ctrl+C
fg
```

### Phase 4 — Ingress Test (The Front Door)

```bash
# Find the Ingress IP (look at the ADDRESS column)
kubectl get ingress -n <namespace>

# Test with a spoofed Host header (no real DNS needed in the exam)
curl -v -H "Host: app.novamart.com" http://<INGRESS_ADDRESS>
```

---

## 12. Kustomize

### Directory Structure

```
kustomize/
├── base/
│   ├── kustomization.yaml   # Lists all local resources
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── production/
    │   └── kustomization.yaml
    └── staging/
        └── kustomization.yaml
```

### `base/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

### `overlays/production/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 1. Point to the base
resources:
  - ../../base

# 2. Global modifiers (affects EVERY resource in base)
nameSuffix: -prod
namespace: web-prod

# 3. Targeted labels (v1beta1 "list" syntax — use 'pairs' to avoid unmarshal errors)
labels:
- includeSelectors: true
  pairs:
    env: production

# 4. Image overrides (changes tag without touching YAML)
images:
- name: nginx       # Must match the image name in base
  newTag: "1.25"

# 5. Scaling (the "single dash" rule — no dash before 'count')
replicas:
- name: web-deploy  # Must match metadata.name in base
  count: 3

# 6. Surgical patches (for everything else)
patches:
- target:
    kind: Deployment
    name: web-deploy
  patch: |-
    - op: add
      path: /spec/template/spec/containers/0/env
      value: [{name: "DB_URL", value: "prod-db"}]
```

### Common Syntax Traps & Fixes

| Error | Cause | Fix |
|---|---|---|
| `cannot unmarshal object... into []types.Label` | Used `env: prod` instead of a list | Use `- pairs: {env: prod}` |
| `resource with name "" does not match` | Put a dash before `count` in replicas | Remove dash before `count` |
| `bad address` (DNS Error) | `nameSuffix` changed your Service name | Update `curl` to use the `-prod` suffix |
| `selector mismatch` | Labels added to Pods but not Service | Ensure `includeSelectors: true` is set |

---

## 13. CustomResourceDefinitions (CRDs)

### CRD Definition

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databasebackups.storage.gameforge.com
spec:
  group: storage.gameforge.com
  names:
    kind: DatabaseBackup
    plural: databasebackups
    shortNames:
    - dbb
  scope: Namespaced
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                dbName:
                  type: string
                provider:
                  type: string
                retentionDays:
                  type: integer
              required: ["dbName", "provider"]
```

### Custom Resource (CR) Instance

```yaml
apiVersion: storage.gameforge.com/v1alpha1
kind: DatabaseBackup
metadata:
  name: mecha-pulse-weekly-backup
  namespace: default
spec:
  dbName: mecha-pulse-prod-db
  provider: aws-s3
  retentionDays: 30
```
