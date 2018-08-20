# Exercise 2 - Installing Istio on IBM Cloud Kubernetes Service
In this module, you download and install Istio. There are many ways to install Istio, for simplicity, we'll go forward with the fastest way to install. For production use please use the [helm chart](https://istio.io/docs/setup/kubernetes/helm-install/).

1.  Either download Istio directly from [https://github.com/istio/istio/releases](https://github.com/istio/istio/releases) or get the latest version by using curl. Note that this workshop requires version `1.0.0`.
    ```bash
    cd ~ # or wherever you want your Istio installation to live
    curl -L https://git.io/getLatestIstio | sh -
    ```
2. Extract the installation files.
3. Add the `istioctl` client to your PATH. The `<version-number>` is in the directory name. For example, run the following command on a MacOS or Linux system:
```
export PATH=$PWD/istio-1.0.0/bin:$PATH
```
4. Change the directory to the Istio file location. (The folder was downloaded in the curl step above.)
```bash
cd istio-1.0.0
```
5. Install the Custom Resource Definitions for Istio. These need to be installed before the main components.
```bash
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
```
6. Install Istio on the Kubernetes cluster. Istio is deployed in the Kubernetes namespace `istio-system`.
```bash
kubectl apply -f install/kubernetes/istio-demo.yaml
```
> Note that this is a full installation of Istio - not a "demo" as the filename suggests.

7. Ensure that the `istio-*` Kubernetes services are deployed before you continue.
```bash
kubectl get svc -n istio-system
```
```
NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                         AGE
grafana                    ClusterIP      172.21.66.155    <none>          3000/TCP                                        1m
istio-citadel              ClusterIP      172.21.125.0     <none>          8060/TCP,9093/TCP                               1m
istio-egressgateway        ClusterIP      172.21.21.33     <none>          80/TCP,443/TCP                                  1m
istio-galley               ClusterIP      172.21.200.241   <none>          443/TCP,9093/TCP                                1m
istio-ingressgateway       LoadBalancer   172.21.214.149   169.60.76.140   80:31380/TCP,443:31390/TCP,31400:<snip>         1m
istio-pilot                ClusterIP      172.21.116.146   <none>          15010/TCP,15011/TCP,8080/TCP,9093/TCP           1m
istio-policy               ClusterIP      172.21.217.159   <none>          9091/TCP,15004/TCP,9093/TCP                     1m
istio-sidecar-injector     ClusterIP      172.21.45.42     <none>          443/TCP                                         1m
istio-statsd-prom-bridge   ClusterIP      172.21.174.92    <none>          9102/TCP,9125/UDP                               1m
istio-telemetry            ClusterIP      172.21.27.106    <none>          9091/TCP,15004/TCP,9093/TCP,42422/TCP           1m
jaeger-agent               ClusterIP      None             <none>          5775/UDP,6831/UDP,6832/UDP                      1m
jaeger-collector           ClusterIP      172.21.10.26     <none>          14267/TCP,14268/TCP                             1m
jaeger-query               ClusterIP      172.21.86.167    <none>          16686/TCP                                       1m
prometheus                 ClusterIP      172.21.216.29    <none>          9090/TCP                                        1m
servicegraph               ClusterIP      172.21.218.87    <none>          8088/TCP                                        1m
tracing                    ClusterIP      172.21.110.66    <none>          80/TCP                                          1m
zipkin                     ClusterIP      172.21.140.16    <none>          9411/TCP                                        1m

```
> Note: For Lite clusters, the `istio-ingressgateway` service will be in `pending` state with no external ip. That is normal.

7. Ensure the corresponding pods `istio-citadel-*`, `istio-ingressgateway-*`, `istio-pilot-*`, and `istio-policy-*` are all in **`Running`** state before you continue.
```bash
kubectl get pods -n istio-system
```
```
NAME                                        READY     STATUS      RESTARTS   AGE
grafana-66469c4d95-jnphz                    1/1       Running     0          2m
istio-citadel-5799b76c66-9f4hx              1/1       Running     0          2m
istio-cleanup-secrets-wndkx                 0/1       Completed   0          2m
istio-egressgateway-6578f84b68-zmjnb        1/1       Running     0          2m
istio-galley-5bf4d6b8f7-vhsxp               1/1       Running     0          2m
istio-grafana-post-install-njw86            0/1       Completed   0          2m
istio-ingressgateway-67995c486c-r2mvt       1/1       Running     0          2m
istio-pilot-5c778f6dfd-wd8m9                2/2       Running     0          2m
istio-policy-8667975f76-8gnc8               2/2       Running     0          2m
istio-sidecar-injector-5b5fcf4df6-tpmgt     1/1       Running     0          2m
istio-statsd-prom-bridge-7f44bb5ddb-zrrvv   1/1       Running     0          2m
istio-telemetry-7d87746bbf-6nfsl            2/2       Running     0          2m
istio-tracing-ff94688bb-xd6h6               1/1       Running     0          2m
prometheus-84bd4b9796-7dwlc                 1/1       Running     0          2m
servicegraph-7875b75b4f-vj98l               1/1       Running     0          2m
```

Before your continue, make sure all the pods are deployed and **`Running`**. If they're in `pending` state, wait a few minutes to let the deployment finish.

Congratulations! You successfully installed Istio into your cluster.

#### [Continue to Exercise 3 - Deploy Guestbook with Istio Proxy](../exercise-3/README.md)
