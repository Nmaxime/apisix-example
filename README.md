## Instructions

- Install Kind (Kubernetes in Docker)
- Create a kind cluster
  - Pull the kind image: `docker pull kindest/node:v1.25.2@sha256:9be91e9e9cdf116809841fc77ebdb8845443c4c72fe5218f3ae9eb57fdb4bace`
  - Then run: `kind create cluster --config ./kind-cluster-config.yaml`
  
- Build the docker image at the root folder of the project with `docker build . -t learnk8s/challenge:0.1.0`
- Add the build image to kind: `kind load docker-image learnk8s/challenge:0.1.0`

- Run: `helm repo add bitnami https://charts.bitnami.com/bitnami`
- Run: `helm repo add apisix https://charts.apiseven.com`

### Apisix Ingress

In `charts/apisix` folder, run:
- `helm dependency update`
- `helm upgrade --install -n ingress-apisix --create-namespace ingress-apisix .`

Verify your installation created 5 services: `kubectl get service --namespace ingress-apisix` 

### Hello application

In `charts/hello` folder, run:
- `helm upgrade --install -n hello --create-namespace hello .`

Verify your hello pod is running: `kubectl get pod --namespace hello`.
You should see a status "Running" in the output.

#

## Verify all is working

### Check the hello route configuration

Note that the API KEY is the default one, it is not intented for production usage.

`kubectl -n ingress-apisix exec -it $(kubectl get pods -n ingress-apisix -l app.kubernetes.io/name=apisix -o name) -- curl "http://127.0.0.1:9180/apisix/admin/routes" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1"`

### Check the hello route response

Run:

`kubectl exec -it  $(kubectl get pods -n ingress-apisix -l app.kubernetes.io/name=apisix -o name) -- curl http://127.0.0.1:9080/ -H 'Host: localhost'` 

Expected output: `Hello Nmaxime!`

### Call the ingress from the outside

Get the kind-worker node IP:

`WORKER_IP=$(kubectl get no kind-worker -o jsonpath='{.status.addresses[0].address}')`

Get the node port of the API gateway service:

`GATEWAY_PORT=$(kubectl get svc ingress-apisix-gateway -o=jsonpath='{.spec.ports[0].nodePort}')`

Then verify you get the `Hello Nmaxime!` response from the hello service:

`curl $WORKER_IP:$GATEWAY_PORT/ -H "Host: localhost"`