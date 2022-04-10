# Prometheus & Grafana Installation Steps
This installtion is propogated by helm package.

## Step 1: Add Repo
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
```

## Step 2: Update helm repository
```
helm repo update
```

## Step 3: Install Prometheus Kubernetes
```
helm install prometheus prometheus-community/kube-prometheus-stack
```

## Step 4: Port Forwarding or patch service for loadBalancer
Port Forwarding
```
kubectl port-forward deployment/prometheus-grafana 3000
```
Creating LoadBalancer for service
```
kubectl patch service <service-name> -p '{"spec": {"type": "LoadBalancer"}}'
```

## Step 5: Login into Grafana
Credentials for login
```
username: admin
password: prom-operator
```
