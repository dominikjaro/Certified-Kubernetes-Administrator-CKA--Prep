***

# 🏋️‍♀️ CKA Practice Lab: Horizontal Pod Autoscaling (HPA)

**Domain:** Workloads & Scheduling (15%)
**Prerequisite:** Ensure `kubectl top nodes` works. If not, install the [Metrics Server](https://github.com/kubernetes-sigs/metrics-server).

---

## 🎯 Challenge 1: The Target Application
**Objective:** Deploy the application that will be scaled.

1.  Create a namespace named **`cka-lab-13`**.
2.  Create a Deployment named **`php-apache`**.
    * **Image:** `registry.k8s.io/hpa-example`
    * **Replicas:** 1
3.  **Critical Constraint:** You **MUST** define resource requests for the container.
    * **CPU Request:** `200m`
    * *(Note: HPA cannot calculate usage percentage without a request value).*
4.  **Service:** Expose this deployment on TCP port **80**.

---

## 🎯 Challenge 2: The Load Generator
**Objective:** Create a "bot" that generates traffic to spike the CPU.

1.  Run a simple Pod named **`load-generator`** in the same namespace.
2.  **Image:** `busybox`
3.  **Command:** Run a shell script that executes an infinite loop.
    * Inside the loop, run: `wget -q -O- http://php-apache` (or whatever your service DNS name is).
4.  Ensure the pod stays running (interactive mode or infinite loop logic).

---

## 🎯 Challenge 3: Configuring HPA (Imperative)
**Objective:** Set up the Autoscaler using the fastest method possible (CLI).

1.  Create a Horizontal Pod Autoscaler targeting the `php-apache` deployment.
2.  **Scaling Rules:**
    * **Minimum Pods:** 1
    * **Maximum Pods:** 10
    * **Target CPU Utilization:** 50%
    * *(Logic: If the pod uses more than 50% of its requested 200m, spin up a new one).*

---

## 🎯 Challenge 4: Verification & Troubleshooting
**Objective:** Observe the autoscaling behavior.

1.  **Monitor:** Run a command to watch the HPA status live.
    * *Success Criteria:* The `TARGETS` column should eventually show a percentage (e.g., `250%/50%`) instead of `<unknown>`.
2.  **Scale Up:** Verify that new Pods are being created.
3.  **Scale Down:** Delete the `load-generator` pod and watch the HPA.
    * *Observation:* Note that it takes a few minutes to scale down (Stabilization Window).

---

## 🔥 Challenge 5: HPA with Declarative YAML
**Objective:** Modify the HPA configuration using a YAML file (Declarative approach).

1.  Export your existing live HPA to a file named `hpa-config.yaml`.
2.  **Modify the file:**
    * Change `maxReplicas` to **5**.
    * Change `targetCPUUtilizationPercentage` to **20%**.
3.  **Apply:** Update the live resource using this file.