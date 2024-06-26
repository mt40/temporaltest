Instructions that works on Ubuntu 20.04.6 LTS (Focal Fossa) inside Github Codespace with config:
```json
{
  "image": "mcr.microsoft.com/devcontainers/universal:2",
  "features": {
  }
}
```

Let's start:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

minikube config set memory 12288
minikube config set cpus 4
minikube start

eval $(minikube docker-env)
echo "`minikube ip` docker.local" | sudo tee -a /etc/hosts > /dev/null

# test
docker run hello-world

mkdir Temporal
cd Temporal
git clone https://github.com/temporalio/helm-charts.git
cd helm-charts/charts/temporal

helm dependency build

# wait for 15 mins after running this
helm install \
    --set server.replicaCount=1 \
    --set cassandra.config.cluster_size=1 \
    --set elasticsearch.replicas=1 \
    --set prometheus.enabled=false \
    --set grafana.enabled=false \
    temporaltest . --timeout 15m

# in case need to retry, run this first
helm uninstall temporaltest; kubectl delete job temporaltest-schema-setup; kubectl delete job temporaltest-es-index-setup

# test, should see positive values in column "Age" for all rows
kubectl get services

# now expose some ports, run each command in separate terminal to let it run continuously
kubectl port-forward service/temporaltest-frontend 7233:7233
kubectl port-forward service/temporaltest-web 8080:8080

# open another terminal
go install github.com/temporalio/tctl/cmd/tctl@latest
tctl --ns default namespace register -rd 3
# test
tctl --address localhost:7233 namespace list

# go back to /Temporal and download some sample code
git clone https://github.com/temporalio/samples-go.git

# try one sample
cd samples-go/helloworld
go test
go run worker/main.go

# open another terminal
go run starter/main.go. # Workflow result: Hello Temporal!

# now, you can repeat running worker and running starter steps to try other samples
````