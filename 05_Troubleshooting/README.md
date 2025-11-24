# Troubleshooting AWS Load Balancer Controller Deployment

This document outlines common issues encountered during the deployment of the AWS Load Balancer Controller and their solutions.

## 1. Hostname Configuration Issues (`hostname: true`)

**Issue:**
The AWS Load Balancer Controller may fail to register nodes or targets if the hostname configuration is incorrect. This often manifests as targets not becoming healthy or the controller logging errors related to node lookup.

**Context:**
In some CNI or Kubelet configurations, explicit hostname settings are required for the controller to correctly map Kubernetes nodes to AWS EC2 instances.

**Troubleshooting:**
*   **Check Kubelet Args:** Ensure that the `--hostname-override` flag is not causing mismatches with the EC2 private DNS name if you are relying on AWS integration.
*   **Controller Flags:** Verify if you need to enable specific hostname features in the controller deployment (e.g., `--hostname-annotation` if using custom DNS).
*   *(Self-Note: If you used a specific flag like `hostname: true` in a Helm chart or config, please verify if this refers to `automountServiceAccountToken` or a specific chart value for hostname topology).*

## 2. Pod Tolerations

**Issue:**
The AWS Load Balancer Controller pods remain in a `Pending` state and are not scheduled on any node.

**Cause:**
If your cluster has taints on specific nodes (e.g., control plane nodes, or nodes reserved for system components) and no other nodes are available, the controller pods will fail to schedule unless they have matching tolerations.

**Solution:**
Add tolerations to the AWS Load Balancer Controller deployment to allow it to run on tainted nodes if necessary.

```yaml
spec:
  template:
    spec:
      tolerations:
      - key: "node-role.kubernetes.io/master"
        operator: "Exists"
        effect: "NoSchedule"
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"
```

## 3. Node Disk Pressure

**Issue:**
Pods are being evicted, or the AWS Load Balancer Controller fails to start with `Evicted` status.

**Cause:**
The node where the pod is trying to run has run out of disk space or inodes, triggering the Kubelet's disk pressure condition.

**Solution:**
*   **Cleanup:** Remove unused Docker images and stopped containers.
    ```bash
    # Check disk space
    df -h    
    # For Docker
    docker system prune -a
    # OR for containerd/crictl
    crictl rmi --prune
    ```
*   **Log Rotation:** Check if system logs or container logs are consuming excessive space.
*   **Scale Nodes:** If cleanup is insufficient, add more disk space to the node or add more nodes to the cluster.

## 4. Service Type Configuration (ClusterIP vs NodePort)

**Issue:**
The Target Group Binding fails, or the Load Balancer cannot route traffic to the pods.

**Context:**
By default, the AWS Load Balancer Controller in `instance` mode expects to route traffic to `NodePort` services. If you use `ip` mode, it can route directly to Pod IPs (compatible with `ClusterIP`).

**Resolution:**
*   **Instance Mode:** If your Ingress or TargetGroupBinding uses `target-type: instance`, you **must** ensure your Service is of type `NodePort`.
*   **IP Mode:** If you want to use `ClusterIP`, ensure you annotate the Ingress/Service with `alb.ingress.kubernetes.io/target-type: ip`.

*In this specific deployment, we switched from `ClusterIP` to `NodePort` to resolve connectivity issues.*

## 5. Error message: 503 Service Temporarily Unavailable

**Issue:**
The Load Balancer returns a 503 error, indicating that the target group is not healthy or the target is not registered.

**Cause:**
*   **Target Group Health:** The target group may not have any healthy targets, or the targets may not be registered in the target group.
*   **Target Registration:** The target may not be registered in the target group, or the target group may not be associated with the Load Balancer.

**Solution:**
*   **Check Target Group Health:** Verify that the target group has healthy targets and that the targets are registered in the target group.
*   **Check Target Registration:** Verify that the target group is associated with the Load Balancer and that the target is registered in the target group.
*   **Check Target Group Configuration:** Verify that the target group is configured correctly and that the target group is not in a failed state.

**🛠️ How to Add Them Manually (Immediate Fix)**
Since the Controller didn't add them automatically (likely due to the previous ProviderID or Tagging issues), you can do it manually to get your site working right now.

1. Find the NodePort You need to know which port opened on the nodes. Run this:
    ```bash
    kubectl get svc <service-name> -n <namespace>
    ```

2. Register in AWS Console
   1. Go to EC2 -> Target Groups.
   2. Select your Target Group -> Targets tab -> Register targets.
   3. Available instances: Select your worker nodes (c1-node1, c1-node2).
   4. Ports: Type in the NodePort number you found in Step 1 (e.g., 31452). Do not use 80 or 8080.
   5. Click Include as pending below.
   6. Click Register pending targets.

## 6. Error message: Request Timed Out

**Issue:**
Meaning: The Load Balancer sent a request, but it hit a firewall.

1. Get the ALB Security Group ID:
    * Go to Load Balancers -> Select your ALB -> Security tab.
    * Copy the Security Group ID.

2. Get the Worker Node Security Group ID:
    * Go to EC2 -> Security Groups.
    * Select your Worker Node Security Group.
    * Copy the Security Group ID.

3. Add Inbound Rule to Worker Node Security Group:
    * Go to EC2 -> Security Groups.
    * Select your Worker Node Security Group.
    * Click Inbound Rules tab.
    * Click Edit Inbound Rules.
    * Click Add Rule.
    * Select Type: All Traffic.
    * Select Source: Security Group.
    * Select Source Security Group: Select your ALB Security Group.
    * Click Save Rules.
    
---

## 7. The Service was serving the wrong deployment

At the exercise: `web_app_deployment_ext_ingress_fanout`

The ingress was serving the /v2/* AND /v3/* path to the v1 deployment.

To find out I was using `curl -v k8s-doitlab0-staticwe-9a130b281c-1939762810.eu-north-1.elb.amazonaws.com/v2/` and watching the logs for the v1 deployemnt, where I saw the error. 

**Cause:**
TBC