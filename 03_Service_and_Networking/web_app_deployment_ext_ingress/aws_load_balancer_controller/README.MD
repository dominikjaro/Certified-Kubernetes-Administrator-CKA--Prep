I have to install the AWS Load Balancer Controller to my cluster on the control plane node. 

To be able to set up ClusterIp Service and Ingress resources that work with AWS Load Balancer Controller, you need to install the controller in your Kubernetes cluster.

I used Helm charts from: https://github.com/kubernetes-sigs/aws-load-balancer-controller/tree/main/helm/aws-load-balancer-controller

