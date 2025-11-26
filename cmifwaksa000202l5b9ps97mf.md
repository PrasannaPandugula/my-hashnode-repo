---
title: "Day 10: TLS Certificates Workflow and API"
seoTitle: "Kubernetes API groups, cluster access, Kubectl Proxy"
datePublished: Wed Nov 26 2025 11:02:06 GMT+0000 (Coordinated Universal Time)
cuid: cmifwaksa000202l5b9ps97mf
slug: day-10-tls-certificates-workflow-and-api
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1764136279470/6bda2949-852c-49a5-b680-8e74de834837.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1764154884682/65071ea4-4396-4313-a779-3eacc5739aab.png
tags: k8s-series

---

## Grant Cluster Access

```bash
$ controlplane ~ ➜  ls
akshay.csr  akshay.key     # new user Akshay requesting access to cluster
$ cat akshay.csr | base64 -w 0   # encode the key to use in object yaml
```

Create Object

```yaml
---
apiVersion: certificates.k8s.io/v1  
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: <Paste the base64 encoded value of the CSR file>
  signerName: kubernetes.io/kube-apiserver-client  #signer name
  usages:
  - client auth

$  kubectl apply -f akshay-csr.yaml #run apply command.
```

Check status of approval

```bash
$  kubectl apply -f akshay-csr.yaml  # Pending, Approved, Issued

$ kubectl certificate approve akshay # approve cert
certificatesigningrequest.certificates.k8s.io/akshay approved
$ kubectl certificate deny agent-smith  # deny request if its suspicious
$ kubectl delete csr agent-smith # delete CSR
```

## KubeConfig Security File

Kubeconfig file consists of 3 sections:

1. Cluster - available k8s cluster.
    
2. Users- who need access to the cluster.
    
3. Context- pair up the user and cluster.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764148461627/d6d55d34-3580-4dfd-bd91-da74909dc0bd.png align="center")

```bash
$ kubectl config view # view the config fle
$ kubectl config view -kubeconfig=my-custom-config  # list specific file
$ kubectl config use-context prod-user@production #change context in config file, changes update file directly
$ kubectl config -h 
```

We don't want to specify the **kubeconfig file** option on each `kubectl` command.

1. open shell configuration file
    
2. ```bash
    vi ~/.bashrc
    ```
    
3. add one of these lines to export the variables
    
    ```bash
    export KUBECONFIG=/root/my-kube-config
    # OR
    export KUBECONFIG=~/my-kube-config
    # OR
    export KUBECONFIG=$HOME/my-kube-config
    ```
    
4. apply changes
    

```bash
source ~/.bashrc
```

Kubectl command to set certificate path

```bash
 kubectl config set-credentials dev-user \ # user = dev-user
  --client-certificate=/etc/kubernetes/pki/users/dev-user/dev-user.crt \
  --client-key=/etc/kubernetes/pki/users/dev-user/dev-user.key \
  --kubeconfig=/root/my-kube-config # file location 
```

## API Groups in Kubernetes

We can check version and pod details using k8s api

```bash
$ curl https://kube-master:6443/version
$ curl https://kube-master:6443/version/api/v1/pods #/api
$ curl https://localhost:6443 -k #list all api groups
```

We have multiple api groups

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764152844284/8046ad63-9084-484e-92f8-adb4817417e8.png align="center")

**/api -** it contains the core functionality of the cluster.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764153574044/a631a80a-b0d1-4f7d-9a65-740d5dd1b63a.png align="center")

/apis- are more organized

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764153921260/17c56696-3baf-4f56-9390-a949678ba8c0.png align="center")

**Kubectl Proxy:** to access cluster use kubectl proxy this way we no need to pass cert with command.

```bash
$ Kubectl Proxy  #it launches the local https server, uses credentials and certs from kube config file
Staring to server on 127.0.0.1:8001 
```