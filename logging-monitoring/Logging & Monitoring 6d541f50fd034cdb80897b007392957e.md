# Logging & Monitoring

### Monitor Cluster Components

- Metrics Server is one of the solutions which aggregates logs about the Cluster
- The kubelet on the Nodes contain a subcomponent called cAdvisor which collects performance metrics and exposes them through the kubelet API
- For minikube use the `minikube addons enable metrics-server`, for other environments use the metrics-server deployment files to deploy Metrics Server in your Cluster
- You can use the `kubectl top node` and `kubectl top pod` commands to view performance metrics in your Cluster

### Managing Application Logs

- You can use both the `docker logs -f $CONTAINER_NAME` and `kubernetes logs -f $POD_NAME $CONTAINER_NAME` commands to view the logs of an application