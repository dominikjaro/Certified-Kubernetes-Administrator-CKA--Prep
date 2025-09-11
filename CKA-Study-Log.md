## What is `containerd`?

1. Kubelet (Kubernetes agent on the node) interacts with `containerd` via Container Runtime Interface (CRI)
2. `containerd` pulls container images, creates containers and manages their lifecycle
3. Containers are executed using `runc`