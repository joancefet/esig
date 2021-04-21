# Implementação RocketChat e WikiJS com Kubernetes
### Para essa implementação, será utilizado a distro Linux Ubuntu 20.04 com o Cluster Kubernetes Minikube e as ferramentas helm, kubeadm, kubectl e kubelet

## Instalação 

### Instalando Minikube e suas ferramentas

Adicionando repositório Kubernetes
```shell
sudo apt install apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt --fix-broken install
```

Instalando kubelet kubeadm kubectl
```shell
sudo apt install -y kubeadm kubelet kubectl kubernetes-cni
sudo apt-mark hold kubelet kubeadm kubectl
sudo apt --fix-broken install
```
Instalando minikube
```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Instalando helm
```shell
curl -O https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
bash ./get-helm-3
helm version
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

Instalando Virtualbox(requerimento para o Minikube)
```shell
sudo apt install virtualbox
```

Iniciando cluster kubernetes
```shell
sudo kubeadm init --pod-network-cidr=10.100.0.0/16
```

Iniciando Minikube
```shell
minikube start --driver=virtualbox
```

### Aplicando os manifestes Postgres
```shell
kubectl create -f postgresql/postgres-config.yaml | set database credentials
kubectl create -f postgresql/postgres-workload.yaml | create pods, services, deployment
kubectl create -f postgresql/pgadmin-workload.yaml | create pods, services, deployment
```

### Aplicando os manifests para o WikiJS
OBS.: Alterar o IP do banco no arquivo wiki-deployment.yaml
```shell
kubectl create -f wikijs/wiki-deployment.yaml | create pods, services, deployment
kubectl create -f wikijs/wiki-ingress.yaml | create server nginx for comunication http
```

### Exportando o WikiJS para um endereço externo
```shell
minikube service wikijs
```

### Instalando RocketChat e MongoDB
```shell
helm install --set mongodb.mongodbUsername=rocketchat,mongodb.mongodbPassword=rocketchat123,mongodb.mongodbDatabase=rocketchat,mongodb.mongodbRootPassword=root rocketchat stable/rocketchat | provisioning all rocketchat cluster
```

Aplicando o port-forward na instancia RocketChat para torna-la acessível externamente
```shell
kubectl port-forward --namespace default $(kubectl get pods --namespace default -l "app.kubernetes.io/name=rocketchat,app.kubernetes.io/instance=rocketchat" -o jsonpath='{ .items[0].metadata.name }') 8888:3000 | create bind to the port 3000
```
