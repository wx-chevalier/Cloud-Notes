# Minikube

# 安装

需要注意的是，Minikube 必须安装在本地，而不是云提供商上。因此，以下硬件规格是针对本地机器的：1 核，2GB 内存，20GB 存储。

## GNU/Linux

```sh
# check if your machine supports virtualization. On GNU / Linux, this can be accomplished with:
$ grep -E --color 'vmx|svm' /proc/cpuinfo

# install kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client

# Download and install the Minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

chmod +x ./minikube

sudo mv ./minikube /usr/local/bin/minikube

minikube version
```

## MacOS

```sh
# verify that the processor supports virtualization
$ sysctl -a | grep -E --color 'machdep.cpu.features|VMX'

$ brew install kubectl
$ kubectl version --client

$ brew install minikube
$ minikube version

# Run the following command to configure the alias and autocomplete for kubectl.

source  <( kubectl completion bash )  # configures the autocomplete in your current session (first, make sure you have installed the bash-completion package).
echo  " source <(kubectl completion bash) "  >>  ~ /.bashrc # add autocomplete permanently to your shell.

# ZSH
source <(kubectl completion zsh)
echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)"
```

## Usage

```sh
minikube start


🎉  minikube 1.10.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.10.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🙄  minikube v1.9.2 on Darwin 10.11
✨  Using the virtualbox driver based on existing profile
👍  Starting control plane node m01 in cluster minikube
🔄  Restarting existing virtualbox VM for "minikube" ...
🐳  Preparing Kubernetes v1.19.1 on Docker 19.03.8 ...
🌟  Enabling addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube"

$ kubectl get nodes

NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   8d    v1.19.1

# Finding the Minikube Address
$ minikube ip

# Accessing the Minikube machine via SSH
$ minikube ssh

# Dashboard
$ minikube dashboard

```
