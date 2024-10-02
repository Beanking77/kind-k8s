# kind-k8s


# Infra overview
![kind-infra](kind-infra.png "kind-infra")

## Installation
### kind
#### For ARM64
```
$ [ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-arm64
$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/kind
```
## Setup k8s cluster
`$ kind create cluster --config kind-config.yaml`

Label workder node
```
kubectl label node kind-worker kind-worker2 node-role.kubernetes.io/worker=infra
kubectl label node kind-worker kind-worker2 node-role.kubernetes.io/infra=
kubectl label node kind-worker3 kind-worker4 node-role.kubernetes.io/worker=app
kubectl label node kind-worker3 kind-worker4 node-role.kubernetes.io/app=
```
## Setup MatalLB
enabling strict ARP
```
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```


get config file and add line to `metallb-frr.yaml`
``` 
$ wget https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-frr.yaml
```

Edit metallb-frr.yaml and add line at #2052
``` 
kubernetes.io/role: "infra"
```

```
kubectl apply -f metallb-frr.yaml
kubectl apply -f ipv4-pool.yaml
kubectl apply -f L2advertisement.yaml 
```



# Prometheus

## Setup
```
kubectl create namespace prometheus
kubectl apply -f prometheus.yaml
kubectl apply -f node-exporter-deploy.yaml
kubectl apply -f stable-kube-state-metrics.yaml
```


# Grafana

## Setup
```
docker compose up -d
```

## Dashboard setup
```
Add prometheus service EXTERNAL-IP as datasource
Import dashboard json files
```

Monitor CPU Throttling metrics

```
rate(container_cpu_cfs_throttled_seconds_total[5m])
```