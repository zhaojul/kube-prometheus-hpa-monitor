# Repo description

This Repo has the following two functions:


* monitor kubernetes
* kubernetes custom hpa monitor

## How to monitor Aks/K8S

**TIPS**: The `metrics-server` is deployed by default in the `aks` cluster, and there is no need to deploy `metrics-server` again in the `azure` environment.

```
kubectl apply -f ./namespaces.yaml
kubectl apply -f ./node-exporter.yaml
kubectl apply -f ./metrics-server/0.3.6/ 
kubectl apply -f ./kube-state-metrics/
kubectl aply -f ./prometheus/
```

## Aks/K8S HPA

Customize monitoring indicators through Prometheus adaptor:

https://github.com/stefanprodan/k8s-prom-hpa


How to depoy it:

We still need to perform the steps of deploying prometheus above, and perform the following steps after completion:

```
make certs  #create secret
kubectl apply -f ./custom-metrics-api/
```


##  Prometheus federation

How does external prometheus grab monitor data of k8s-prometheus?

The prometheus pod is already running in the k8s cluster, and external access is provided through nodeport, we only need to configure remote crawling:

```
  - job_name: 'kubernetes-cluster'
    scrape_interval: 30s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job=~"kubernetes-.*"}'
    static_configs:
      - targets:
        - k8s1-node1-ip:30090
        - k8s1-node2-ip:30090
        labels:
          k8scluster: noprod-cluster
          resourcetype: 'azure-paas'
          department: 'BU1'
      - targets:
        - k8s2-node1-ip:30090
        - k8s2-node2-ip:30090
        labels:
          k8scluster: prod-cluster
          environment: 'production'
          department: 'BU1'
      ...
```

