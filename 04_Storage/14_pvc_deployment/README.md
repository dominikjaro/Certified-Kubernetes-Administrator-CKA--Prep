## 🎯 Challenge 1: The Setup

```bash
# Create the namespace
kubectl create namespace cka-lab-14

# Check if your cluster has a default `StorageClass`
kubectl get sc
```

*Note:* If one exists (e.g., `gp2`), you could use dynamic provisioning. For this exercise, assume you **don't** have one and must build it manually.

## 🎯 Challenge 2: The Persistent Volume (PV)

Create a YAML file for a **PersistentVolume** named **`redis-pv`** and apply it.

`kubectl apply -f redis-pv.yaml`

## 🎯 Challenge 3: The Persistent Volume Claim (PVC)

Create a YAML file for a **PersistentVolumeClaim** named **`redis-claim`** in the `cka-lab-14` namespace and apply it.

`kubectl apply -f redis-claim.yaml -n cka-lab-14`

Verify the status of your PVC:

`kubectl get pvc -n cka-lab-14`
* It should show **`Bound`**.
* If it is **`Pending`**, your PV and PVC definitions do not match (Capacity

## 🎯 Challenge 4: The Workload

TBC

