# Exercise 3 - Deploy the Guestbook app with Istio Proxy

The Guestbook app is a sample app for users to leave comments. It consists of a web front end, Redis master for storage, and replicated set of Redis slaves. Here are the steps to deploy the app on your Kubernetes cluster:

### Download the Guestbook app
1. Open your preferred terminal and download the Guestbook app from GitHub. (Note we are using a version customized for Dash!)
  ```sh
  git clone https://github.com/nibalizer/guestbook-dash.git
  ```
2. Navigate into the app directory.
  ```sh
  cd guestbook-dash/v1
  ```

### Create a Redis database
The Redis database is a service that you can use to persist the data of your app. The Redis database comes with a master and slave modules.

1. Create the Redis controllers and services for both the master and the slave.
  ``` sh
  kubectl create -f redis-master-deployment.yaml
  kubectl create -f redis-master-service.yaml
  kubectl create -f redis-slave-deployment.yaml
  kubectl create -f redis-slave-service.yaml
  ```
2. Verify that the Redis controllers for the master and the slave are created.
  ```sh
  kubectl get deployment
  NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  redis-master   1         1         1            1           5d
  redis-slave    2         2         2            2           5d
  ```
3. Verify that the Redis services for the master and the slave are created.
  ```sh
  kubectl get svc
  NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
  redis-master   ClusterIP      172.21.85.39    <none>          6379/TCP       5d
  redis-slave    ClusterIP      172.21.205.35   <none>          6379/TCP       5d
  ```
4. Verify that the Redis pods for the master and the slave are up and running.
  ```sh
  kubectl get pods
  NAME                            READY     STATUS    RESTARTS   AGE
  redis-master-4sswq              1/1       Running   0          5d
  redis-slave-kj8jp               1/1       Running   0          5d
  redis-slave-nslps               1/1       Running   0          5d
  ```
## Sidecar injection

In Kubernetes, a sidecar is a utility container in the pod, and its purpose is to support the main container. For Istio to work, Envoy proxies must be deployed as sidecars to each pod of the deployment. There are two ways of injecting the Istio sidecar into a pod: manually using istioctl CLI tool or automatically using the Istio Initializer. In this exercise, we will use the manual injection. Manual injection modifies the controller configuration, e.g. deployment. It does this by modifying the pod template spec such that all pods for that deployment are created with the injected sidecar.

## Install the Guestbook app with manual sidecar injection

1. Inject the Istio Envoy sidecar into the guestbook pods, and deploy the Guestbook app on to the Kubernetes cluster.
```sh
kubectl apply -f <(istioctl kube-inject -f guestbook-deployment.yaml)
```
This command will inject configuration for an Istio Envoy sidecar into the guestbook deployment file, and then deploy the guestbook app and sidecar.

2. Create the guestbook service.
```sh
kubectl create -f guestbook-service.yaml
```

3. Verify that the service was created.

```sh
kubectl get svc
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
guestbook      LoadBalancer   172.21.36.181   169.61.37.140   80:32149/TCP   5d
```
**Note: For Lite clusters, the external ip will not be avaiable. That is expected.**

4. Verify that the pods are up and running.
```sh
kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
guestbook-v1-56d98b558c-7fvd5   2/2       Running   0          5d
guestbook-v1-56d98b558c-dshkh   2/2       Running   0          5d
```

Note that each guestbook pod has 2 containers in it. One is the guestbook container, and the other is the Envoy proxy sidecar. Tip: You can validate that the pod has two containers running with ``kubectl describe pod <podname>``.


## Access the guestbook

Now it's time to view the app. Since this is a free tier cluster, we'll be using ``node port`` to access the cluster. In paid versions, ``LoadBalancers`` and ``Ingress Controllers`` are available.

1. Get the port the service is running on.

```sh
kubectl get svc
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
guestbook      LoadBalancer   172.21.36.181   <none>          80:32149/TCP   5d
...
```

In this example, the port is ``32149`` which is the number following ``80:`` in the ``PORT(S)`` section.

2. Get the external (public) IP of the cluster. For a single node, this is straightforward.


```sh
$ bx cs workers testcluster
OK
ID                                                 Public IP       Private IP    Machine Type   State    Status   Zone    Version   
kube-hou02-pa1d801f8732df4e1fb8e067cfdcfa1646-w1   173.193.99.97   10.47.64.86   free           normal   Ready    hou02   1.9.8_1517 
```

The Public IP is ``173.193.99.97``.

3. Access the guestbook application in a browser : http://<public_ip>:<nodeport>. In this example, it's http://173.193.99.97:32149/.


#### [Continue to Exercise 4 - Istio Ingress Gateway](../exercise-4/README.md)
