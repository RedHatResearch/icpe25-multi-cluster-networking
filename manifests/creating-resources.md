# Test Setup

For the purposes of testing, we will be creating resources on two different clusters, cluster-1 and cluster-2. cluster-2 will have a service that will be exposed on cluster-1 through the multi-cluster networking solution. This service can be any kind of server like iperf3, uperf or nginx. cluster-1 will run the client pod that will send traffic to the service whose endpoints include the server.

Please follow the below steps exactly in the order specified for the test setup.

On **cluster-2**, after exporting the appropriate path to the KUBECONFIG

```
$ kubectl create -f server.yaml
$ kubectl expose deployment server
```
On **cluster-1**, after exporting the appropriate path to the KUBECONFIG

```
$ kubectl create -f uperf-configs.yaml
$ kubectl create -f client.yaml
```

## Submariner

On **cluster-2**:
 - Make sure the server pod is running
 - Create the service:
    ```
    $ kubectl create -f server-svc-pp.yaml
    ```
 - Export the service (it creates a `ServiceExport` object):
    ```
    $ subctl export service --namespace default server-pp
    ```
 - Check the Status information on the ServiceExport object:
    ```
    $ kubectl -n default describe serviceexports | egrep "Message|Status"
    Status:
        Message:
        Status:                True
        Message:               Service was successfully exported to the broker
        Status:                True
    ```

On **cluster-1**:
 - Verify that the exported service was imported to cluster-1 as expected. Submariner (via Lighthouse) automatically creates a corresponding ServiceImport in the service namespace:
    ```
    $ kubectl get -n default serviceimport
    NAME        TYPE           IP    AGE
    server-pp   ClusterSetIP         10h
    ```
 - Log into the client pod:
    ```
    $ oc rsh client-pp-7544c8c457-7lvzf
    ```
 - Run the networking througput test:
    ```
    sh-5.1# for size in 64 1024 8192; do for th in 1 2 4; do for i in {1..3}; do echo "Threads: ${th}, size: ${size}"; nthr=$th size=$size uperf_service_host=server-pp.default.svc.clusterset.local uperf_service_port_uperf_data=30000 uperf -m /tmp/throughput/throughput.xml| grep Total; sleep 10s ; done; done; done
    Threads: 1, size: 64
    Total      3.55GB /  62.37(s) =   489.11Mb/s      955302op/s
    Threads: 1, size: 64
    Total      3.57GB /  62.36(s) =   492.30Mb/s      961522op/s
    Threads: 1, size: 64
    Total      3.55GB /  62.37(s) =   489.51Mb/s      956066op/s
    Threads: 2, size: 64
    Total      5.68GB /  62.35(s) =   782.36Mb/s     1528049op/s
    Threads: 2, size: 64
    Total      5.81GB /  62.36(s) =   799.74Mb/s     1561998op/s
    Threads: 2, size: 64
    Total      6.04GB /  62.36(s) =   831.72Mb/s     1624455op/s
    Threads: 4, size: 64
    Total      8.31GB /  62.36(s) =     1.14Gb/s     2236111op/s
    Threads: 4, size: 64
    Total      8.45GB /  62.36(s) =     1.16Gb/s     2274573op/s
    Threads: 4, size: 64
    Total      8.41GB /  62.36(s) =     1.16Gb/s     2263536op/s
    Threads: 1, size: 1024
    Total     18.52GB /  62.37(s) =     2.55Gb/s      311417op/s
    ...
    ```