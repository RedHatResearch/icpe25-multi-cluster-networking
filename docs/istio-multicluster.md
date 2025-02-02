# Istio Multicluster

The Istio framework several [deployment models](https://istio.io/latest/docs/setup/install/multicluster/) to set up a Istio mesh across multiple Kubernetes clusters.

We've used the [multi-primary on different networks model](https://istio.io/latest/docs/setup/install/multicluster/multi-primary_multi-network/) on this experiment. In this model, the two clusters (cluster1 and cluster2) have a running istio mesh control plane, where each of them watches the k8s API Servers in both clusters for to discover endpoints, the clusters are deployed on different networks (network1 and network2).

Service workloads across cluster boundaries communicate indirectly, via dedicated ingress gateways for east-west traffic. The gateway in each cluster must be reachable from the other cluster.

Deployment of **Istio multi-primary on different networks** is out of scope of this document, this document will only walkthrough the process of the test execution.

## Creating resources

Deploy the istio-perf and its resources in both clusters:

```shell
kubectl create -f - << EOF
apiVersion: v1
kind: Namespace
metadata:
  name: istio-perf
  labels:
    istio-injection: enabled
EOF
kubectl create -f manifests/istio-multicluster/
```