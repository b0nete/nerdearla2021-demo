# nerdearla2021

## 1- Deploy minikube cluster.
**Multiple nodes.**  (Each one 2cores and 2048mb)
- `minikube start --nodes 2 -p nerdearla`

**Single node.** (With 4cores and 4096mb)
- `minikube config set memory 4096`
- `minikube config set cpus 4`
- `minikube start -p nerdearla`

## 2- Deploy metrics-server.
**Source:** https://github.com/bitnami/charts/tree/master/bitnami/metrics-server
- `helm repo add bitnami https://charts.bitnami.com/bitnami`
- `helm install metrics-server bitnami/metrics-server --values ./2-metrics-server/values-minikube.yaml --namespace metrics-server --create-namespace`

## 3- Deploy kube-prometheus-stack.
**Source:** https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

- `helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`
- `helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --values ./3-kube-prometheus-stack/values-minikube.yaml --namespace kube-prometheus-stack --create-namespace`

## 4- Deploy InfluxDB .
**Source:** https://github.com/bitnami/charts/tree/master/bitnami/influxdb/#installing-the-chart)

> Use influxdb 1.8, influxdb 2 is not compatible with k6 tests
> (https://community.k6.io/t/influxdb-couldnt-write-stats-unauthorized/1514/3)
- `helm repo add bitnami https://charts.bitnami.com/bitnami`
- `helm install influxdb bitnami/influxdb --values ./4-influxdb/values-minikube.yaml --namespace influxdb --create-namespace`

> After deploy, we have to enter manually to pod and create
> influxdb database for use with k6 tests and grafana.

- `influx -username 'admin' -password 'Sjkl21hr6ds'`
- `CREATE DATABASE "primary"`

## 5- Deploy k6s operator.
**Source:** 
 1. https://k6.io/blog/running-distributed-tests-on-k8s/
 2. https://k6.io/docs/es/visualizacion-de-resultados/influxdb-+-grafana/

- Clone https://github.com/grafana/k6-operator repository and change folder name to '5-k6-operator'
- `cd 5-k6-operator && make deploy && cd ..` (This deploy k6-operator in k6-operator-system namespace)

  

## 6 - Deploy Sloth.
**Source:** https://sloth.dev/
You can read more about sloth here: https://itnext.io/slos-should-be-easy-say-hi-to-sloth-9c8a225df0d4

- `cd 6-sloth/helm/ && helm install sloth . --namespace sloth --create-namespace && cd ../../`
- Install sloth CLI (https://sloth.dev/introduction/install/#building-from-source-code)
- `kubectl apply -f ./6-sloth/slo/slo.yaml` (Apply slo.yaml defined for grafana service)
- `sloth generate -i ./6-sloth/slo/slo.yaml > ./6-sloth/slo/slo-rules.yaml` (Generate PrometheusRules)
- `kubectl apply -f ./6-sloth/slo/slo-rules.yaml` (Apply prometheusRules generated with sloth CLI)

# 7 - Execute k6 tests.
> **Before to init k6-operator load tests, configure influxdb datasource and import K6 and Sloth dashboards to grafana.**

  Load influxDB datasource in grafana and import dashboards. (https://k6.io/docs/es/visualizacion-de-resultados/influxdb-+-grafana/)

**K6 Dashboards:**
 1. https://grafana.com/grafana/dashboards/2587
 2. https://grafana.com/grafana/dashboards/4411
 3. https://grafana.com/grafana/dashboards/10660
 4. https://grafana.com/grafana/dashboards/10553

**Sloth dashboard:**
 1. https://grafana.com/grafana/dashboards/14348
 2. https://grafana.com/grafana/dashboards/14643


- `kubectl apply -f ./5-k6-operator/config/samples/k6_v1alpha1_configmap.yaml` (Create configmap with desired tests and apply).
- `kubectl apply -f ./5-k6-operator/config/samples/k6_v1alpha1_k6_with_output.yaml` (Execute tests with k6-operator using previous applied file).