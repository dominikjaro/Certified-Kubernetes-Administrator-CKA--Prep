### 🎯 Challenge 1: The "One-Shot" Job

**Objective:** Create a basic Job named `process-data-1` in namespace `cka-lab-12`.

- **Image:** `busybox`
- **Command:** It should simply print "Job Complete" and exit successfully.
- **Verification:** Ensure the Pod status becomes `Completed` (not `Running` or `CrashLoopBackOff`).

**💡 Hint:**

- Remember that `busybox` needs a command or it crashes.
- Use `kubectl create job --image=...` to generate the skeleton.

---

### 🎯 Challenge 2: The "Work Queue" (Completions)

**Objective:** Create a Job named `queue-worker`.

- **Image:** `busybox`
- **Command:** `echo "Processing item" && sleep 2`
- **Logic:** Imagine you have 5 items in a database to process. You want this Job to run exactly **5 times** successfully to process them all.
- **Constraint:** Only process **one item at a time** (Sequential).

**💡 Hint:**

- Look for a field in the `spec` called `completions`.
- How do you ensure they don't run at the same time? (Default parallelism is 1, but check the docs).

---

### 🎯 Challenge 3: The "High Performance" Job (Parallelism)

**Objective:** Create a Job named `fast-batch-processor`.

- **Image:** `busybox`
- **Command:** `sleep 5`
- **Logic:** You have **10 tasks** to finish (`completions`).
- **Constraint:** You want to finish faster by running **2 Pods at the same time**.

**💡 Hint:**

- You need `completions` (total work) AND `parallelism` (concurrency).
- Watch the pods with `kubectl get pods -w`. You should see them appearing in pairs.

---

### 🎯 Challenge 4: The "Time-Bound" Job (ActiveDeadline)

**Objective:** Create a Job named `risky-job`.

- **Image:** `busybox`
- **Command:** `sleep 60` (It hangs for a minute).
- **Constraint:** You only have budget/time for this job to run for **15 seconds**. If it takes longer, Kubernetes must kill it forcefully.

**💡 Hint:**

- You are looking for a field inside `spec` (not `spec.template.spec`) that defines a **Deadline**.
- Search `kubectl explain job.spec` for "deadline".

---

### 🎯 Challenge 5: The "Self-Cleaning" Job (TTL)

**Objective:** Create a Job named `temporary-job`.

- **Image:** `busybox`
- **Command:** `echo "I am done"`
- **Constraint:** You don't want to delete this manually. Configure the Job so that **30 seconds after it finishes**, the Job object (and its pods) automatically disappear from the cluster.

**💡 Hint:**

- This is a feature called **TTL (Time To Live)**.
- Look for `ttlSecondsAfterFinished`.

---

### ⏰ Bonus Challenge 6: The CronJob

**Objective:** Create a **CronJob** named `checker-cron`.

- **Schedule:** Runs every minute (`/1 * * * *`).
- **Logic:** It creates a Job that prints "Checking system...".
- **Constraint:** If the job fails or hangs, we don't want a pile-up. Keep only the last **3 successful** job histories and **1 failed** job history.

**💡 Hint:**

- `kubectl create cronjob` can generate the basics.
- Check the YAML for `successfulJobsHistoryLimit`.

---

### 🧪 How to verify your work

For every challenge, perform these checks:

1. **Check Status:** `kubectl get jobs -n cka-lab-12`
    - Look at the `COMPLETIONS` column (e.g., `0/5` -> `5/5`).
2. **Check Pods:** `kubectl get pods -n cka-lab-12`
    - Are they `Completed`?
    - For Challenge 4, is the status `Terminated` or `DeadlineExceeded`?
3. **Check Cleanup:** For Challenge 5, wait 30 seconds and run `kubectl get jobs`. It should return "No resources found".

---

### Commands I used to complete the challenges:

```bash
