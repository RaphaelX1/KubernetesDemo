# Kubernetes Demo

Brief POC using WSL, Docker and Minikube to verify the behavior of the most common kubernetes object types.

## Table of Contents

- [Configuring](#configuring)
- [Usage](#usage)
- [Commands](#Commands)

##  Configuring

It is necessary to have WSL, docker instaled into the WSL and Minikube Instaled.

- WSL https://learn.microsoft.com/pt-br/windows/wsl/install
- Docker into WSL 
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Installing:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Minikube https://minikube.sigs.k8s.io/docs/start/

Minikube install an sigle node local cluster already with kubectl CLI and some default features. You can check more about them in: https://minikube.sigs.k8s.io/docs/handbook/

## Overview

Our project is a .net 8 default API using Docker and Swagger. All the necessary ".yaml" files will be in the deploy folder.

> Points of interest

```
KubernetesDemo
├── deploy
│   ├── clusterIP //deployment using clusterIP Type
│   │   ├── deployment.yaml
|   |   ├── ingress.yaml
│   │   └── service.yaml
|   |   
│   └── NodePort //deployment using NodePort Type
│       ├── deployment.yaml
|       └── service.yaml
│       
├── Dockerfile
└── README.md

```

## Usage

Run minikube and install the ingress AddOn Controller to test ingress files:

```bash
minikube start 
minikube addons enable ingress
```
After that is necessary to create the namespaces configured into our deploy folder types:

```bash
kubectl create ns kubernetes-demo-cluster-ip
kubectl create ns kubernetes-demo-node-port
```
To check it just run the command:

```bash
kubectl get ns
```
It will also be necessary to build and load out docker image into minikube, the image version must be correct check the deployment file, in the root folder of the project:

```bash
docker build -t kubernetesdemo:0.0.3 .

minikube load image kubernetesdemo:0.0.3
```

Just apply the yaml files of the selected folder, at this example we will use ClusterIP folder:

```bash
cd deploy/ClusterIP
kubectl apply -f '*.yaml'
```

Expose the Ingress outside the WSL with:

```bash
minikube tunnel
```
PS: This will lock the WSL bash, to abort use ctrl + c

Finally, in your Host file, difine and save: 

```bash
127.0.0.1   kubernetes-demo.com
```
Then we will be able to navigate into the host above in your browser and access the swagger route.

PS: the NodePort type will naturally not expose a external host, to access the service internally just run:

```bash
 minikube service kubernetes-demo -n kubernetes-demo-node-port
```

## Commands
Usefull commands to help in troubleshooting.



```bash

#Navigate into windows folders inside WSL
cd /mnt/c/

#Get the minikube ip
minikube ip

#Get all <resource> of the namespace (pod|svc|ingress|configMap)
kubectl get pods -n kubernetes-demo-cluster-ip

#Get all resource inside a namespace
kubectl get all -n kubernetes-demo-cluster-ip

#Exposes an external port to access the <resource> directly (pod|svc)
kubectl port-forward service/kubernetes-demo 8081:8080 -n kubernetes-demo-cluster-ip

#Delete the <resource> of the namespace (pod|svc|ingress|configMap)
kubectl delete deployment -n  kubernetes-demo-cluster-ip

```