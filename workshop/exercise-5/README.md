# Exercise 5 - Perform traffic management

In this exercise, we'll upgrade the version of guestbook to v2, where we've implemented the use of an [avatar microservice](https://github.com/tobiaslins/avatar/) that automatically generates an avatar with every guestbook entry. We'll make use of Istio routing rules to do a canary deployment where the newer version is incrementally rolled out to users to minimize risk.

First, let's deploy the avatar microservice that v2 will be using. Navigate into the `guestbook/v2` directory.

```sh
cd guestbook/v2
```

### Deploy Avatar Microservice

1. Inject the Istio sidecar config into the Avatar deployment and deploy the microservice and sidecar to the cluster.
```sh
kubectl apply -f <(istioctl kube-inject -f avatar-deployment.yaml)
```

2. Create the Avatar service record (allows Guestbook:v2 to access it using the hostname `avatar`)
```sh
kubectl create -f avatar-service.yaml
```

### Deploy Guestbook v2

1. Deploy Guestbook v2 pods with Istio sidecars.
```sh
kubectl apply -f <(istioctl kube-inject -f guestbook-deployment.yaml)
```

Note that you don't need to apply another Guestbook service entry - we did that in [exercise 3](../exercise-3/README.md).

## Using rules to manage traffic
The core component used for traffic management in Istio is Pilot, which manages and configures all the Envoy proxy instances deployed in a particular Istio service mesh. It lets you specify what rules you want to use to route traffic between Envoy proxies, which run as sidecars to each service in the mesh.  Each service consists of any number of instances running on pods, containers, VMs etc.  Each service can have any number of versions (a.k.a. subsets).  There can be distinct subsets of service instances running different variants of the app binary. These variants are not necessarily different API versions. They could be iterative changes to the same service, deployed in different environments (prod, staging, dev, etc.).  Pilot translates high-level rules into low-level configurations and distributes this config to Envoy instances.  Pilot uses three types of configuration resources to manage traffic within its service mesh: Virtual Services, Destination Rules, and Service Entries.

### Virtual Services
A [VirtualService](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html#VirtualService) defines a set of traffic routing rules to apply when a host is addressed. Each routing rule defines matching criteria for traffic of a specific protocol. If the traffic is matched, then it is sent to a named [destination](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html#Destination) service (or [subset](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html#Subset) or version of it) defined in the service registry.

### Destination Rules
A [DestinationRule](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html#Destination) defines policies that apply to traffic intended for a service after routing has occurred. These rules specify configuration for load balancing, connection pool size from the sidecar, and outlier detection settings to detect and evict unhealthy hosts from the load balancing pool. Any destination `host` and `subset` referenced in a `VirtualService` rule must be defined in a corresponding `DestinationRule`.

## The Guestbook app
In the Guestbook app, there is one service: guestbook. The guestbook service has two distinct versions: the base version (version 1) and the modernized version (version 2). Each version of the service has two instances based on the number of replicas in [guestbook-deployment.yaml](https://github.com/linsun/examples/blob/master/guestbook-go/guestbook-deployment.yaml) and [guestbook-v2-deployment.yaml](https://github.com/linsun/examples/blob/master/guestbook-go/guestbook-v2-deployment.yaml). By default, prior to creating any rules, Istio will route requests equally across version 1 and version 2 of the guestbook service and their respective instances in a round robin manner. However, new versions of a service can easily introduce bugs to the service mesh, so following A/B Testing and Canary Deployments is good practice.

### A/B testing with Istio
A/B testing is a method of performing identical tests against two separate service versions in order to determine which performs better.

To begin, let's route all traffic to Guestbook v1 and prevent default Istio routing behavior.

>TODO: CD TO THE RIGHT DIRECTORY

```console
istioctl create -f virtualservice-all-v1.yaml
```

```yaml
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
            subset: v1
```

Note that we define a `destination` with the host `guestbook` and `subset` v1. However, there is no link between the v1 `subset` and the guestbook:v1 `deployment`. Let's do that - create the `DestinationRule` so we can match the v1 `subset` to the Guestbook v1 deployment.

```console
istioctl create -f guestbook-destination.yaml
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: destination-guestbook
spec:
  host: guestbook
  subsets:
    - name: v1
      labels:
        version: '1.0'
    - name: v2
      labels:
        version: '2.0'
```

In the `DestinationRule`, we define two subsets for both versions of the guestbook app, v1 and v2. Within those subsets, we provide labels which match the v1 `subset` to the label `version: '1.0'` - recall that when we deployed guestbook with the `v1/guestbook-deployment.yaml` file, we provided that exact label. The `VirtualService` created above routes 100% of all traffic to a `subset` named v1.

You can view the guestbook service UI using the IP address and port obtained in [Exercise 4](../exercise-4/README.md). Try it now by accessing it in your browser - you should see the familiar Guestbook v1.

Next, enable access to Guestbook v2 for testing purposes. Realistically, you'd want to check for a header or cookie to route testers to the new version. Instead, we'll route all Firefox browsers to Guestbook v2. To enable this, modify the original `VirtualService` rule:

```console
istioctl replace -f virtualservice-test.yaml
```

```yaml
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
    - match:
        - headers:
            user-agent:
              regex: '.*Firefox.*'
      route:
        - destination:
            host: guestbook
            subset: v2
    - route:
        - destination:
            host: guestbook
            subset: v1
```

> Note: If you want to use a differnet browser, replace Firefox with Chrome, Safari, etc. based on the user-agent of that browser.

In Istio `VirtualService` rules, there can be only one rule for each service and therefore when defining multiple [HTTPRoute](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html#HTTPRoute) blocks, the order in which they are defined in the yaml matters.  Hence, the original `VirtualService` rule is modified rather than creating a new rule.  With the modified rule, incoming requests originating from `Firefox` browsers will go to the newer version of guestbook.  All other requests fall-through to the next block, which routes all traffic to the original version of guestbook.  

Test this now by accessing the guestbook service UI you obtained earlier using a Firefox browser. Trying this in any other browser will serve a v1 guestbook.

### Canary deployment
In `Canary Deployments`, newer versions of services are incrementally rolled out to users to minimize the risk and impact of any bugs introduced by the newer version.  To begin incrementally routing traffic to the newer version of the guestbook service, modify the original `VirtualService` rule:

```console
istioctl replace -f virtualservice-80-20.yaml
```

```yaml
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
            subset: v1
          weight: 80
        - destination:
            host: guestbook
            subset: v2
          weight: 20
```

In the modified rule, the routed traffic is split between two different subsets of the guestbook service.  In this manner, traffic to the modernized version 2 of guestbook is controlled on a percentage basis to limit the impact of any unforeseen bugs.  This rule can be modified over time until eventually all traffic is directed to the newer version of the service.

### Implementing circuit breakers with destination rules
Istio `DestinationRules` allow users to configure Envoy's implementation of [circuit breakers](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/circuit_breaking).  Circuit breakers are critical for defining the behavior for service-to-service communication in the service mesh.  In the event of a failure for a particular service, circuit breakers allow users to set global defaults for failure recovery on a per service and/or per service version basis.  Users can apply a [traffic policy](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html#TrafficPolicy) at the top level of the `DestinationRule` to create circuit breaker settings for an entire service, or it can be defined at the subset level to create settings for a particular version of a service.

Depending on whether a service handles [HTTP](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html#ConnectionPoolSettings.HTTPSettings) requests or [TCP](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html#ConnectionPoolSettings.TCPSettings) connections, `DestinationRules` expose a number of ways for Envoy to limit traffic to a particular service as well as define failure recovery behavior for services initiating the connection to an unhealthy service.

## Further reading
* [Istio Concept](https://istio.io/docs/concepts/traffic-management/)
* [Istio Rules API](https://istio.io/docs/reference/config/istio.networking.v1alpha3.html)
* [Istio V1alpha1 to V1alpha3 Converter Tool](https://istio.io/docs/reference/commands/istioctl.html#istioctl%20experimental%20convert-networking-config)
* [Istio Proxy Debug Tool](https://istio.io/docs/reference/commands/istioctl.html#istioctl%20proxy-config)
* [Traffic Management](https://blog.openshift.com/istio-traffic-management-diving-deeper/)
* [Circuit Breaking](https://blog.openshift.com/microservices-patterns-envoy-part-i/)
* [Timeouts and Retries](https://blog.openshift.com/microservices-patterns-envoy-proxy-part-ii-timeouts-retries/)



## Questions
1. Where are routing rules defined?  Options: (VirtualService, DestinationRule, ServiceEntry)  Answer: VirtualService
1. Where are service versions (subsets) defined?  Options: (VirtualService, DestinationRule, ServiceEntry)  Answer: DestinationRule
1. Which Istio component is responsible for sending traffic management configurations to Istio sidecars?  Options: (Mixer, Citadel, Pilot, Kubernetes)  Answer: Pilot
1. What is the name of the default proxy that runs in Istio sidecars and routes requests within the service mesh?  Options: (NGINX, Envoy, HAProxy)  Answer: Envoy

#### [Continue to Exercise 6 - Telemetry](../exercise-6/README.md)
