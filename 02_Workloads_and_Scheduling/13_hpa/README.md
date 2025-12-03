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

TBC



