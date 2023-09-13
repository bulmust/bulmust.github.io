---
title: Kubernetes MiniKube
date: 2023-09-07 14:00:00 +0300
tags: ["k8s", "installations"]
categories: ["kubernetes"]
math: true
mermaid: true
toc: true
---

## Connecting to Minikube

1. Run the following commands
```sh
# Show minikube ip
minikube ip
# Connect to minikube via ssh
minikube ssh
# Show minikube docker container if you created cluster with --driver=docker flag
docker ps -a
```

## Installation

1. Before installing minikube, install `kubectl`. See [this file](/posts/k8s-kubectl-kubeadm-kubelet) for installation instructions.
2. Install docker. See [this file](/) for installation instructions.
3. Documentation and recent version of minikube is [here](https://minikube.sigs.k8s.io/docs/start/).
4. Run the following commands
```sh
# Download installation script and run
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
# Make the binary executable
sudo install minikube-linux-amd64 /usr/local/bin/minikube
# Remove the downloaded file
rm minikube-linux-amd64
# Check version
minikube version
```

## Starting minikube

1. Run the following commands
```sh
# Start minikube with docker driver
minikube start --driver=docker
# Check status
minikube status
# Check kubectl context
kubectl config current-context
```

## Stopping and deleting minikube

1. Run the following commands
```sh
# Stop minikube
minikube stop
# Delete minikube
minikube delete
# Remove minikube directory
sudo rm -rf /usr/local/bin/minikube
```
