---
title: "Exercise"
weight: 40
---

# Workflow Exercise

## Setup

Define the environment.

```sh
export ENV=non-prod
```

Create a local cluster representing this environment.

```sh
kind create cluster --name $ENV
```

## Bootstrap the Cluster

Install the GitOps engine.

```sh
kustomize build "https://github.com/codeformio/k8s-cluster-config/library/argocd?ref=$ENV" | kubectl apply -f -
```

Bootstrap the cluster configuration by installing the root ArgoCD Application.

```sh
kubectl apply -f https://raw.githubusercontent.com/codeformio/k8s-cluster-config/$ENV/deploy/$ENV/root.yaml
```

## Login to UI

The ArgoCD UI can be used to view the application. By default the password for the `admin` user is set to the server's pod name.

```sh
kubectl get pods -n argocd
```

Port forward and login by visiting [https://localhost:8080](https://localhost:8080).

```sh
kubectl port-forward -n argocd svc/argocd-server 8080:443
```

Port forward the application's port to see it respond.

```sh
kubectl port-forward -n greeter svc/hello-api 8000:80
```

In another terminal...

```sh
curl localhost:8000 -v
```

## Update the Application

TODO

## Redeploy the Application

TODO

## Cleanup

Delete the cluster.

```sh
kind delete cluster --name $ENV
```
