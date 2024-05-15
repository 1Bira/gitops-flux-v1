# Flux Repo
## Introduce
GitOps is a methodology that leverages Git as the single source of truth for declarative infrastructure and application code. It emphasizes version control, collaboration, and automation to manage infrastructure and deployments. By continuously syncing desired state with observed state, GitOps ensures consistency, repeatability, and scalability in modern software delivery pipelines.


## Design

![Design, Model Initial](/docs/init-design.png)


Gitops get a new model to deploy, The git repos is the unique information about state the cluster kubernets.


![Design, Model Initial](/docs/final-design.png)

## Preconditions
1. kubernets (kind or Minikube)
1. kustomize
1. git (github)
1. fluxCD

## Configuration

1. Install flux
```sh
curl -s https://fluxcd.io/install.sh | sudo bash

# Basic Instalation
flux install

#Complete Install
flux install --components-extra='image-reflector-controller,image-automation-controller'
```

2. Flux configuration

```sh
# Gitrepository Create
flux create source git podinfo \
  --url=https://github.com/1bira/podinfo \
  --branch=master \
  --interval=30s \
  --export > gitrepo-podinfo.yaml

# kustomization Create
flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --interval=30s \
  --export > kustomization-podinfo.yaml

# Helm respository Create
flux create source helm podinfo \
  --url https://1bira.github.io/podinfo --namespace default \
  --interval 1h \
  --export > helmrepository.yaml

```

## Implementation


Repository git needs same information to configure the gitops in flux.

information:
1. Repository url
1. branch
1. Clone interval
1. credencial secrets to clone git (if private)
1. Manifests path

Example:

```yaml
apiversion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
    name: podinfo
    namespace: flux-system
spec:
    interval: 30s
    ref:
        branch: master
    url: https://github.com/1bira/GITOPS-FLUX_V1


apiversion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
    name: podinfo
    namespace: flux-system
spec:
    interval: 30s
    path: ./kustomize
    prune: true
    sourceRef:
        kind: GitRepository
        name: podinfo
    targetNamespace: default


```



## Complements
https://github.com/weaveworks/weave-gitops

### Install weave - gitops

```sh
# install bin 
curl --silent --location "https://github.com/weaveworks/weave-gitops/releases/download/v0.38.0/gitops-$(uname)-$(uname -m).tar.gz" | tar xz -C /tmp
sudo mv /tmp/gitops /usr/local/bin
gitops version

# Manifests create to instaal weave gitops
PASSWORD="<your password>"
gitops create dashboard ww-gitops \
  --password=$PASSWORD \
  --export > weave-gitops.yaml

#Apply
kubectl apply -f weave-gitops.yaml

#portfoward
kubectl port-forward svc/ww-gitops-weave-gitops 8080:9001


```