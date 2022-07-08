KUBERNETES MONITORING
=====================

Kubernetes demonstration stack for monitoring and logs.

# Requirements

You should first check the [requirements](requirements.md) to install before launching the demo.

# Demos

## Start a k3s or k3d cluster for the demo

### Start local k3d Kubernetes

Start a local k3d stack, exposing 3 nodeports.

**Launch cluster:**

```sh
k3d cluster create --agents 2 --port 8081:30001@agent:0 --port 8082:30002@agent:0 --port 8083:30003@agent:0 --port 8084:30004@agent:0 --port 8080:80@loadbalancer
```

**Use Kubeconfig:**
By default your default ~/.kube/config file is updated with k3d credentials. 
Otherwise, change your environment variable:

```sh
export KUBECONFIG="$(k3d kubeconfig write 'k3s-default')"
```

### Deploy Remote K3S cluster
**Preparation**
```
# Set your external IP VM for demo
export SERVER_IP=______IP______
export NEXT_SERVER_IP=______IP______
export AGENT1_IP=______IP______
export AGENT2_IP=______IP______

# Set your keypair and user for ssh login:
export KEY_LOCATION=~/.ssh/key.pem
export SUDO_USERNAME=ec2-user
```

**Install**
```
# add the master node
k3sup install --ip $SERVER_IP --ssh-key $KEY_LOCATION --user $SUDO_USERNAME --cluster

# add worker nodes
k3sup join --server-ip $SERVER_IP --ip $AGENT1_IP --ssh-key $KEY_LOCATION --user $SUDO_USERNAME

k3sup join --server-ip $SERVER_IP --ip $AGENT2_IP --ssh-key $KEY_LOCATION --user $SUDO_USERNAME

# add another master node
k3sup join --server-ip $SERVER_IP --ip $NEXT_SERVER_IP --user $SUDO_USERNAME --server --ssh-key $KEY_LOCATION
```

**Verifications:**
```
export KUBECONFIG=$(PWD)/kubeconfig

# Verification
kubectl get nodes -o wide

# check the master node
ssh $SUDO_USERNAME@$SERVER_IP -i $KEY_LOCATION

```


## Deploy Kubernetes dashboard

The Kubernetes dashboard will run on 8081 local port, and 30001 node port in internal.

```sh
# Create namespace for Kubernetes dashboard
kubectl create ns kubernetes-dashboard

# Install customized chart for nodeport and security configuration
helm install kubernetes-dashboard --namespace kubernetes-dashboard helm-charts/kubernetes-dashboard
```

**Open dashboard:**

Use **Skip** to connect to the dashboard:

```sh
# With k3d
open https://localhost:8081

# With k3s
open "https://${AGENT1_IP}:30001"
```

## Deploy Prometheus Grafana

Add the Prometheus chart repo:

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

The Grafana dashboard will run on 8082 local port, and 30002 node port in internal.

```sh
# Create Namespace
kubectl create namespace kube-prometheus-stack

# Install Helm chart
helm upgrade --install kube-prometheus-stack --namespace kube-prometheus-stack -f helm-values/kube-prometheus-stack.yaml prometheus-community/kube-prometheus-stack --version 36.2.1
```

> Doc:
> https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
> For information:
> - Default values: https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml

**Open Grafana:**

Use credentials admin / admin123 :

```sh
open http://localhost:8082
open "https://${AGENT1_IP}:30002"
```

## Deploy Elastic Stack (ElasticSearch Kibana and FileBeat log integration)

Add the Elastic chart repo:
```sh
helm repo add elastic https://helm.elastic.co
helm repo update
```

The Kibana dashboard will run on 8083 local port, and 30003 node port in internal.

**Install Elastic Stack operators:**
```sh
# Namespace for complete Elastic Stack components
kubectl create namespace elastic

# Install Elastic Stack operators which will operate the installation
helm --namespace elastic upgrade --install elastic-operator --wait elastic/eck-operator --version 2.3.0

# Install custom user 'admin' with superuser rights
kubectl apply --namespace elastic -f manifests/elastic/secret.yaml

# Install ElasticSearch
kubectl apply --namespace elastic -f manifests/elastic/elasticsearch.yaml

# Install FileBeat
kubectl apply --namespace elastic -f manifests/elastic/beat.yaml

# Install Kibana
kubectl apply --namespace elastic -f manifests/elastic/kibana.yaml
```

**Open Kibana:**

Use credentials admin / admin123 :

```
open https://localhost:8083
open "https://${AGENT1_IP}:30003"
```

> For information:
> - see values: https://github.com/elastic/cloud-on-k8s/tree/2.3.0/deploy/eck-operator
> - install Elastic Stack: https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-elasticsearch.html


## Install Jaeger


**Install cert-manager with CRDs:**
```sh
# Add JetStack repo
helm repo add jetstack https://charts.jetstack.io
helm repo update

# Install prerequisite cert-manager
helm install cert-manager jetstack/cert-manager  --namespace cert-manager  --create-namespace  --version v1.8.2  --set installCRDs=true
```

See: https://cert-manager.io/docs/installation/helm/#steps

**Install Jaeger:**
```sh
kubectl create namespace jaeger
helm -n jaeger upgrade --install jaeger -f helm-values/jaeger-values.yaml jaegertracing/jaeger
```


## Test with custom services

```sh
# client-server Namespace
kubectl create namespace client-server

# Deploy 1 server and 3 clients
helm -n client-server upgrade --install client-server ./helm-charts/client-server
```

> Infos: https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/