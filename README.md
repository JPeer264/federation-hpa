## Apollo Federation Demo

This repository is a demo of using Apollo Federation to build a single schema on top of multiple services. The microservices are located under the [`./services`](./services/) folder and the gateway that composes the overall schema is in the [`gateway.js`](./gateway.js) file.

### Installation

To run this demo locally, clone the repository and run the following commands:

```sh
npm install
```

This will install all of the dependencies for the gateway and each underlying service.

```sh
npm run start-services
```

This command will run all of the microservices at once. They can be found at http://localhost:4001, http://localhost:4002, http://localhost:4003, and http://localhost:4004.

In another terminal window, run the gateway by running this command:

```sh
npm run start-gateway
```

This will start up the gateway and serve it at http://localhost:4000

### What is this?

This demo showcases four partial schemas running as federated microservices. Each of these schemas can be accessed on their own and form a partial shape of an overall schema. The gateway fetches the service capabilities from the running services to create an overall composed schema which can be queried.

To see the query plan when running queries against the gateway, click on the `Query Plan` tab in the bottom right hand corner of [GraphQL Playground](http://localhost:4000)

To learn more about Apollo Federation, check out the [docs](https://www.apollographql.com/docs/apollo-server/federation/introduction)

### Run it on minikube locally

```sh
# start minikube and set it up
$ minikube start --cpus 4 --memory 6144 --vm true
$ minikube addons enable ingress
$ minikube addons enable metrics-server
$ kubectl create namespace dinoverse
$ kubectl config set-context --current --namespace=dinoverse

# generate SSL certificates for ingress
$ openssl genrsa -out ca.key 2048
$ openssl req -new -x509 -days 365 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt
$ openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=minikube.dev" -out server.csr
$ openssl x509 -req -extfile <(printf "subjectAltName=DNS:minikube.dev,DNS:www.minikube.dev") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
$ kubectl create secret tls minikube-dev-tls --cert=server.crt --key=server.key
$ open server.crt # trust CA

# in case the generated keys want to be deleted
$ rm ca.crt ca.srl ca.key server.crt server.csr server.key

# set up OneAgent (optional)
$ wget https://github.com/dynatrace/dynatrace-operator/releases/latest/download/install.sh -O install.sh && sh ./install.sh --api-url TENANT_API_URL --api-token API_TOKEN --paas-token PAAS_TOKEN --cluster-name "Minikube"

# add minikubes ip and add "minikube.dev" as domain to 127.0.0.1
$ minikube ip | pbcopy
$ sudo vi /etc/hosts



# build to run deployments locally
$ eval $(minikube docker-env)
$ docker build -f .docker/Dockerfile -t accounts --build-arg DIRECTORY=accounts .
$ docker build -f .docker/Dockerfile -t inventory --build-arg DIRECTORY=inventory .
$ docker build -f .docker/Dockerfile -t gateway --build-arg DIRECTORY=gateway .
$ docker build -f .docker/Dockerfile -t products --build-arg DIRECTORY=products .
$ docker build -f .docker/Dockerfile -t reviews --build-arg DIRECTORY=reviews .

# finally deploy it locally
$ helm upgrade -i dinoverse ops/k8s -f ops/k8s/values.yaml

# wait until everything is Running
$ kubectl get pods -w

# run load tests on another tab and watch the PODs scale automatically
$ npx artillery run -k artillery-config.yaml
```

### useful resource

- metric update interval https://github.com/kubernetes-sigs/metrics-server/issues/112
