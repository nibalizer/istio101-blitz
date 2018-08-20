# Exercise 3 - Deploy the Guestbook app with Istio Proxy

The Guestbook app is a sample app for users to leave comments. It consists of a web front end, Redis master for storage, and replicated set of Redis slaves.

In this exercise, we'll upgrade the version of guestbook to v2, where we've implemented the use of an [avatar microservice](https://github.com/tobiaslins/avatar/) that automatically generates an avatar with every guestbook entry.

### Install guestbook
1. In the terminal, with your kuberenetes credentials active, change directory to `exercise-5`:
  ```sh
  cd workshop/exercise-5
  ```
2. Apply the guestbook installation spec file
  ```sh
  kubectl apply -f guestbook.yaml
  ```

3. Validate the installation. You should see the following pods. Note you may still have a `poke` or `nginx` laying around from previous steps.

  ```sh
  kubectl get pod
  NAME                            READY     STATUS    RESTARTS   AGE
  avatar-68d84fdc87-qnnwx         2/2       Running   0          8m
  guestbook-v2-64f5fc8db8-rmkv2   2/2       Running   0          8m
  guestbook-v2-64f5fc8db8-vl5tl   2/2       Running   0          8m
  redis-master-5d8b66464f-gh4qt   2/2       Running   0          8m
  redis-slave-586b4c847c-54m5f    2/2       Running   0          8m
  redis-slave-586b4c847c-5f2dt    2/2       Running   0          8m
  ```

4. Validate the installation. Verify that the service was created.

```sh
kubectl get svc
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
guestbook      LoadBalancer   172.21.36.181   <none>          80:32149/TCP   5d
```
**Note: For Lite clusters, the external ip will not be avaiable. That is expected.**


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


#### [Continue to Exercise 6 - Tracing](../exercise-6/README.md)
