# Create Cluster

```bash
kind get clusters
kind delete cluster --name dev
kind create cluster --name dev --config kind.yaml

# Verification:
kubectl cluster-info --context kind-dev
kubectl get nodes -o wide

kind get clusters
```

# Infrastructure

## Nginx Ingress

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s

# Troubleshooting
kubectl get pods -n ingress-nginx
kubectl describe pod -n ingress-nginx ingress-nginx-admission-create-wl6c2
kubectl describe pod -n ingress-nginx ingress-nginx-controller-867bbcb78-ch797

# 0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector
k label node dev-control-plane ingress-ready=true
k label node dev-worker ingress-ready=true
k label node dev-worker2 ingress-ready=true
```

### Testing

```bash
kubectl apply -f ./ingress
curl http://localhost:10080/foo/hostname  # Output: foo-app    scoop install curl
Invoke-RestMethod -Uri "http://localhost:10080/foo/hostname" -Method GET
```

## Metrics Server 

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

```

### Testing

```bash
kubectl top pod
```

### Troubleshooting

In some environments, especially in local or development environments like Kind, the Metrics Server might need additional configuration to work correctly. You might need to adjust the args in the deployment to include the following flags:
```bash
kubectl edit deployment metrics-server -n kube-system

# Add or modify the args section to look like this:
spec:
  containers:
  - args:
    - --cert-dir=/tmp
    - --secure-port=4443
    - --kubelet-insecure-tls      <==== Add this
    - --kubelet-preferred-address-types=InternalIP
```

# Dev Tools

## Add namespace

```bash
kubectl create namespace dev-tools
```

## Redis

```bash
kubectl apply -f redis/redis.yaml --namespace dev-tools

# Verification
.\redis-cli.exe     # scoop install main/redis      redis-cli
get mykey   # Output: nil
setex mykey 5 "Testing"
get mykey   # Output: Testing
```

## RabbitMQ

```bash
kubectl apply -f rabbitmq --namespace dev-tools
kubectl wait --namespace dev-tools  --for=condition=ready pod --selector=app=rabbitmq --timeout=90s

# Verification
curl -v http://localhost:15672/
explorer http://localhost:15672/   # admin:admin  OR dev:dev
```

### Troubleshooting
The latest version 4.x is not suppor the ha policy definition, need remove the policy from the config or use 3.X version


## Kafka

> [Running Kafka on Kubernetes for local development with Storage class](https://dev.to/thegroo/running-kafka-on-kubernetes-for-local-development-with-storage-class-4oa9)

```bash
kubectl create namespace dev-kafka
kubectl apply -f kafka-dev -n dev-kafka

kubectl wait --namespace dev-kafka  --for=condition=ready pod --selector=service=kafka --timeout=90s

# To delete: kubectl delete -f kafka-dev

```

### Install Kowl & Testing

```bash
helm repo list
helm repo add cloudhut https://raw.githubusercontent.com/cloudhut/charts/master/archives
helm repo update

helm install -f kafka-dev-kowl/kowl-values.yaml kowl cloudhut/kowl -n dev-kafka
kubectl apply -f kafka-dev-kowl/kowl-service.yaml -n dev-kafka

# open browser to view kowl UI
explorer http://localhost:18081/topics
```