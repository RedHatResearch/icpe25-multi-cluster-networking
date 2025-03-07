# Test Setup

For the purposes of testing, we will be creating resources on two different clusters, cluster-1 and cluster-2. cluster-2 will have a service that will be exposed on cluster-1 through the multi-cluster networking solution. This service can be any kind of server like iperf3, uperf or nginx. cluster-1 will run the client pod that will send traffic to the service whose endpoints include the server.

## Submariner

### uperf

On **cluster-2**:
 - Make sure the uperf server pod is running
 - Create the service:
    ```
    $ kubectl create -f server-svc-pp.yaml
    ```
 - Export the service (it creates a `ServiceExport` object):
    ```
    $ subctl export service --namespace default server-pp
     ✓ Service exported successfully
    ```
 - Check the Status information on the ServiceExport object:
    ```
    $ kubectl -n default describe serviceexports serverpp | egrep "Message|Status"
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
 - Run the networking throughput test:
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

### netperf

On **cluster-2**:
 - Make sure the netperf server pod is running
 - Create the service:
    ```
    $ kubectl create -f netperf-svc.yaml
    ```
 - Export the service (it creates a `ServiceExport` object):
    ```
    $ subctl export service --namespace default netperf-server
     ✓ Service exported successfully
    ```
 - Check the Status information on the ServiceExport object:
    ```
    $ kubectl -n default describe serviceexports netperf-server | egrep "Message|Status"
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
    NAME             TYPE           IP    AGE
    netperf-server   ClusterSetIP         10h
    ```
 - Log into the client pod:
    ```
    $ oc rsh netperf-6dcfff7bc6-vkv9v
    ```
 - Run the networking latency test:
    ```
    sh-5.1# netperf -l 60 -H netperf-server.default.svc.clusterset.local -t TCP_RR -- -P 10000 -k rt_latency,p99_latency
    ```
