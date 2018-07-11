
#### Grafana

1. Establish port forwarding from local port 3000 to the Grafana instance:
   ````
   kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000
   ````
   
2. Due to a known Istio dashboard performance issue with 0.8, browse to http://localhost:3000, and install the following dashboards:
   1. Click on the `+` sign on the top left side, then click on the `import` menu
   2. On the resulting page, click the `update .json File` button.
   3. Navigate to the workshop/dashboards directory, and select the `istio-mesh-dashboard.json` file.
   Repeat the above steps for the `istio-tcp-service-dashboard.json` and `istio-http-grpc-service-dashboard.json` files.

3. Browse to http://localhost:3000 and navigate to the Istio Mesh Dashboard by clicking on the Home menu on the top left.


#### Prometheus

1. Establish port forwarding from local port 9090 to the Prometheus instance.

   ```console
   kubectl -n istio-system port-forward \
     $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') \
     9090:9090
   ```
2. Browse to http://localhost:9090/graph, and in the “Expression” input box, enter: istio_request_count. Click Execute.

