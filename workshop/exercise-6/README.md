# Exercise 6 - Observe service telemetry: metrics and tracing

### Challenges with microservices

We all know that microservice architecture is the perfect fit for cloud native applications and it increases the delivery velocities greatly.   Envision you have many microservices that are delivered by multiple teams, how do you observe the the overall platform and each of the service to find out exactly what is going on with each of the services?  When something goes wrong, how do you know which service or which communication among the few services are causing the problem?

### Istio telemetry

Istio's tracing and metrics features are designed to provide broad and granular insight into the health of all services. Istio's role as a service mesh makes it the ideal data source for observability information, particularly in a microservices environment. As requests pass through multiple services, identifying performance bottlenecks becomes increasingly difficult using traditional debugging techniques. Distributed tracing provides a holistic view of requests transiting through multiple services, allowing for immediate identification of latency issues. With Istio, distributed tracing comes by default. This will expose latency, retry, and failure information for each hop in a request.

You can read more about how [Istio mixer enables telemetry reporting](https://istio.io/docs/concepts/policy-and-control/mixer.html).

### Adding Datadog to Kubernetes

To begin monitoring Istio using Datadog, you must first monitor Kubernetes.

1. Start by setting up the Datadog RBAC role, service account, and role binding. Note that you can easily install all the yaml files in a directory by passing the directory name to `kubectl`.
   ```console
   kubectl apply -f datadog/rbac
   ```
2. Next, retrieve your Datadog API key (``DD_API_KEY``) from [the Datadog Kubernetes Integration page](https://app.datadoghq.com/account/settings#agent/kubernetes) and place it in ``datadog/datadog-agent.yaml``. Note, you can also find your API key on the [Datadog APIs page](https://app.datadoghq.com/account/settings#api), but this is a good opportunity to examine the default Datadog Kubernetes yaml file.

   ```console
   vim datadog/datadog-agent.yaml
   ```

3. Next, create the Datadog Agent [daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
   ```console
   kubectl apply -f datadog/datadog-agent.yaml
   ```

3. Once the Datadog Agent is running, you'll see an Agent start event in your [Datadog Events Stream](https://app.datadoghq.com/event/stream). You'll also be able to view the containers running in your Kubernetes cluster in the [Datadog Container Map](https://app.datadoghq.com/infrastructure/map?node_type=container) and [Containers List](https://app.datadoghq.com/containers).

### Monitoring Istio with Datadog

Datadog can collect Istio metrics in two ways:

1. Using the [DogStatsD adapter](https://istio.io/docs/reference/config/policy-and-telemetry/adapters/datadog/) that comes with Istio to send DogStatsD metrics to the Datadog Agent.
2. Using the [Istio integration](https://docs.datadoghq.com/integrations/istio/) built into the Datadog Agent to scrape exposed Prometheus endpoints in Istio.

In this exercise we'll use the second method to monitor Istio. In [Exercise 2](../exercise-2/README.md), we installed Istio and started the Prometheus service, so we're already half-way done!

1. Create a [ConfigMap](https://docs.datadoghq.com/agent/basic_agent_usage/kubernetes/#configmap) to enable the Datadog-Istio integration.
   ```console
   kubectl apply -f datadog/istio-config.yaml
   ```
2. Again, retrieve your Datadog API key (``DD_API_KEY``) from [the Datadog APIs page](https://app.datadoghq.com/account/settings#api) and place it in ``datadog/datadog-agent-istio.yaml``. You can also copy it from ``datadog/datadog-agent.yaml`` if you prefer.

   ```console
   vim datadog/datadog-agent-istio.yaml
   ```

3. Update the Datadog Agent daemonset to use your ConfigMap.
   ```console
   kubectl apply -f datadog/datadog-agent-istio.yaml
   ```

4. In order to run the daemonset changes, find the running Datadog Agent pod.
   ```
   kubectl get pods
   NAME                            READY     STATUS    RESTARTS   AGE
   datadog-agent-zkd72             1/1       Running   0          1h
   guestbook-v1-f85d87ddc-572zp    2/2       Running   1          3h
   ```

   In this example, the pod is `datadog-agent-zkd72`. Next, delete the running pod. The daemonset will automatically start a new pod using the updated configuration.
   ```
   kubectl delete po datadog-agent-zkd72
   ```

4. Now that Datadog is monitoring Istio, let's test it out by generating some load on your guestbook application.

   1. For paid cluster, you can acceess the guestbook via the external IP for your service as guestbook is deployed as a load blanacer service.  Get the EXTERNAL-IP of the guestbook service via output below:
      ```console
      kubectl get service guestbook -n default
      ```

   2. For lite cluster, first, get the worker's public IP:
      ```console
      bx cs workers <cluster_name>
      ```

      Examples:
      ```
      bx cs workers cluster1
      ID             Public IP      Private IP      Machine Type        State    Status   Zone    Version
      kube-xxx       169.60.87.20   10.188.80.69    u2c.2x4.encrypted   normal   Ready    wdc06   1.9.7_1510*
      ```

      Second, get the node port:
      ```console
      kubectl get svc guestbook -n default
      ```

      Examples:
      ```
      $ kubectl get svc guestbook -n default
      NAME        TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
      guestbook   LoadBalancer   172.21.134.6   pending        80:31702/TCP   4d
      ```

      The node port in above sample output is `169.60.87.20:31702`

   3. Generate a small load to the app.
      ```console
      while sleep 0.5; do curl http://<guestbook_endpoint/lrange/guestbook; echo; done
      ```

   4. Log into your Datadog account and use the Metrics Explorer to view Istio metrics (e.g. [`istio.mesh.request.count`](https://app.datadoghq.com/metric/explorer?live=true&page=0&exp_metric=istio.mesh.request.count)) or create some Dashboards.


## Create RE dashboard in Datadog

RED (in monitoring) stands for Rate, Errors, and Duration. It is a quick and simple way to get some visibility. Using a datadog notebook, create the R(ate) and E(rrors) graphs on a simple dashboard. Use the following pictures as guides. Remember to add a 1 minute or 1 second average!


![rate](rate_dd.png)
![errors](errors_dd.png)


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


## Distributing Tracing, Service Graphs & the Future

The Datadog Trace Agent runs automatically as part of the Datadog Agent daemonset. For your own custom applications, you can use one of the [Datadog APM libraries](https://docs.datadoghq.com/developers/libraries/#apm-tracing-client-libraries) to instrument your application. However, just like Istio is a good point to collect telemetry in one place, it's also useful for easily seeing how all of your services communicate with each other. Support for Datadog APM directly in Istio is coming soon.


## Open source and built in monitoring features of Istio

### Configure Istio to receive telemetry data

1. Verify that the Grafana, Prometheus, ServiceGraph and Jaeger add-ons were installed successfully. All add-ons are installed into the `istio-system` namespace.
   ```console
   kubectl get pods -n istio-system

   kubectl get services -n istio-system
   ```

2. Configure Istio to automatically gather telemetry data for services that run in the service mesh.
   1. Go back to your v2 directory.
      ````
      cd guestbook-dash/v2
      ````

   2. Create a rule to collect telemetry data.
      ```sh
      istioctl create -f guestbook-telemetry.yaml
      ```

### Jaeger

1. Browse to your Jaeger service, after finding the service endpoint:

      ```sh
      kubectl get svc tracing -n istio-system
      ```

          Examples:
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

Congratulations! You installed Istio to Kubernetes, deployed Guestbook, upgraded it from v1 to v2 with best-practice traffic routing policies and enabled DataDog telemetry.

To learn about mTLS, go-on to the next optional exercise.

#### [Optional Exercise 7 - Security](../exercise-7/README.md)
