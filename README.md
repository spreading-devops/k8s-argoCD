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

K3d releases
https://github.com/k3d-io/k3d/releases

K3d docs
https://k3d.io/v5.5.1/usage/configfile/

Helm releases
https://github.com/helm/helm/releases

## Steps

### Download k3d binary and put in your PATH environment variable
https://github.com/k3d-io/k3d/releases/download/v5.5.1/k3d-windows-amd64.exe

### Start a K3s kubernetes cluster using K3d (Powershell in my case)
```powershell
> .\k3d-windows-amd64.exe cluster create argocd-cluster --config .\cluster-config.yaml
```

### Create a namespace to ArgoCD resources on K8s and install the ArgoCD from official manifest
```shell
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.7.6/manifests/install.yaml
```

### Download the ArgoCD CLI and install on Ubuntu
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

### Get the initial admin password
```shell
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

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

### Login
```shell
$ argocd login localhost:8080
WARNING: server certificate had error: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context 'localhost:8080' updated
```
### Changing the admin default password
```shell
$ argocd account update-password
*** Enter password of currently logged in user (admin): 
*** Enter new password for user admin: 
*** Confirm new password for user admin: 
Password updated
Context 'localhost:8080' updated
```

### Detele the Application using ArgoCD web UI

Click em DELETE e confirm.

## Manage multiple K8s clusters with ArgoCD

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

### On ArgoCD, create a new project for the new k8s cluster (namespace: default) using the existing public github repository.
```shell
$ argocd proj create dev-argocd -d https://192.168.0.35:51937,default -s https://github.com/spreading-devops/argocd-public-repo
```

### Create a K8s Secret used to connect to a Private Docker Hub repository
```shell
$ kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v2/ --
docker-username=<your-username> --docker-password=<your-password> --docker-email=<your-email>
```

### Create a new Private repo on Github
https://github.com/spreading-devops/argocd-private-repo

### Create a new Repository Connection to this new Private repository using ArgoCD web UI
In my case, I use a new SSH private key pair to connect ArgoCD to my private Github repository.

### Add a new source on dev-argocd Project using ArgoCD cli
```shell
$ argocd proj add-source dev-argocd git@github.com:spreading-devops/argocd-private-repo.git
```

### Create a new App udsing ArgoCD web UI or Yaml file

### Start de deployment

Click in SYNC and wait the deployment process end

### Expose the Service created via Port-Forward
```shell
$ kubectl port-forward svc/hello-app-service -n default 8081:6000
```
### Access the application
https://localhost:8081/

### Make changes in Python code, commit then on Git, build the new container image and push to Docker Hub



