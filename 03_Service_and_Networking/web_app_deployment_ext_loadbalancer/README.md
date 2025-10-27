Issues:


I had to make sure the cloud provider is set to external in kubelet configuration.
Cloud Controller Manager is not deployed by default in kubeadm clusters.
I have to create a daemonset for cloud-controller-manager.

First:
I had issues so I had to add --cloud-provider=external to `/var/lib/kubelet/kubeadm-flags.env` file on each node.
Restart the daemonset and the kubelet:
```
systemctl daemon-reload
systemctl restart kubelet
```

