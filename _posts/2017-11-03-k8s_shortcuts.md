---
layout: post
title: K8s Shortcuts
---

Here are my favorite custom shortcuts for my daily work with Kubernetes. I'm using oh-my-zsh. First, some simple aliases:

```sh
alias k='kubectl'
alias kdel='kubectl delete'
alias klog='kubectl logs'
alias klf='kubectl logs -f'
alias kaf='kubectl apply -f'
```

Then some functions to switch between clusters and namespaces:

```sh
# list available contexts
alias kcontext="kubectl config get-contexts"
# switch between different kubernetes contexts
alias kc="kubectl config use-context"
# set default namespace in current context
function kns() {kubectl config set-context $(kubectl config current-context) --namespace=${1}}
```

The last one is especially valueable, since the kubernetes auto-completion only works within the selected namespace.

If I want to merge multiple kubernetes config-files I do `kconfmerge $(pwd)/newCluster.config > ~/.kube/conf` using this function in background:

```sh
function kconfmerge() {export KUBECONFIG=~/.kube/config:${1}; kubectl config view}
```

If I am to lazy to switch between namespaces I use these commands:
```sh
# List pods by label
function kget() {kubectl get pods -l "$@" --all-namespaces}

# Get pods with namespace
function kl() {kubectl get pods -l $@ --all-namespaces -o json | jq -r '.items[].metadata | "\(.name) -n \(.namespace)"'}
# Jump into a pod
function kex() {kubectl exec -it "$@" sh}
```

This is especially valueable in combination using something like: `kex $(kl app=gitlab)`, this way you can get into pods ignoring their random suffix.

Sometimes I also want to select pods just by their name. This can be improved using zsh's autocompletion. Therefore add the follwing file at `~/.zsh/completion/_kex`:
```sh
#compdef kex

_arguments "1: :($(kubectl get pods | awk '{if(NR>1) print $1}' | tr '\n' ' '))"
```

Finally some helpers to output some information I often need

```sh
# Get public IPs of ingress
function kdns() {k get ingress -o json | jq '.items[] | (.spec.rules[].host + "  (" + .metadata.name + ")" )'}
# Get public IPs of services
function ksvc() {kubectl get svc -o json | jq '.items[] | (.metadata.name +": " +.spec.clusterIP)'}
# get node of pod by label
function knode() {kubectl get node $(kubectl get pods -l app=${1} -n kube-system -o json |  jq -r '.items[].spec.nodeName') -o json | jq '{id: .spec.providerID, status: .status.volumesAttached}'}
```

Further helpers and lookup things regarding K8s I also list in my personal [k8s-cheatsheet](https://gitlab.com/kreiling/cheatsheets/blob/master/Kubernetes.md).
