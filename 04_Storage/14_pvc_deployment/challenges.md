***

# 💾 CKA Practice Lab: Persistent Storage

**Domain**: Storage (10%) **Goal**: Persist data across Pod restarts using PVs and PVCs.

---

## 🎯 Challenge 1: The Setup
**Objective:** Prepare the environment.

1.  Create a namespace named **`cka-lab-14-extra`**.
2.  **Context Check:** Check if your cluster has a default `StorageClass`.
    * *Hint:* `kubectl get sc`.
    * *Note:* If one exists (e.g., `gp2`), you could use dynamic provisioning. For this exercise, assume you **don't** have one and must build it manually.

---

## 🎯 Challenge 2: The Persistent Volume (PV)
**Objective:** Manually provision a storage asset in the cluster.

Create a YAML file for a **PersistentVolume** named **`redis-pv`**.
* **Capacity:** `1Gi`
* **Access Mode:** `ReadWriteOnce`
* **Storage Class Name:** `manual` (This is arbitrary strings, but important for binding).
* **Volume Type:** `hostPath`.
    * *Path on the Node:* `/mnt/data/redis`
    * *Note:* In the exam, they might ask you to ensure this directory exists on the node, but Kubernetes usually creates it if missing.

---

## 🎯 Challenge 3: The Persistent Volume Claim (PVC)
**Objective:** Create a request for storage that binds to your PV.

Create a YAML file for a **PersistentVolumeClaim** named **`redis-claim`** in the `cka-lab-14` namespace.
* **Request Storage:** `1Gi` (Must match or be less than the PV).
* **Access Mode:** `ReadWriteOnce` (Must match the PV).
* **Storage Class Name:** `manual` (Must match the PV to ensure binding).

**🛑 Verification Stop:**
Before proceeding, check the status of your PVC.
* It should show **`Bound`**.
* If it is **`Pending`**, your PV and PVC definitions do not match (Capacity, AccessMode, or StorageClassName).

---

## 🎯 Challenge 4: The Workload
**Objective:** Deploy Redis and mount the storage.

Create a Deployment named **`redis`** in the `cka-lab-14` namespace.
* **Image:** `redis:alpine`
* **Replicas:** 1
* **Volume Mount:**
    * Mount the `redis-claim` PVC to the container path **`/data`**.
    * *Reasoning:* Redis defaults to saving its database dump (`dump.rdb`) to the working directory (`/data`).

---

## 🎯 Challenge 5: The "Destructive" Test
**Objective:** Prove that data survives a Pod deletion (Persistence).

1.  **Write Data:**
    * Exec into the Redis pod.
    * Use `redis-cli` to set a key (e.g., `set mykey cka-passed`).
    * *Crucial Step:* Force Redis to save to disk *now* by running the command `save` inside `redis-cli` (or verify `dump.rdb` exists in `/data`).
2.  **Delete:** Delete the Redis **Pod** (not the Deployment).
    * *Observation:* The Deployment controller will immediately start a new Pod.
3.  **Read Data:**
    * Exec into the **new** Pod.
    * Use `redis-cli` to retrieve the key (`get mykey`).
    * *Success Criteria:* It returns `"cka-passed"`.

---

## 🔥 Extra Credit (StatefulSet Concept)
In the introduction, the lab mentioned **StatefulSets**. While not required for this specific task, try this mental exercise:

* **Question:** If you scaled your Redis Deployment to **3 Replicas**, what happens?
    * *Hint:* All 3 Pods would try to write to the **exact same** `hostPath` directory on the node (ReadWriteOnce) or share the same PVC. This usually leads to data corruption in databases.
    * *Why StatefulSets?* A StatefulSet with a `volumeClaimTemplate` creates a **unique** PVC for *every* replica (pod-0 gets pvc-0, pod-1 gets pvc-1), which is why they are preferred for databases.