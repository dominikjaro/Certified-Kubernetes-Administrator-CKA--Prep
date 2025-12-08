# Install Kubernetes Metrics Server

I had to install the Metrics Server in order for the Horizontal Pod Autoscaler (HPA) to function correctly. The Metrics Server collects resource metrics from Kubelets and exposes them via the Kubernetes API.

See the GitHub repository for the Metrics Server for installation instructions:
- [Metrics Server GitHub](https://github.com/kubernetes-sigs/metrics-server)

To test if the Metrics Server is working correctly, run the following command:

```bash
kubectl top nodes
```
---

## 🎯 Challenge 1: The Target Application

**Create the namespace:**

`kubectl create namespace cka-lab-13`

**Create the deployment with resource requests:**

`kubectl create deployment php-apache --image=registry.k8s.io/hpa-example --namespace=cka-lab-13 --replicas=1 --dry-run=client -o yaml > php-apache-deployment.yaml`

**Edit the `php-apache-deployment.yaml` file to include resource requests:**

```yaml
spec:
  containers:
  - name: hpa-example
    image: registry.k8s.io/hpa-example
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: "200m"
```

**Apply the deployment:**
`kubectl apply -f php-apache-deployment.yaml --namespace=cka-lab-13`

**Expose the deployment as a service:**
```bash
kubectl expose deployment php-apache --name=php-apache --port=80 --target-port=80 --type=ClusterIP --namespace=cka-lab-13 --dry-run=client -o yaml > service.yaml
kubectl apply -f service.yaml --namespace=cka-lab-13
```

---

## 🎯 Challenge 2: The Load Generator

**Create the load-generator pod:**

`kubectl run load-generator --image=busybox --namespace=cka-lab-13 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done" --dry-run=client -o yaml > load-generator-pod.yaml`

**Apply the pod:**
`kubectl apply -f load-generator-pod.yaml --namespace=cka-lab-13

## Challange 3: Configuring HPA (Imperative)

**Create the HPA:**

```bash
kubectl autoscale deployment php-apache \
  --min=1 \
  --max=10 \
  --cpu-percent=50 \
  --namespace=cka-lab-13
```

`kubectl get hpa --namespace=cka-lab-13`

## 🎯 Challenge 4: Verification & Troubleshooting

**Monitor the HPA status live:**
`kubectl get hpa php-apache --namespace=cka-lab-13 --watch`

**Scale Down:**
`kubectl delete pod load-generator --namespace=cka-lab-13`

## 🔥 Challenge 5: HPA with Declarative YAML

**Export the existing HPA to a file:**
`kubectl get hpa php-apache --namespace=cka-lab-13 -o yaml > hpa.yaml`

**Modify the `hpa.yaml` file:**
```yaml
spec:
  maxReplicas: 5
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 30
        type: Utilization
    type: Resource
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
```

**Apply the updated HPA configuration:**
`kubectl apply -f hpa.yaml --namespace=cka-lab-13`

**Verify the changes:**
`kubectl get hpa php-apache --namespace=cka-lab-13`

