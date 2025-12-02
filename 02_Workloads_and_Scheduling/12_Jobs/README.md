## Outcome and the commands I used during the challenges

First I create the namespace by running `kubectl create namespace cka-lab-12`.

### 🎯 Challenge 1: The "One-Shot" Job

```bash
kubectl  create job process-data-1 --image=busybox --namespace=cka-lab-12 --dry-run=client -o yaml -- /bin/sh -c 'echo "Hello, the one-shot job is done"' > process-data-1-challenge-1.yaml
kubectl apply -f process-data-1-challenge-1.yaml
```

One pod was created and the job completed fine.

### 🎯 Challenge 2: The "Work Queue" (Completions)

```bash
kubectl  create job queue-worker --image=busybox --namespace=cka-lab-12 --dry-run=client -o yaml -- /bin/sh -c 'echo "Processing item" && sleep 2' > queue-worker-challenge-2.yaml
```

Then just edited the file to add `completions: 5` under `spec:` and parallelism is 1 by default.

```bash
kubectl apply -f queue-worker-challenge-2.yaml
```

Five pods were created sequentially and the job completed fine.

### 🎯 Challenge 3: The "High Performance" Job (Parallelism)

```bash
kubectl  create job fast-batch-processor --image=busybox --namespace=cka-lab-12 --dry-run=client -o yaml -- /bin/sh -c 'echo "Fast batch processor is running..." && sleep 5' > fast-batch-processor-challenge-3.yaml
```

Then just edited the file to add `completions: 10` and `parallelism: 2` under `spec:`.

`kubectl apply -f fast-batch-processor-challenge-3.yaml`

Ten pods were created in pairs and the job completed fine.

### 🎯 Challenge 4: The "Time-Bound" Job (ActiveDeadline)

`kubectl create job risky-job --image=busybox --namespace=cka-lab-12 --dry-run=client -o yaml -- /bin/sh -c 'echo "Risky job is running..." && sleep 60' > risky-job-challenge-4.yaml`

Then just edited the file to add `activeDeadlineSeconds: 15` under `spec:`.

`kubectl apply -f risky-job-challenge-4.yaml`

The pod was killed after 15 seconds as expected and the job failed.

### 🎯 Challenge 5: The "Self-Cleaning" Job (TTL)

`kubectl create job temporary-job --image=busybox --namespace=cka-lab-12 --dry-run=client -o yaml -- /bin/sh -c 'echo "I am done"' > temporary-job-challenge-5.yaml`

Then just edited the file to add `ttlSecondsAfterFinished: 30` under `spec:`.

`kubectl apply -f temporary-job-challenge-5.yaml`

The job completed fine and after 30 seconds the job and its pods were deleted automatically. When I ran `kubectl get pods -n cka-lab-12` the temporary pods did not appear anymore. 

### ⏰ Bonus Challenge 6: The CronJob

`kubectl create cronjob checker-cron --image=busybox --namespace=cka-lab-12 --schedule="*/1 * * * *" --dry-run=client -o yaml -- /bin/sh -c 'echo "Checking system..."' > checker-cron-bonus-challenge-6.yaml
`

The `schduele= "*/1 * * * *" means it will run every minute.

`kubectl apply -f checker-cron-challenge-6.yaml`

Once I have verified the jobs ran fine I suspended the Cronjob - just added the `Suspend:true` under the `spec.`