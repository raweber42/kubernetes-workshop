# kubernetes-workshop
This repository is meant to be the starting point for students who are attending the kubernetes workshop at 42 wolfsburg.

## Prereqisites:
- [docker](https://docs.docker.com/get-docker/) installed
- [golang](https://go.dev/dl/) installed
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) installed

## Create a local kubernetes cluster with kind
- run `go install sigs.k8s.io/kind@v0.23.0` to download kind
- run `kind create cluster --name first-k8s-cluster` to create a local kubernetes cluster in a docker container with [kind](https://kind.sigs.k8s.io/)
- run `docker ps` to see the created [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) container which is being used to interact with the kubernetes API
- run `kind get nodes` or `kubectl get nodes` to see that the above container represents a node in your local kubernetes cluster
- for now, you can delete your cluster with the command `kind delete cluster --name first-k8s-cluster`


## Deploy a NGINX webserver on a configured cluster
- when creating a kind cluster, we can pass a config file like `kind-cluster.yaml` in order to add certain configurations for e.g. accessing the cluster right from the start.
- create a fresh cluster with `kind create cluster --name second-k8s-cluster --config kind-cluster.yaml`
- have a look at the `webserver.yaml` file, this is the webserver we will create and expose on port 80
- run `kubectl apply -f webserver.yaml` to deploy the webserver to your cluster
- run `kubectl get deployments` to see that the deployment was successful
- run `kubectl get pods` to see that the application us up and running in the pod
- run `kubectl get services` to see that the service which will route traffic to our pod later
- run `kubectl exec -it <POD_NAME> -- bash` to log into that container
- inside the container, run `curl localhost:80` to see that the nginx server is serving traffic as expected
- run `exit` to leave the container

## Expose your webserver to the outside world (your computer)
- you can try to access your webserver from your laptops browser, but this will fail, as you didn't tell kubernetes to expose anything to outside of your cluster yet. It's similar to exposing ports of a docker container.

- we need something like a driver for our cluster through which it is technically enabled to receive traffic from the outside world
- we choose the popular [NGINX ingress controller](https://kubernetes.github.io/ingress-nginx/)

- next, we create an [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) which will manage external access to the services in our cluster
- run `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml` to create the ingress controller (fyi: on GCP/AWS/Azure this is not needed, as ingress controllers are handled by the cloud provider)
- run `k get all -n ingress-nginx` to see all the created resources (NOTE: the `-n` flag indicated that we look in the `ingress-nginx` _namespace_. Have a look at the concept of namespaces in kubernetes [here](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/))
- lastly, we define ingress rules for our cluster, such that our ingress controller knows, which traffic is allowed to enter the cluster. these rules are defined in `ingress.yaml`
- run `kubectl apply -f ingress.yaml` to apply the rules to your cluster (NOTE: the previous step takes a minute or two, just wait a bit and try again in case you get an error)
- run `kubectl get ingress` to see the newly created ingress and that it exposes port 80 of the cluster to the outside world!
- now your webserver is successfully exposed to the outside world of your cluster!
- run `curl localhost:80` to see that you can access the server from your laptop. try the same address from your browser!

## Scale up your NGINX instance
- with kubernetes we can easily make changes to our cluster and e.g. scale our replicas (amount of instances of a server in our case) up and down. This can be done via `kubectl` (`kubectl scale deployment <deployment-name> --replicas=3`), where the deployment name is `nginx` in our case. A more git-ops like approach and best practice is to put this configuration into our yaml files and check them into git. Change the `replicas` field in our `webserver.yaml` file to `2` and run `kubectl apply -f webserver.yaml`. This way you make use of the declarative approach of kubernetes!


## Cleanup
- once you are done, you can delete your cluster with the command `kind delete cluster --name second-k8s-cluster`
