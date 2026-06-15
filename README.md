# Example Voting App

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:8080](http://localhost:8080), and the `results` will be at [http://localhost:8081](http://localhost:8081).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder `k8s-specifications` contains the YAML specifications of the Voting App's services.

### Deploy the application

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it):

```shell
kubectl apply -f k8s-specifications/
```

> **Note:** `kubectl apply` is the declarative approach and is preferred for managing applications. The older `kubectl create -f` command also works but `apply` allows for incremental updates.

### Access the application

The `vote` web app is available on port **31000** on each host of the cluster, and the `result` web app is available on port **31001**.

If you are using **Minikube**, you can get the direct URL to access the services:

```shell
minikube service vote-service --url
# Or open it directly in your browser
minikube service vote-service
```

### Verify your deployment

Use these commands to check the status of your resources:

```shell
# List all pods in the current namespace
kubectl get pods

# List all services and their associated ports
kubectl get svc

# View detailed status of a specific deployment
kubectl get deployments

# Watch real-time status changes
kubectl get pods -w
```

### Scale the application

Scale the number of replicas for a specific deployment. For example, to scale the `vote` app to 3 instances:

```shell
kubectl scale deployment vote-deployment --replicas=3
```

To verify the scaling:

```shell
kubectl get pods
```

### Update the application

If you make changes to the YAML specifications, re-apply them:

```shell
kubectl apply -f k8s-specifications/
```

To update the image of a specific deployment:

```shell
kubectl set image deployment/vote-deployment vote=your-new-image:tag
```

### Troubleshooting

If a pod fails to start or behaves unexpectedly:

```shell
# Get logs from a specific pod
kubectl logs <pod-name>

# Stream logs in real-time
kubectl logs -f <pod-name>

# Get detailed description of a pod
kubectl describe pod <pod-name>

# Check events in the namespace
kubectl get events --sort-by='.lastTimestamp'

# Access a pod's shell for debugging (if container has a shell)
kubectl exec -it <pod-name> -- /bin/bash
```

### Working with namespaces

By default, these commands operate in the `default` namespace. To use a separate namespace:

```shell
# Create a namespace
kubectl create namespace vote

# Deploy to the namespace
kubectl apply -f k8s-specifications/ -n vote

# List resources in the namespace
kubectl get pods -n vote

# Switch namespace permanently (optional - using kubectx or similar tools)
kubectl config set-context --current --namespace=vote
```

### Remove the application

To remove all resources created by the specifications:

```shell
kubectl delete -f k8s-specifications/
```

To delete resources in a specific namespace:

```shell
kubectl delete -f k8s-specifications/ -n vote
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

- A front-end web app in [Python](/vote) which lets you vote between two options
- A [Redis](https://hub.docker.com/_/redis/) which collects new votes
- A [.NET](/worker/) worker which consumes votes and stores them in…
- A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
- A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to deal with them in Docker at a basic level.
