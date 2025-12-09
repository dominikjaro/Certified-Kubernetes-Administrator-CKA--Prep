This is the perfect way to combine **Storage** (10%) and **Workloads** (15%) into a single "Exam-Level" question.

In the CKA, you might be asked to create a StorageClass to ensure specific PVs bind to specific PVCs (using the class name as a binding key).

Here is your **Level 2 Storage Challenge**.

---

# 💾 CKA Extra Challenge: The Persisted Sidecar

**Scenario:**
You need a logging system where an application writes logs to a permanent storage location, and a sidecar agent reads those logs to "ship" them (simulated by tailing). The storage must be defined via a specific StorageClass to ensure correct binding.

---

## 🎯 Challenge 1: The StorageClass
**Objective:** Create a logical group for your storage.

Create a YAML file for a **StorageClass** named **`local-storage`**.
* **Provisioner:** `kubernetes.io/no-provisioner`
    * *Why:* Since we are using `hostPath` (static provisioning), we don't have a dynamic provisioner like AWS EBS. This tells Kubernetes "Don't try to create a cloud disk; I'll handle the PV creation manually."
* **VolumeBindingMode:** `WaitForFirstConsumer`
    * *Why:* This is a best practice. It prevents the PVC from binding until a Pod is actually scheduled (ensuring the Pod lands on the same node where the `hostPath` exists).

---

## 🎯 Challenge 2: The PV & PVC
**Objective:** Create the storage assets bound by the class.

1.  **PersistentVolume (`sidecar-pv`)**:
    * **Capacity:** `500Mi`
    * **Access Mode:** `ReadWriteMany` (Since multiple containers in the pod—and potentially other pods—might read it).
    * **StorageClassName:** `local-storage` (Matches Challenge 1).
    * **HostPath:** `/mnt/logs/shared`
2.  **PersistentVolumeClaim (`sidecar-pvc`)**:
    * **Namespace:** `cka-lab-14`
    * **Request:** `100Mi`
    * **Access Mode:** `ReadWriteMany`
    * **StorageClassName:** `local-storage`

**🛑 Checkpoint:**
If you run `kubectl get pvc -n cka-lab-14` now, the status should be **`Pending`**.
* *Why?* Because of `WaitForFirstConsumer` in the StorageClass! It is waiting for the Pod (Challenge 3) to trigger the binding.

---

## 🎯 Challenge 3: The Multi-Container Pod
**Objective:** Deploy the Pod that consumes this storage.

Create a **Pod** (not Deployment, to keep it simple) named **`logger-pod`** in `cka-lab-14`.

**Volume Configuration:**
* Define a volume named **`log-vol`** that uses the **`sidecar-pvc`**.

**Container 1: The Writer (`busybox`)**
* **Name:** `app-writer`
* **Mount:** Mount `log-vol` to **`/var/app/logs`**.
* **Command:** Write the current date to a file named `access.log` every 5 seconds.
    * `sh -c 'while true; do date >> /var/app/logs/access.log; sleep 5; done'`

**Container 2: The Reader (`busybox`)**
* **Name:** `sidecar-reader`
* **Mount:** Mount `log-vol` to **`/usr/share/logs`**. (Notice the different path!)
* **Command:** Tail the file to see the logs appearing in real-time.
    * `sh -c 'tail -f /usr/share/logs/access.log'`

---

## 🧪 Verification Steps

1.  **Check Binding:** Once the Pod is `Running`, check your PVC again. It should now be **`Bound`**.
2.  **Check the Sidecar:**
    * View the logs of the *sidecar* container (not the writer).
    * `kubectl logs logger-pod -c sidecar-reader -n cka-lab-14`
    * **Success:** You should see timestamps printing every 5 seconds.
3.  **Check Persistence:**
    * Delete the Pod: `kubectl delete pod logger-pod -n cka-lab-14`
    * Recreate the Pod (apply the same YAML).
    * Check the sidecar logs again.
    * **Success:** You should see the *old* timestamps (from before deletion) followed by new ones. The data survived!

**Good luck!** This covers SC, PV, PVC, Pods, Multi-Containers, and Volume Mounting all in one task.