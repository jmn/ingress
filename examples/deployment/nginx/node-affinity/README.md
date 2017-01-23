# Nginx Ingress with Node affinity

This example aims to demonstrate how to configure 
the deployment of an nginx ingress controller to bind to 
specific nodes using: NodeSelector, Labels, (Affinity, AntiAffinity), 
Taints (v1.5+) and Tolerations (v.1.5+).

## NodeSelector and Labels (GCE Example)

You may keep your loadbalancer nodes in a pool e.g: `lb-pool-1`
GCE will automatically apply the label `cloud.google.com/gke-nodepool: "lb-pool-1"`.
It is possible to ensure that new pods will only be 
scheduled onto nodes in that pool by using NodeSelector 
(Note: NodeSelector is slated for eventual deprecation in favor of NodeAffinity.)
in the PodSpec.e.g:

```
[...]
    spec:
      nodeSelector:
        cloud.google.com/gke-nodepool: "lb-pool-1"
      containers:
      - image: gcr.io/google_containers/nginx-ingress-controller:0.8.3
[...]
```

## Keeping other pods out - Taints and Tolerations (Kubernetes v1.5+)
- [Kubectl taint](https://kubernetes.io/docs/user-guide/kubectl/kubectl_taint/)

Prevent overloading your loadbalancer node buy strictly denying access using
taints.

To directly apply a taint using kubectl, do: 
`kubectl taint nodes foo dedicated=production-loadbalancer:NoSchedule`

BE AWARE: In GCE at least, kubectl taint will be ephemeral and you will 
loose the taint if you say, upgrade the node's Kubernetes API version
thus creating a new node. To make a "permanent" or taint all nodes in a pool:


### Toleration - Permitting specific scheduling onto a "tainted" node

Configure nginx-ingress-controller:
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  labels:
    k8s-app: nginx-ingress-controller
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      annotations:
        scheduler.alpha.kubernetes.io/tolerations: '[{"key":"dedicated", "value":"production-loadbalancer"}]'
      labels:
        k8s-app: nginx-ingress-controller
[...]
```
