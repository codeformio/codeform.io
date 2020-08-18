---
title: "Kubernetes"
weight: 10
---

# Kubernetes

## Three Key Concepts

{{< columns >}}

### 1. Universal
Kubernetes is everywhere, on every cloud provider and on-prem. It provides a consistent, open set of APIs for describing your application deployment across any provider. All concerns tangential to applications can also be specified using the same APIs: networking, security policies, access policies, etc.

<--->

### 2. Extensible
Kubernetes provides mechanisms for extending its API and functionality. This has resulted in a strong ecosystem of off-the-shelf extensions for solving specific use-cases. It also means well-defined patterns exist for encoding organization-specific operational knowledge into the platform.

<--->

### 3. Declarative
You don't tell kubernetes how to deploy your application. You declare to it what you would like to see. It works to ensure that your applications are always running according to what you specified. This makes it easy to version control and understand desired state over time.

{{< /columns >}}

## What does a manifest look like?

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3 # <-- How many instances should there be?
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2 # <-- What should be run?
        ports:
        - containerPort: 80 # <-- How should it be exposed?
```

## Who should use Kubernetes

