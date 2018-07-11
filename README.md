# Istio 101

Istio 101 Workshop presented by IBM at Datadog Dash 7-11-2018

## Presenters:

* Sai Vennam, IBM
* Jason Yee, Datadog

## Assistants:

* Joe Sepi, IBM
* Helen Lam, IBM
* Spencer Krum, IBM

# Beyond the Basics: Istio and IBM Cloud Kubernetes Service
[Istio](https://istio.io/) is an open platform to connect, secure, and manage a network of microservices, also known as a service mesh, on cloud platforms such as Kubernetes in IBM Cloud Kubernetes Service. With Istio, You can manage network traffic, load balance across microservices, enforce access policies, verify service identity on the service mesh, and more.

In this course, you can see how to install Istio alongside microservices for a simple mock app called Guestbook. When you deploy Guestbook's microservices into an IBM Cloud Kubernetes Service cluster where Istio is installed, you inject the Istio Envoy sidecar proxies in the pods of each microservice.

**Note**: Some configurations and features of the Istio platform are still under development and are subject to change based on user feedback. Allow a few months for stablilization before you use Istio in production.

## Objectives
After you complete this course, you'll be able to:
- Download and install Istio in your cluster
- Deploy the Guestbook sample app
- Set up the Istio Ingress Gateway
- Perform simple traffic management, such as A/B tests and canary deployments
- Secure your service mesh
- Enforce policies for your microservices
- Use metrics, logging and tracing to observe services

## Prerequisites
You will set up a free [IBM Cloud account](https://ibm.biz/BdYB5d) to complete all the modules in this course. A feature code will be provided to you to enable kubernetes clusters.

Use Kubernetes 1.9.x or newer because earlier versions may require changes in manifests. This is the default on IBM Kubernetes Service (IKS)

You will [create a cluster](https://console.bluemix.net/docs/containers/container_index.html#container_index) in IKS.

If you are using a Trial IBM Cloud Account, be aware that you may encounter resource caps, especially if there are existing resources in your cluster.  During the course, if any pods remain in `Pending` status, you may need to adjust the number of `replicas` in the various deployment yamls to a value of 1, delete the deployment, and attempt the steps again.

You should have a basic understanding of containers, IBM Cloud Kubernetes Service, and Istio. If you have no experience with those, take the following courses:
1. [Get started with Kubernetes and IBM Cloud Kubernetes Service](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/)
2. [Get started with Istio and IBM Cloud Kubernetes Service](https://developer.ibm.com/courses/all/get-started-istio-ibm-cloud-container-service/)


## Workshop setup
- [Exercise 0 - Creating an IBM Cloud Account and creating a kubernetes cluster](workshop/exercise-0/README.md)
- [Exercise 1 - Accessing a Kubernetes cluster with IBM Cloud Kubernetes Service](workshop/exercise-1/README.md)
- [Exercise 2 - Installing Istio](workshop/exercise-2/README.md)
- [Exercise 3 - Deploying Guestbook with Istio Proxy](workshop/exercise-3/README.md)

## Creating a service mesh with Istio

- [Exercise 4 - Expose the service mesh with the Istio Ingress Gateway](workshop/exercise-4/README.md)
- [Exercise 5 - Perform traffic management](workshop/exercise-5/README.md)
- [Exercise 6 - Observe service telemetry: metrics and tracing](workshop/exercise-6/README.md)
- [Exercise 7 - Secure your service mesh](workshop/exercise-7/README.md)
- [Exercise 8 - Enforce policies for microservices](workshop/exercise-8/README.md)


# Where to get more?

Twitter: @gitbisect, @nibalizer, @sai_vennam

IBM Code: [https://developer.ibm.com/code](https://developer.ibm.com/code)

Twitch: [https://twitch.tv/ibmcode](https://twitch.tv/ibmcode)


## Further reading

* [Networking in istio](https://istio.io/blog/2018/v1alpha3-routing/)
* [Datadog Istio Adapter](https://docs.datadoghq.com/integrations/istio/)
