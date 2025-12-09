## 🎯 Challenge 1: The Setup

```bash
# Create the namespace
kubectl create namespace cka-lab-14
# Save it to kubeconfig
kubectl config set-context --current --namespace=cka-lab-14

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

```bash
# Create the Deployment
kubectl run redis --image=redis:alpine --replicas=1 -n cka-lab-14 --dry-run=client -o yaml > deployment.yaml
#Add the Volume and Volume Mount to deployment.yaml
kubectl apply -f deployment.yaml
```

## 🎯 Challenge 5: The "Destructive" Test

```bash
# Get the Pod name
kubectl get pods

# Exec into the Pod and create a test file
kubectl exec -it <redis-pod-name> -- sh
# Inside the Pod - use redis-cli to set a key
redis-cli set mykey cka-passed
redis-cli get mykey # Verify the key is set
exit

# Delete the Pod
kubectl delete pod <redis-pod-name>

# Wait for the Pod to be recreated And then exec into the new Pod
kubectl exec -it <new-redis-pod-name> -- sh
# Inside the Pod - verify the key still exists
redis-cli get mykey # It should return 'cka-passed'
```
