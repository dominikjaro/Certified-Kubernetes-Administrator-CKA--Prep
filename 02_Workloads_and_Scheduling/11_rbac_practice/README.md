### 🎯 Challenge 1: The Setup (Identity)

**Objective:** Create the environment and the identity that will be used by our workload.

1. Create a new Namespace named `cka-lab-11`.
2. Create a ServiceAccount named `pod-manager` inside that namespace.
    - *Concept Check:* This ServiceAccount will act as the "ID card" for your future Pods.

---

### 🔐 Challenge 2: The "Broken" Rules (RBAC)

**Objective:** Create a set of permissions that are *insufficient* for the task, mimicking the error in the lab.

1. Create a **Role** named `pod-viewer` in the `cka-lab-11` namespace.
    - **Rule:** It should only allow **listing** and **watching** resources of type **pods**.
2. Create a **RoleBinding** named `viewer-bind` in the same namespace.
    - **Action:** Bind the `pod-viewer` Role to the `pod-manager` ServiceAccount you created in Step 1.

---

### 🕵️‍♂️ Challenge 3: Verification (The "Can I?" Test)

**Objective:** Verify the permissions *without* deploying a Pod yet. This is a critical exam skill called **Impersonation**.

1. Use the `kubectl auth can-i` command to check permissions.
2. **Test 1 (Should say YES):** Check if the `pod-manager` ServiceAccount can **list** pods in `cka-lab-11`.
    - *Hint:* You will need the flags `-as`, `-namespace`, and potentially the full username format for a service account (`system:serviceaccount:<ns>:<sa>`).
3. **Test 2 (Should say NO):** Check if the `pod-manager` ServiceAccount can **patch** or **delete** pods in `cka-lab-11`.

---

### 🛠️ Challenge 4: The Fix

**Objective:** Grant the missing permissions to resolve the error.

1. **Edit** the existing `pod-viewer` Role (don't delete and recreate it—practice editing live objects!).
2. Add the **verbs** required to modify pods: `patch` and `delete`.
3. **Verify:** Run the **Test 2** command from Challenge 3 again. It should now say **YES**.
4. **Final Test:** If you exec back into your `test-client` pod, you should now be able to delete pods successfully.

---

### 🧠 Key CKA Takeaways

- **Troubleshooting Flow:** When you see `403 Forbidden` in logs, immediately think: "Check the ServiceAccount, then check the Role/RoleBinding."
- **ServiceAccount format:** Memorize that the user string for a ServiceAccount is `system:serviceaccount:<namespace>:<name>`.
- **`auth can-i`:** This is your best friend. It saves you from deploying dummy pods just to check if permissions are correct.

---

### Commands I used to complete the challenges:

```bash
# Create the namespace
kubectl create namespace cka-lab-11

# Create the ServiceAccount
kubectl create serviceaccount pod-manager -n cka-lab-11 --dry-run=client -o yaml > serviceaccount.yaml
kubectl apply -f serviceaccount.yaml

# Create the Role
kubectl create role pod-viewer --verb=list,get --resource=pods -n cka-lab-11 --dry-run=client -o yaml > role_pod_viewer.yaml
kubectl apply -f role_pod_viewer.yaml

# Create the RoleBinding
kubectl create rolebinding viewer-bind --role=pod-viewer --serviceaccount=pod-manager -n cka-lab-11 --dry-run=client -o yaml > rolebinding.yaml
kubectl apply -f rolebinding.yaml

# Verification - Test
kubectl auth can-i list pods --as=system:serviceaccount:cka-lab-11:pod-manager -n cka-lab-11
# Verification - Test 2 - check all the permissions
kubectl auth can-i --list --as=system:serviceaccount:cka-lab-11:pod-manager -n cka-lab-11

# Edit the Role to add patch and delete verbs
kubectl edit role pod-viewer -n cka-lab-11
# (Add patch and delete to the verbs list)
# Verify again
```