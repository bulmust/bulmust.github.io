---
title: Local Kubernetes Cluster (kind)
date: 2023-09-11 09:04:04 +0300
tags: ["k8s", "local", "myk8s", "kind"]
categories: ["kubernetes", "myk8s"]
toc: true
math: true
mermaid: true
---

<!--## Install Loki-Distributed-->

<!--1. Loki helm chart is [here](https://github.com/grafana/loki/tree/main/production/helm/loki)-->
<!--2. Requirements-->
<!--   1. <https://charts.min.io/>, minio, version 4.0.12-->
<!--   2. <https://grafana.github.io/helm-charts>, grafana-agent-operator, version 0.2.16-->

<!--## Install Grafana Agent-->

<!--1. Grafana Agent helm chart is [here](https://github.com/grafana/agent/tree/main/operations/helm/charts/grafana-agent)-->
<!--2. Run the following commands-->
<!--```sh-->
<!--# Add grafana-agent-operator helm repo-->
<!--helm repo add grafana-agent https://grafana.github.io/agent-->
<!--# Update helm repo-->
<!--helm repo update-->
<!--# Check grafana-agent-operator helm chart and version-->
<!--helm search repo grafana-agent/grafana-agent-->
<!--```-->
<!--3. Default grafana-agent-operator helm chart values for version **0.24.0** is [here](/assets/charts/values-default/def-grafana-agent-0.24.0.yaml).-->
<!--4. Override grafana-agent-operator helm chart values for version **0.24.0** is [here](/assets/charts/values-override/ovr-grafana-agent-0.24.0_v1.yaml).-->
<!--  5. Install grafana-agent helm chart -->-->
<!--```sh-->
<!--helm upgrade --install grafana-agent grafana/grafana-agent --version 0.24.0 -n monitoring --create-namespace -f /assets/charts/values-override/ovr-grafana-agent-0.24.0_v1.yaml-->
<!--```-->

## Install Grafana

1. Grafana helm chart is [here](https://github.com/grafana/helm-charts/tree/main/charts/grafana)
2. Run the following commands
```sh
# Add grafana helm repo
helm repo add grafana https://grafana.github.io/helm-charts
# Update helm repo
helm repo update
# Check grafana helm chart and version
helm search repo grafana/grafana
```
3. Default grafana helm chart values for version **6.59.4** is [here](/assets/charts/values-default/def-grafana-6.59.4.yaml).
4. Override grafana helm chart values for version **6.59.4** is [here](/assets/charts/values-override/ovr-grafana-6.59.4_v1.yaml).
5. Install grafana helm chart
```sh
helm upgrade --install grafana grafana/grafana --version 6.59.4 -n monitoring --create-namespace -f /assets/charts/values-override/ovr-grafana-6.59.4_v1.yaml
```
6. User: `taygun`, password: `taygun`

## Install Minio

### Minio Operator

1. Minio helm chart is [here](https://github.com/minio/operator/tree/master/helm/operator).
2. Run the following commands
```sh
# Add minio helm repo
helm repo add minio https://operator.min.io/
# Update helm repo
helm repo update
# Check minio helm chart and version
helm search repo minio/minio-operator
```
3. Default minio helm chart values for version **4.3.7** is [here](/assets/charts/values-default/def-minio-operator-4.3.7.yaml)
4. Override minio helm chart values for version **4.3.7** is [here](/assets/charts/values-override/ovr-minio-operator-4.3.7_v1.yaml)
5. Install minio helm chart
```sh
helm upgrade --install minio minio/minio-operator --version 4.3.7 -n minio --create-namespace -f /assets/charts/values-override/ovr-minio-operator-4.3.7_v1.yaml
```
6. Create secret for minio console
```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: console-sa-secret
  namespace: minio
  annotations:
    kubernetes.io/service-account.name: console-sa
type: kubernetes.io/service-account-token
EOF
```
7. Go to [http://minio-console.localhost](http://minio-console.localhost) and login with the following credentials
```sh
SA_TOKEN=$(kubectl -n minio  get secret console-sa-secret -o jsonpath="{.data.token}" | base64 --decode)
echo $SA_TOKEN
```

## Install Prometheus

1. Prometheus helm chart is [here](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus).
2. Run the following commands
```sh
# Add prometheus helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
# Update helm repo
helm repo update
# Check prometheus helm chart and version
helm search repo prometheus-community/prometheus
```
3. Default prometheus helm chart values for version **24.3.0** is [here](/assets/charts/values-default/def-prometheus-24.3.0.yaml)
4. Override prometheus helm chart values for version **24.3.0** is [here](/assets/charts/values-override/ovr-prometheus-24.3.0_v1.yaml)
5. Install prometheus helm chart
```sh
helm upgrade --install prometheus prometheus-community/prometheus --version 24.3.0 -n monitoring --create-namespace -f /assets/charts/values-override/ovr-prometheus-24.3.0_v1.yaml
```

## Creating a cluster

1. Run the following commands. See [this link](https://kind.sigs.k8s.io/docs/user/ingress/) for installation instructions.
```sh
# Create cluster
cat <<EOF | kind create cluster --name myk8s --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
# Check status
kind get clusters
# Check nodes
kubectl get nodes
```

### Deploy Ingress NGINX

1. Run the following commands
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```
2. Wait for the ingress-nginx-controller pod to be ready
```sh
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```
3. Deploy test pod with ingress
```sh
cat <<EOF | kubectl apply -f -
kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
  - command:
    - /agnhost
    - netexec
    - --http-port
    - "8080"
    image: registry.k8s.io/e2e-test-images/agnhost:2.39
    name: foo-app
---
kind: Service
apiVersion: v1
metadata:
  name: foo-service
spec:
  selector:
    app: foo
  ports:
  # Default port used by the image
  - port: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: foo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - http:
      paths:
      - pathType: ImplementationSpecific
        path: /foo(/|$)(.*)
        backend:
          service:
            name: foo-service
            port:
              number: 8080
EOF
```
3. Check the ingress. You should see the `NOW: ...` output.
```sh
curl localhost/foo
```