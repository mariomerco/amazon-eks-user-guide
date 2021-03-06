# Control Plane Metrics with Prometheus<a name="prometheus"></a>

The Kubernetes API server exposes a number of metrics that are useful for monitoring and analysis\. These metrics are exposed internally through a metrics endpoint that refers to the `/metrics` HTTP API\. Like other endpoints, this endpoint is exposed on the Amazon EKS control plane\. This topic explains some of the ways you can use this endpoint to view and analyze what your cluster is doing\.

## Viewing the Raw Metrics<a name="view-raw-metrics"></a>

To view the raw metrics output, use `kubectl` with the `--raw` flag\. This command allows you to pass any HTTP path and returns the raw response\.

```
kubectl get --raw /metrics
```

Example output:

```
...
# HELP rest_client_requests_total Number of HTTP requests, partitioned by status code, method, and host.
# TYPE rest_client_requests_total counter
rest_client_requests_total{code="200",host="127.0.0.1:21362",method="POST"} 4994
rest_client_requests_total{code="200",host="127.0.0.1:443",method="DELETE"} 1
rest_client_requests_total{code="200",host="127.0.0.1:443",method="GET"} 1.326086e+06
rest_client_requests_total{code="200",host="127.0.0.1:443",method="PUT"} 862173
rest_client_requests_total{code="404",host="127.0.0.1:443",method="GET"} 2
rest_client_requests_total{code="409",host="127.0.0.1:443",method="POST"} 3
rest_client_requests_total{code="409",host="127.0.0.1:443",method="PUT"} 8
# HELP ssh_tunnel_open_count Counter of ssh tunnel total open attempts
# TYPE ssh_tunnel_open_count counter
ssh_tunnel_open_count 0
# HELP ssh_tunnel_open_fail_count Counter of ssh tunnel failed open attempts
# TYPE ssh_tunnel_open_fail_count counter
ssh_tunnel_open_fail_count 0
```

This raw output returns verbatim what the API server exposes\. These metrics are represented in a [Prometheus format](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md)\. This format allows the API server to expose different metrics broken down by line\. Each line includes a metric name, tags, and a value\.

```
metric_name{"tag"="value"[,...]} value
```

While this endpoint is useful if you are looking for a specific metric, you typically want to analyze these metrics over time\. To do this, you can deploy [Prometheus](https://prometheus.io/) into your cluster\. Prometheus is a monitoring and time series database that scrapes exposed endpoints and aggregates data, allowing you to filter, graph, and query the results\.

## Deploying Prometheus<a name="deploy-prometheus"></a>

This topic helps you deploy Prometheus into your cluster with Helm\. Helm is a package manager for Kubernetes clusters\. For more information, see [Using Helm with Amazon EKS](helm.md)\.

After you configure Helm for your Amazon EKS cluster, you can use it to deploy Prometheus with the following steps\.

**To deploy Prometheus using Helm**

1. Follow the steps in [Using Helm with Amazon EKS](helm.md) to get working `helm` and `tiller` terminal windows, so that you can install Helm charts\.

1. In the Helm terminal window, run the following commands to deploy Prometheus\.

   1. Create a Prometheus namespace\.

      ```
      kubectl create namespace prometheus
      ```

   1. Deploy Prometheus\.

      ```
      helm install stable/prometheus \
      --name prometheus \
      --namespace prometheus \
      --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2"
      ```

1. Verify that all of the pods in the `prometheus` namespace are in the `READY` state\.

   ```
   kubectl get pods -n prometheus
   ```

   Output:

   ```
   NAME                                             READY   STATUS    RESTARTS   AGE
   prometheus-alertmanager-848fb754f5-2wpbm         2/2     Running   0          85s
   prometheus-kube-state-metrics-86cbcf9b6f-drnfq   1/1     Running   0          85s
   prometheus-node-exporter-8qpcl                   1/1     Running   0          85s
   prometheus-node-exporter-czz9g                   1/1     Running   0          85s
   prometheus-node-exporter-ffsl9                   1/1     Running   0          85s
   prometheus-pushgateway-564f65fcc8-hmzp6          1/1     Running   0          85s
   prometheus-server-5b65bd569b-6wgwx               2/2     Running   0          85s
   ```

1. Use `kubectl` to port forward the Prometheus console to your local machine\.

   ```
   kubectl --namespace=prometheus port-forward deploy/prometheus-server 9090
   ```

1. Point a web browser to [localhost:9090](localhost:9090) to view the Prometheus console\.

1. Choose a metric from the **\- insert metric at cursor** menu, then choose **Execute**\. Choose the **Graph** tab to show the metric over time\. The following image shows `container_memory_usage_bytes` over time\.  
![\[Prometheus metrics\]](http://docs.aws.amazon.com/eks/latest/userguide/images/prometheus-metric.png)

1. From the top navigation bar, choose **Status**, then **Targets**\.  
![\[Prometheus console\]](http://docs.aws.amazon.com/eks/latest/userguide/images/prometheus.png)

   All of the Kubernetes endpoints that are connected to Prometheus using service discovery are displayed\.