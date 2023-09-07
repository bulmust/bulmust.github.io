---
title: Kubectl, Kubeadm, Kubelet
date: 2023-09-07 14:50:02 +0300
tags: ["k8s", "kubectl", "installations"]
categories: ["kubernetes"]
toc: true
math: true
mermaid: true
---

## Accessing Clusters

1. `kubectl` will look for a file named `config` in the `$HOME/.kube` directory. It means that if you have a file named `config` in the `$HOME/.kube` directory, you can access the cluster with `kubectl` command.
2. Another method to access a cluster is to set the `KUBECONFIG` environment variable to point to a file that contains your configuration. The following command add two kubernetes configuration to `zshrc` file. We suppose that config files in the `~` or `$HOME` directory and named `config1` and `config2`.
```sh
echo "export KUBECONFIG=~/config1:~/.kube/config2" >> ~/.zshrc
```
1. Another method to access a cluster is creating alias (to `~/.zshrc`) with setting `KUBECONFIG` enviroment variable using `export` command. The following command will create an alias named `exportK8sConfig1` for `config1` file and `exportK8sConfig2` for `config2` file. We suppose that config files in the `~` or `$HOME` directory.
```sh
alias exportK8sConfig1="export KUBECONFIG=~/config1"
alias exportK8sConfig2="export KUBECONFIG=~/config2"
```
1. Another method is combining to config files. See [here](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/#merging-kubeconfig-files).

## Useful `kubectl` Commands

1. The `k` is an alias for `kubectl` command. To do that you have to add `alias k=kubectl` to `bashrc` or `zshrc` file.

| Command | Description |
| --- | --- |
| `k get pods` | Show pods in current namespace. |
| `k get pods -A` | Show pods in all namespaces. |
| `k get pods --watch` | Show pods and watch for changes. |
| `k get pods -n <namespace-name>` | Show pods in a specific namespace. |
| `k get nodes` | Show nodes. |
| `k get nodes -o wide` | Show nodes with more details. |
| `k get namespaces` | Show namespaces. |
| `k get deployments` | Show deployments. |
| `k get services` | Show services. |
| `k get events` | Show events. |
| `k create namespace <namespace-name>` | Create namespace. |
| `k delete namespace <namespace-name>` | Delete namespace. |
| `k delete pod <pod-name>` | Delete pod. |
| `k delete node <node-name>` | Delete node. |
| `k delete deployment <deployment-name>` | Delete deployment. |
| `k delete service <service-name>` | Delete service. |
| `k describe pod <pod-name>` | Show details of a pod. |
| `k describe node <node-name>` | Show details of a node. |
| `k config get-contexts` | Show all contexts. |
| `k config use-context <context-name>` | Switch to a context. |
| `k drain <node-name>` | Drain a node. Drain means removing all workloads from a node without deleting them. |

### Shortcuts

1. You can use the following shortcuts instead of using commands. Ex. `kubectl get pods` -> `kubectl get po`.

| Commmand | Shortcut |
| --- | --- |
| `pods` | `po` |
| `nodes` | `no` |
| `namespaces` | `ns` |
| `deployments` | `deploy` |
| `services` | `svc` |
| `replicasets` | `rs` |
| `daemonsets` | `ds` |
| `statefulsets` | `sts` |
| `jobs` | `job` |
| `cronjobs` | `cj` |
| `configmaps` | `cm` |
| `secrets` | `sec` |
| `ingresses` | `ing` |
| `persistentvolumeclaims` | `pvc` |
| `persistentvolumes` | `pv` |
| `storageclasses` | `sc` |
| `horizontalpodautoscalers` | `hpa` |

1. If you installed `oh-my-zsh`, you can use the following shortcuts/alias with `kubectl` plugin.
2. For more details, see [here](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/kubectl).


## Plugins

Some useful plugins are listed below.
1. Kubectl-cost: <https://github.com/kubecost/kubectl-cost>
1. Kubectx: <https://github.com/ahmetb/kubectx>
1. flyte: <https://github.com/flyteorg/flyte>
1. Karmada: <https://github.com/karmada-io/karmada>
1. Kudo: <https://kudo.dev/>
1. Popeye: <https://popeyecli.io/>
1. Kube-capacity: <https://github.com/robscott/kube-capacity>

### krew

1. Documentation and recent version of `krew` is [here](https://krew.sigs.k8s.io/).
2. Run following script copied from [this link](https://krew.sigs.k8s.io/docs/user-guide/setup/install/).
```sh
set -x; cd "$(mktemp -d)" &&
OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
KREW="krew-${OS}_${ARCH}" &&
curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
tar zxvf "${KREW}.tar.gz" &&
./"${KREW}" install krew
```
1. Add `krew` to your path permanently (`zsh`):
```sh
echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ~/.zshrc
```
1. Close and reopen your terminal.
2. Run `kubectl krew` to check if it is installed correctly.


## Definitions

kubectl
: Command line tool for Kubernetes. It is used to manage Kubernetes clusters.

kubeadm
: Tool for creating a cluster.

kubelet
: Tool for running containers on nodes.

## Installation

1. Install `kubectl, kubeadm, kubelet` [[1]].
```sh
## Update and upgrade.
sudo apt-get update && sudo apt upgrade -y
## Install needed packages.
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg containerd ufw net-tools nano git wget
## Download the Google Cloud public signing key.
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
## Add the Kubernetes `apt` repository.
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
## Install kubeadm, kubelet and kubectl.
sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
```
1. Add `kubectl` allias to `bashrc` or `zshrc`.
```sh
# Bash
#echo "alias k=kubectl" >> ~/.bashrc
# Zsh
echo "alias k=kubectl" >> ~/.zshrc
```
   - If you have "oh-my-zsh", add kubectl plugin to `zshrc`.
   ```zsh
   nano ~/.zshrc
   ```
   - Add `kubectl` plugin to `zshrc`.
   ```sh
   plugins=(kubectl)
   ```
1. Restart terminal.
```sh
# Bash
#source ~/.bashrc
# Zsh
source ~/.zshrc
```
1. Check versions.
```sh
# kubectl version
kubectl version --client
# kubeadm version.
kubeadm version
# kubelet version.
kubelet --version
```

[1]: <https://kubernetes.io/docs/tasks/tools/> https://kubernetes.io/docs/tasks/tools/