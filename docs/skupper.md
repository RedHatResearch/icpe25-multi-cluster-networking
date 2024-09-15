Skupper creates a Virtual Application Networking that enables multi-cluster networking at the L7 level. It supports any TCP based protocol. To connect the iperf3/uperf client running on one cluster to the server running on another cluster, please follow the below steps. Installation of skupper cli is beyond the scope of this doc, we will only cover skupper configuration to connect namesapces across different OpenShift clusters.

### On cluster-1

```
export KUBECONFIG=<PATH to Kubeconfig>
oc new-project client
```

### On cluster-2
```
export KUBECONFIG=<PATH to Kubeconfig>
oc new-project server
```

Deploy the resources need to setup the test by following this [doc] (../manifests/creating-resources.md)

## Create sites

### On cluster-1

```
skupper init
skupper status
```
	
### On cluster-2

```
skupper init
skupper status
```  

## Link sites

### On cluster-1

```
skupper token create ~/secret.token
```  

### On cluster-2

```
skupper link create ~/secret.token
```

## Expose services

### On cluster-2

```
skupper expose svc/iperf3
skupper expose svc/uperf
```

Now these services on cluster-2 should be visible on cluster-1 and you can conduct your performance tests between cluster-1 and cluster-2.

