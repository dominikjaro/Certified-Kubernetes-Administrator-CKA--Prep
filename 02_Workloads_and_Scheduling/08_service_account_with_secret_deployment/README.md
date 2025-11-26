In this lab I created a service account, a secret and a deployment.

The service account is named `cka-lab-sa-08` and the secret is named `cka-lab-08-db-creds`.

To save some time I used a hybrid approach, where I created the service account and the secret first, and then I created the deployment.

## 💡 The Winning CKA Strategy: "Imperative to Declarative"
The best strategy is a Hybrid Approach: Use kubectl create to generate the YAML skeleton for you, save it to a file, and then edit it.

You rarely create directly. You create to generate.
`--dry-run=client -o yaml > file.yaml`

1. Secret
```bash
kubectl create secret generic cka-lab-08-db-creds --from-literal=username=admin --from-literal=password=<my_secret> --dry-run=client -o yaml > secret.yaml
kubectl apply -f secret.yaml
```

2. Service Account
```bash
kubectl create serviceaccount cka-lab-sa-08 --dry-run=client -o yaml > cka_lab_08_sa.yaml
kubectl apply -f cka_lab_08_sa.yaml
```

3. Deployment
```bash
kubectl create deployment cka-lab-08-deployment --image=nginx --dry-run=client -o yaml > cka_lab_08_deployment.yaml
kubectl apply -f cka_lab_08_deployment.yaml
```

4. Edit the deployment to use the service account and the secret
`sudo vim cka_lab_08_deployment.yaml`

```yaml
spec:
      serviceAccountName: cka-lab-sa-08
      containers:
      - image: nginx
        name: nginx
        resources: {}
        env:
        - name: PASS
          valueFrom:
            secretKeyRef:
              name: cka-lab-08-db-creds
              key: password
        - name: USER
          valueFrom:
            secretKeyRef:
              name: cka-lab-08-db-creds
              key: username
```

---

## ⚠️ When MUST you use YAML from scratch?
There are certain resources that do not have imperative generators. For these, you must know how to find the YAML in the Kubernetes Documentation (allowed during the exam) and copy-paste it.

No kubectl create exists for:

PersistentVolumes (PV)

PersistentVolumeClaims (PVC)

NetworkPolicies

StorageClasses (usually)

For these, you:

Search kubernetes.io/docs for "PersistentVolume".

Copy the example YAML.

Paste into vi pv.yaml.

Edit the values.

---

## Troubleshooting

I was not able to generate the `yaml` file because of some permission issues.
I had to change the ownership of the current directory.
`sudo chown -R $USER:$USER .`

---

## Testing

Check the objects make sure the pods are running. Then you can check the environment variables.

**Pick one of your running pods**
`kubectl exec -it <pod-name> -n cka-lab-08 -- env | grep -E "USER|PASS"`