# ArgoCD labs

### Environment
```
Windows 11 Pro 22H2 (OS Build 22621.1848)
WSL2 + Ubuntu 22.04.2 LTS
Docker Desktop 4.20.1
k3d v5.5.1
argocd 2.7.6
```

K3d releases
https://github.com/k3d-io/k3d/releases

K3d docs
https://k3d.io/v5.5.1/usage/configfile/

## Steps

### Download k3d binary and put in your PATH environment variable
https://github.com/k3d-io/k3d/releases/download/v5.5.1/k3d-windows-amd64.exe

### Start a K3s kubernetes cluster using K3d (Powershell)
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
https://localhost:8080/

