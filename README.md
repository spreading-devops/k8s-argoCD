# ArgoCD labs

### Environment
```
Windows 11 Pro 22H2 (OS Build 22621.1848)
WSL2 + Ubuntu 22.04.2 LTS
Docker Desktop 4.20.1
k3d v5.5.1
argocd 2.7.6
helm v3.12.1
```

## Part 1 - Create a Local Kubernetes cluster to run ArgoCD, if you don't have one already.

### Download k3d binary and put in your PATH environment variable
https://github.com/k3d-io/k3d/releases/download/v5.5.1/k3d-windows-amd64.exe

### Start a K3s kubernetes cluster using K3d (Powershell in my case)
```powershell
> .\k3d-windows-amd64.exe cluster create argocd-cluster --config .\cluster-config.yaml
```

## Part 2 - Deploy ArgoCD on local Kubernetes cluster

-> Using kubectl

### Create a namespace && Install the ArgoCD from official YAML manifest
```shell
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.7.6/manifests/install.yaml
```

-> Using HELM

### Install the ArgoCD using Helm
```shell
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm upgrade -i argo-cd argo/argo-cd -n argocd --create-namespace

# The full set of tunable parameters provided by the Argo CD Helm chart can be viewed by listing the available chart Values.
$ helm show values argo/argo-cd
```

## Part 3 - Connect to ArgoCD

### Download the ArgoCD CLI and install it (Ubuntu in my cenario)
```shell
$ wget https://github.com/argoproj/argo-cd/releases/download/v2.7.6/argocd-linux-amd64
$ sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
$ argocd version
$ argocd version --client
```

### Expose ArgoCD service
```shell
$ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

### Expose ArgoCD web UI via Port-Forward
```shell
$ kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Access ArgoCD web UI
https://localhost:8080/

### The OpenAPI specification provided by Argo CD is located at the endpoint [optional]
https://localhost:8080/swagger.json

### Get the initial admin password
```shell
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

### Login via argocd-cli
```shell
$ argocd login localhost:8080
WARNING: server certificate had error: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context 'localhost:8080' updated
```
### Changing ArgoCD admin default password
```shell
$ argocd account update-password
*** Enter password of currently logged in user (admin): 
*** Enter new password for user admin: 
*** Confirm new password for user admin: 
Password updated
Context 'localhost:8080' updated
```

## Part 4 - Deploy your first application (Public Github repository)

### Create a new Repository Connection using ArgoCD web UI

https://github.com/spreading-devops/argocd-public-repo.git

### Create a new Application using ArgoCD web UI

Specify the Repository Connection created

### Sync the Application and the Repository using ArgoCD web UI

Click in SYNC

### Expose the Service created via Port-Forward
```shell
$ kubectl port-forward svc/nginx-service -n default 8081:80
```

### Access the application
https://localhost:8081/

### Detele the Application using ArgoCD web UI

Click em DELETE e confirm.

## Part 5 - Managing multiple K8s clusters with ArgoCD

### Create another k8s cluster using K3d (Powershell in my case)
```powershell
> .\k3d-windows-amd64.exe cluster create dev-cluster --config dev-cluster-config.yaml
```

### Show .kube/config settings
```shell
$ kubectl config view
```

### Show all kube contexts
```shell
$ kubectl config get-contexts
$ kubectl config get-contexts -o name
```

### Switch kubectl between k8s clusters contexts
```shell
$ kubectl config use-context k3d-dev-cluster
$ kubectl config use-context k3d-argocd-cluster
```

### Added the new K8s cluster into ArgoCD
```shell
$ argocd cluster add k3d-dev-cluster --name dev-cluster
```

### Authenticate to ArgoCD from command line
```shell
$ argocd login localhost:8080 --insecure
```

### List all ArgoCD projects
```shell
$ argocd proj list
```

### List all Git repositories in ArgoCD
```shell
$ argocd repo list
```

### List all Apps in ArgoCD
```shell
$ argocd app list
```

### With argocd-cli, create a new project for the new local k8s cluster (namespace: default) using the existing public github repository.
```shell
$ argocd proj create dev-argocd -d https://192.168.0.35:51937,default -s https://github.com/spreading-devops/argocd-public-repo
```

## Part 6 [optional] - Create a Kubernetes Secret to store your credentials of Docker Hub

### Create a K8s Secret used to connect to a Private Docker Hub repository
```shell
$ kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v2/ --docker-username=<your-username> --docker-password=<your-password> --docker-email=<your-email>
```

## Part 7 - Deploy a new application using ArgoCD (Private Github repository)

### Create a new Private repo on Github
https://github.com/spreading-devops/argocd-private-repo

### Create a new Repository Connection to this new Private repository using ArgoCD web UI
In my case, I use a new SSH private key pair to connect ArgoCD to my private Github repository.

### Add a new source on dev-argocd Project using ArgoCD cli (Github private repo)
```shell
$ argocd proj add-source dev-argocd git@github.com:spreading-devops/argocd-private-repo.git
```

### Create a new App using ArgoCD web UI or Yaml file

### Start de deployment

Click in SYNC and wait the deployment process end

### Expose the Service created via Port-Forward
```shell
$ kubectl port-forward svc/hello-app-service -n default 8081:6000
```
### Access the application
https://localhost:8081/

## Part 8 - Enable Automatic Sync and Self-Healing

Use the ArgoCD web UI to apply this on both applications.

### Make changes in Python code, commit then on Git, build the new container image and push to Docker Hub

Use the ArgoCD web UI to follow the changes.

## Part 9 - Create a Kubernetes Cluster on Oracle Cloud Infrastrusture

### Managing multiple K8s clusters with ArgoCD (Oracle Cloud - Orace Kubernetes Engine - OKE)

https://github.com/spreading-devops/oci-oke-terraform

### Create a new project for the OKE k8s cluster (namespace: default) using the existing public github repository.
```shell
$ argocd proj create oci-oke-argocd -d https://141.148.95.90:6443,default -s https://github.com/spreading-devops/argocd-public-repo
```

### Add a new source on oci-oke-argocd Project using ArgoCD cli (Github private repo)
```shell
$ argocd proj add-source oci-oke-argocd git@github.com:spreading-devops/argocd-private-repo.git
```

### Create a K8s Secret used to connect to a Private Docker Hub repository [optional]
```shell
$ kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v2/ --docker-username=<your-username> --docker-password=<your-password> --docker-email=<your-email>
```

## Part 10 - Deploying another application with ArgoCD

### Deploying `whoami` web application using ArgoCD

https://github.com/spreading-devops/whoami

### Add a new source on dev-argocd Project using ArgoCD cli (Github public repo)
```shell
$ argocd proj add-source dev-argocd git@github.com:spreading-devops/whoami.git
```

### Switch kubectl between k8s clusters contexts
```shell
$ kubectl config use-context k3d-dev-cluster
```

### Expose `whoami` service created via Port-Forward
```shell
$ kubectl port-forward svc/whoami-service -n default 8088:80
```

### Access `whoami` application
https://localhost:8088





## Refferences
```
O'Really Book   - GitOps Cookbook
O'Really Book   - Argo CD: Up & Running
Lynda training  - Kubernetes: GitOps with ArgoCD
```

## Documentation

### K3d releases
https://github.com/k3d-io/k3d/releases

### K3d docs
https://k3d.io/v5.5.1/usage/configfile/

### Helm releases
https://github.com/helm/helm/releases