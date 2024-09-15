# Test Setup

For the purposes of testing, we will be creating resources on two different clusters, cluster-1 and cluster-2. cluster-2 will have a service that will be exposed on cluster-1 through the multi-cluster networking solution. This service can be any kind of server like iperf3, uperf or nginx. cluster-1 will run the client pod that will send traffic to the service whose endpoints include the server. 

Please follow the below steps exactly in the order specified for the test setup.

On cluster-2, after exporting the appropriate path to the KUBECONFIG

```
oc create -f server.yaml
oc create -f service.yaml
```
On cluster-1, after exporting the appropriate path to the KUBECONFIG

```
oc create -f throughput-config.yaml
oc create -f rr-config.yaml
oc create -f client.yaml
```

