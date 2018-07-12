# Exercise 4 - Expose the service mesh with the Istio Gateway

The components deployed on the service mesh by default are not exposed outside the cluster. Generally in a Kubernetes environment, external access to individual services is provided by using an external load balancer with the Kubernetes Ingress or a Node Port on individual services. However when working with an Istio service mesh, it's a better approach to use an Istio Gateway which enables us to use Istio features such as monitoring and route rules.

Configure the guestbook default route with the Istio Ingress Gateway.

```sh
cd guestbook-dash
istioctl create -f guestbook-gateway.yaml
```

  guestbook-gateway.yaml:
  ```sh
  apiVersion: networking.istio.io/v1alpha3
  kind: Gateway
  metadata:
    name: guestbook-gateway
  spec:
    selector:
      istio: ingressgateway # use istio default controller
    servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
      - "*"
  ---
  apiVersion: networking.istio.io/v1alpha3
  kind: VirtualService
  metadata:
    name: virtual-service-guestbook
  spec:
    hosts:
      - '*'
    gateways:
      - guestbook-gateway
    http:
      - route:
          - destination:
              host: guestbook
  ```
The guestbook-gateway.yaml file creates two different Istio resources - a `Gateway` and a `VirtualService`. A `Gateway` is a load balancer and specifies things like ports and protocols. A `VirtualService` defines traffic routing rules for when a host is matched by a `Gateway` as seen in the `gateways` element of the `VirtualService` In this example, we use a wildcard "*" for the hosts as we don't have a proper DNS entry yet, such as `guestbook.com`.

Next, obtain the URL to access your guestbook application. This is slightly different based on whether or not your cluster has an external load balancer. Lite clusters do not.

### Accessing your app through the Istio Gateway with a Lite cluster

Check the node port of the Istio Gateway resource.

```sh
kubectl get svc istio-ingressgateway -n istio-system

NAME            TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
istio-ingress   LoadBalancer                    *              80:31702/TCP,443:32290/TCP   10d


bx cs workers <cluster_name>

ID             Public IP      Private IP      Machine Type        State    Status   Zone    Version   
kube-xxx       169.60.87.20   10.188.80.69    u2c.2x4.encrypted   normal   Ready    wdc06   1.9.7_1510*   

```

The URL to access your Istio Gateway through the `node port` in the above example is `169.60.87.20:31702`. Make a note of this URL. You should be able to access your guestbook app with this URL - all requests are now flowing through the Istio Gateway.

### Accessing your app through the Istio Gateway with a Paid cluster

Get the **EXTERNAL-IP** of the Istio Ingress Gateway.

  ```sh
  kubectl get service istio-ingressgateway -n istio-system

  NAME            TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
  istio-ingressgateway       LoadBalancer   172.21.254.53    169.6.1.1   80:31380/TCP,443:31390/TCP,31400:31400/TCP                            2d
  ```

Make note of the external IP address that you retrieved in the previous step as it will be used to access the Guestbook app in later parts of the course.
  Example:
  ```
  http://169.61.37.141
  ```

## References:
[Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)           
[Istio Ingress](https://istio.io/docs/tasks/traffic-management/ingress.html)

#### [Continue to Exercise 5 - Traffic Management](../exercise-5/README.md)
