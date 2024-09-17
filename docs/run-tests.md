Once the tests clusters have been deployed, resources [created](../manifests/creating-resources.md) and the multi-cluster networking solution has been configured between the two namespsaces, you are now ready to begin testing.

# iperf3

## On cluster-1
 
```
oc rsh <client-pod>
iperf3 -c <server_service_ip>
```
`server_service_ip` is the server service that has been exposed using the multi-cluster networking solution. In the case of skupper it is exposed by using `skupper expose deployment server` on cluster-2. The service ip can be obtained using `oc get svc` on cluster-1.

# uperf

## On cluster-1

```
oc rsh <client-pod>
uperf -m /tmp/throughput/throughput.xml
uperf -m /tmp/rr/rr.xml
```

All the required remote IPs/ports will be auto-populated in the config using environment variables.

