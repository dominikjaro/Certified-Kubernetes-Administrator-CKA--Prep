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
TBC