---
title: "Teams"
weight: 50
---

# Teams

## Delegation of Resource Management

When organizing teams and processes, it helps to think of Kubernetes manifests as either application or cluster related:

1. Application-related manifests are namespaced, some common ones being:
	- Deployment (What container should be running)
	- Service (Service discovery / cluster DNS)
	- ConfigMap (Define application confiuration files and env vars)

2. Some examples of cluster-concerns / administrative manifests include:
	- Namespace (A group of other resources)
	- ResourceLimit (How much cpu/memory can be used in a Namespace)
	- NetworkPolicy (What apps can talk to eachother)
	- ClusterRole/ClusterRoleBinding (Access control, what k8s users can see/do)

Typically, cluster operations teams have the ability to manage what namespaces (and associated restrictions) exist. They would then delegate access to configure application-related manifests into those namespaces to individual app teams.

## User Access Control

TODO

