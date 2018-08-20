# Exercise 4 - Istio Features

Istio has a number of features it provides. In this exercise we'll provide brief overviews of some of them.

![istio architecture](../exercise-3/istio-whatisit.svg)


## mTLS

Istio can optionally set up mutual TLS between all applications inside a cluster. Istio handles issuing, authenticating, revoking, and rotating x509 certificates for each sidecar. With this, all connections between services inside the mesh will be authenticated and encrypted.


## Tracing

Istio watches all traffic between services on the mesh. When the correct headers are put in the application, tools like Zipkin and Jaeger can be leveraged to look at call traces between microservices on the mesh.

## Metrics

Istio takes the data collection peformed by `envoy` and sends them in to prometheus, enabling beautiful and insightful graphs to be created in grafana.

## Istio Ingress

Istio provides a 'front door' service for north-south traffic. It removes the need for traditional Kubernetes ingress objects and controllers. The front door service is powered by `envoy`. When this service is used, all incoming requests are tracked by Istio for reporting purposes.


## Routing Control

Istio can modify the flow of traffic between services. These are called routing rules. Simple examples are splitting traffic based on `User-Agent` header, and sending all iPhone traffic to a specific set of pods. Another example is setting 1% of all traffic to go to a set of pods. The latter example leads naturally into using Istio to do canary deploys or blue green deploys.


## More

There are more features, but this gives you an overview.

You can now continue to [Exercise 5](../exercise-5/README.md), where we'll deploy an application in an istio-enabled cluster, and explore more of Istio's features.
