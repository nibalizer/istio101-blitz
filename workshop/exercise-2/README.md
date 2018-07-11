# Exercise 2 - Installing Istio on IBM Cloud Kubernetes Service
In this module, you download and install Istio.

1.  Either download Istio directly from [https://github.com/istio/istio/releases](https://github.com/istio/istio/releases) or get the latest version by using curl. Note that this workshop requires version `0.8.0`.
    ```bash
    cd ~ # or wherever you want your Istio installation to live
    curl -L https://git.io/getLatestIstio | sh -
    ```
2. Extract the installation files.
3. Add the `istioctl` client to your PATH. The `<version-number>` is in the directory name. For example, run the following command on a MacOS or Linux system:
```
export PATH=$PWD/istio-0.8.0/bin:$PATH
```
4. Change the directory to the Istio file location. (The folder was downloaded in the curl step above.)

```bash
cd istio-0.8.0
```

5. Install Istio on the Kubernetes cluster. Istio is deployed in the Kubernetes namespace `istio-system`.
```bash
kubectl apply -f install/kubernetes/istio-demo.yaml
```
> Note that this is a full installation of Istio - not a "demo" as the filename suggests.

6. Ensure that the `istio-*` Kubernetes services are deployed before you continue.
```bash
kubectl get svc -n istio-system
```
```
NAME                       CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                               AGE
grafana                    172.22.xxx.xxx   <none>        3000/TCP                                                              4d
istio-citadel              172.22.xxx.xxx   <none>        8060/TCP,9093/TCP                                                     1m
istio-egressgateway        172.22.xxx.xxx   <none>        80/TCP,443/TCP                                                        1m
istio-ingressgateway       172.22.xxx.xxx   <pending>     80:31380/TCP,443:31390/TCP,31400:31400/TCP                            1m
istio-pilot                172.22.xxx.xxx   <none>        15003/TCP,15005/TCP,15007/TCP,15010/TCP,15011/TCP,8080/TCP,9093/TCP   1m
istio-policy               172.22.xxx.xxx   <none>        9091/TCP,15004/TCP,9093/TCP                                           1m
istio-sidecar-injector     172.22.xxx.xxx   <none>        443/TCP                                                               1m
istio-statsd-prom-bridge   172.22.xxx.xxx   <none>        9102/TCP,9125/UDP                                                     1m
istio-telemetry            172.22.xxx.xxx   <none>        9091/TCP,15004/TCP,9093/TCP,42422/TCP                                 1m
prometheus                 172.22.xxx.xxx   <none>        9090/TCP                                                              1m
servicegraph               172.22.xxx.xxx   <none>        8088/TCP                                                              1m
tracing                    172.22.xxx.xxx   <pending>     80:30132/TCP                                                          1m
zipkin                     172.22.xxx.xxx   <none>        9411/TCP                                                              1m
```
  **Note: For Lite clusters, the istio-ingressgateway service will be in `pending` state with no external ip. That is normal.**

7. Ensure the corresponding pods `istio-citadel-*`, `istio-ingressgateway-*`, `istio-pilot-*`, and `istio-policy-*` are all in **`Running`** state before you continue.
```
kubectl get pods -n istio-system
```
```
NAME                                        READY     STATUS    RESTARTS   AGE
grafana-cd99bf478-6xjv5                     1/1       Running     0          9h
istio-citadel-ff5696f6f-jsrtj               1/1       Running     0          9h
istio-cleanup-old-ca-dxtkn                  0/1       Completed   0          9h
istio-egressgateway-58d98d898c-j2zp9        1/1       Running     0          9h
istio-ingressgateway-6bc7c7c4bc-bqhq2       1/1       Running     0          9h
istio-mixer-post-install-fz7hj              0/1       Completed   0          9h
istio-pilot-6c5c6b586c-fb8qn                2/2       Running     0          9h
istio-policy-5c7fbb4b9f-ntf85               2/2       Running     0          9h
istio-sidecar-injector-dbd67c88d-nvskh      1/1       Running     0          9h
istio-statsd-prom-bridge-6dbb7dcc7f-7rskf   1/1       Running     0          9h
istio-telemetry-54b5bf4847-dcd2j            2/2       Running     0          9h
istio-tracing-67dbb5b89f-28zfs              1/1       Running     0          9h
prometheus-586d95b8d9-fj6mq                 1/1       Running     0          9h
servicegraph-6d86dfc6cb-v6cmz               1/1       Running     0          9h
```

Before your continue, make sure all the pods are deployed and **`Running`**. If they're in `pending` state, wait a few minutes to let the deployment finish.

Congratulations! You successfully installed Istio into your cluster.

#### [Continue to Exercise 3 - Deploy Guestbook with Istio Proxy](../exercise-3/README.md)
