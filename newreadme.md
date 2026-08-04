## Install jdk and jenkins

```
sudo apt update
sudo apt install openjdk-21-jdk -y

```
### Install Jenkins

```
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update -y
sudo apt install jenkins -y
```

### Install Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy -y

### Docker Install
#Docker
curl https://get.docker.com | bash
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
newgrp docker
sudo systemctl stop docker 
sudo systemctl enable --now docker 
sudo systemctl start docker

### Install az cli

curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

### Kubectl Install
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"


### Deploy AKS
RG=Jenkins
NAME=springboot

az aks create --resource-group $RG --name $NAME \
--kubernetes-version 1.31.1 --nodepool-name systempool --node-count 2 --enable-node-public-ip \
--enable-managed-identity --enable-cluster-autoscaler --min-count 2 --max-count 3 \
--generate-ssh-keys


### Create secret in Kubernetes for connecting to ACR
