# Exercise 6 - Observe service telemetry: metrics and tracing

### Challenges with microservices

We all know that microservice architecture is the perfect fit for cloud native applications and it increases the delivery velocities greatly.   Envision you have many microservices that are delivered by multiple teams, how do you observe the the overall platform and each of the service to find out exactly what is going on with each of the services?  When something goes wrong, how do you know which service or which communication among the few services are causing the problem?

### Istio telemetry

Istio's tracing and metrics features are designed to provide broad and granular insight into the health of all services. Istio's role as a service mesh makes it the ideal data source for observability information, particularly in a microservices environment. As requests pass through multiple services, identifying performance bottlenecks becomes increasingly difficult using traditional debugging techniques. Distributed tracing provides a holistic view of requests transiting through multiple services, allowing for immediate identification of latency issues. With Istio, distributed tracing comes by default. This will expose latency, retry, and failure information for each hop in a request.

You can read more about how [Istio mixer enables telemetry reporting](https://istio.io/docs/concepts/policy-and-control/mixer.html).


## Useful commands to generate load on the app

To generate 404s

```shell
while true; do sleep 0.5;  curl http://<guestbook_endpoint>/lrandomurl; echo; done
```

To generate simple 'read access' 200s

```shell
while true; do sleep 0.5;  curl http://<guestbook_endpoint>/lrange/guestbook; echo; done
```

To generate 200s, write something to the database, and cause guestbook to hit avatar

```shell
while true; do sleep 0.5; curl 'http://173.193.99.97:31667/rpush/guestbook/Spencer/CoolWebsite'; echo; done
```
## Open source and built in monitoring features of Istio

### Configure Istio to receive telemetry data

1. Verify that the Grafana, Prometheus, ServiceGraph and Jaeger add-ons were installed successfully. All add-ons are installed into the `istio-system` namespace.
   ```console
   kubectl get pods -n istio-system

   kubectl get services -n istio-system
   ```

2. Configure Istio to automatically gather telemetry data for services that run in the service mesh.
   1. Go to the exercise-7 directory
      ```bash
      cd workshop/exercise-7
      ```

   2. Create a rule to collect telemetry data.
      ```bash
      istioctl create -f guestbook-telemetry.yaml
      ```

### Jaeger

1. Browse to your Jaeger service, after finding the service endpoint:

      ```sh
      kubectl get svc tracing -n istio-system
      ```

          Example:
          ```
          $ kubectl get svc tracing -n istio-system
            NAME      TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
            tracing   LoadBalancer   172.21.35.209   <pending>       80:30217/TCP   29m
          ```

Access the tracing service via node port, using the same technique as described earlier for discovering the node port for the guestbook service. The ip is the same, and the port is seen as the second number in the output of the above command.

2. From the **Services** menu, select either the **guestbook** or **avatar** service.

3. Scroll to the bottom and click on **Find Traces** button to see traces


#### Service Graph

1. Establish port forwarding from local port 8088 to the Service Graph instance:

   ```console
   kubectl -n istio-system port-forward \
     $(kubectl -n istio-system get pod -l app=servicegraph -o jsonpath='{.items[0].metadata.name}') \
     8088:8088
   ```

2. Browse to http://localhost:8088/dotviz

3. Browse to http://localhost:8088/dotviz?filter_empty=true&time_horizon=30s

4. Browse to http://localhost:8088/force/forcegraph.html

Congratulations! You installed Istio to Kubernetes, deployed Guestbook, and used some of Istio's features.
