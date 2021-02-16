# 用户管理

# 创建用户

要在 Kubernetes 上创建一个用户，我们需要为该用户生成一个 CSR（证书签名请求）。我们要使用的用户是 linuxtips 作为例子。

```sh
$ openssl req -new -newkey rsa:4096 -nodes -keyout linuxtips.key -out linuxtips.csr -subj "/CN=linuxtips"

$ cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: linuxtips-csr
spec:
  groups:
  - system:authenticated
  request: $(cat linuxtips.csr | base64 | tr -d '\n')
  usages:
  - client auth
EOF
```

要查看创建的 CSR，请使用以下命令。

```sh
$ kubectl get csr

# The CSR must have the status Pending, we will approve it
$ kubectl certificate approve linuxtips-csr

```

现在证书已经被集群的证书颁发机构(CA)签署，我们将使用下面的命令来获取签署的证书。

```sh
$ kubectl get csr linuxtips-csr -o jsonpath='{.status.certificate}' | base64 --decode > linuxtips.crt

```

这将是必要的配置 kubeconfig 的文件指的是集群的 CA，为了获得它，我们将提取它从 kubeconf 当前我们正在使用的。

```sh
$ kubectl config view -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' --raw | base64 --decode - > ca.crt

```

一旦完成，我们将为新用户设置我们的 kubeconfig。

```sh
$ kubectl config set-cluster $(kubectl config view -o jsonpath='{.clusters[0].name}') --server=$(kubectl config view -o jsonpath='{.clusters[0].cluster.server}') --certificate-authority=ca.crt --kubeconfig=linuxtips-config --embed-certs

# Now setting the confs of user key:
$ kubectl config set-credentials linuxtips --client-certificate=linuxtips.crt --client-key=linuxtips.key --embed-certs --kubeconfig=linuxtips-config

# Now let's define context linuxtipsand then we will use it:
$ kubectl config set-context linuxtips --cluster=$(kubectl config view -o jsonpath='{.clusters[0].name}')  --user=linuxtips --kubeconfig=linuxtips-config
$ kubectl config use-context linuxtips --kubeconfig=linuxtips-config

# test
$ kubectl version --kubeconfig=linuxtips-config
```
