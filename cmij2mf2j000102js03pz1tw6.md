---
title: "Day 12: K8s Service Account and  Image Security"
seoTitle: "Image Security, Private Registry, Security Context, Network Policies"
datePublished: Fri Nov 28 2025 16:22:34 GMT+0000 (Coordinated Universal Time)
cuid: cmij2mf2j000102js03pz1tw6
slug: day-12-k8s-service-account-and-image-security
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1764346876530/0846a274-76c4-4e68-8ef0-b57fba7d38aa.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1764346935365/350d07cc-0d7c-4ac5-a0e6-9947ce53d171.png
tags: k8s-series

---

## Service Accounts in Kubernetes

\-Service Accounts are used by other applications or services to interact with Kubernetes, Ex: Jenkins, Prometheus

\-Tokens are created for service accounts to prove identity.

\-Create a service account, run the command below.

```bash
$ kubectl create serviceaccount srv678905
```

\-To use the service account from an **external application** (CI/CD, Monitoring Tools)

* create a token
    
* ```bash
            $ kubectl create token srv678905 --duration 2h   #default expiry 1h
    ```
    

**Service account for Internal applications**

* Every namespace has a default service account.
    
* The default service account is automatically attached to the pod on creation.
    
* To attach a new service account to pod, use serviceAccountName in pod object.
    
* ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: nginx
            spec:
              containers:
              - name: nginx
                image: nginx:1.14.2
              serviceAccountName: srv678905  # binding new sc
    ```
    
    \- When service account is attached to a Pod, Kubernetes:
    
* * Automatically creates a token and mounts as a projected token
        
        \* Automatically rotates the token.
        
        \* Automatically expires token when Pod is deleted.
        

## Image Security

### Basics of Image Names

Let’s see Docker image naming convention.

```yaml
image: nginx     #we use, but internally it treated as 

image: docker.io/liberary/nginx  [docker.io # registry where images are pulled from]
                                 [liberary # is user account if we don't have one, it uses default docker acc]
                                 [nginx # image or Repository]
Ex: gcp.io/kubernetes-e2e-test-images/dnsutils #public gcp image
```

### Private Repository

Run docker container using private repository using imperative commands.

```bash
$ docker login private-repository.io # login with credentials
$ docker run private-registory.io/apps/may-app
```

Use private registry image in Pod definition file.

```yaml
apiVersion: V1
knid: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: private-registory.io/apps/may-app
# how to implement authentication login part ?
```

In Kubernetes, images pull and run by the docker runtime on worker node, we have to pass credentials to docker runtime on worker nodes.

* Create secret object with credentials.
    
* ```bash
          $ kubectl create secret docker-registry regcred \   # docker-registry is build in secret to store credentials
              --docker-server= private-registry.io  \
              --docker-username= prasanna \
              --docker-password= prasanna@898  \
              --docker-email= prasanna@gmail.com
          
          # Specify the secret name in Pod file under containers
    ```
    

```yaml
spec:
  containers:
  - name: nginx
    image: private-registory.io/apps/may-app
  imagePullSecrets:
  - name: regcred   # name should match
```

## Docker Security

As we know docker uses shared libraries of kernel or host OS, root user can perform any action on system, so docker security limits access to user.

\-Add user while running container.

```bash
 $ docker run --user=a67389 ubuntu sleep 3000    #defining user 
 $ docker run -cap-add MAC_ADMIN ubuntu          # -cap-add, defining linux capability to limit access
```

## Kubernetes Security Context

In kubernetes, containers are encapsulated in Pods, we can set security setting at container level or pod level.

\-Pod level security carries to all the containers within the pod.

\-If we configure security setting both Pod and Container level, Container setting will override.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:                  #Pod level security 
     runAsUser: a67389
  containers:
   - name: ubuntu
     image: ubuntu
     command: ["sleep" "3600"]
     securityContext:                  #container level security 
        runAsUser: a67389
        capabilities:                  #capabilities= access restrict to user
           add: ["MAC_ADMIN"]
# Capabilities are only supported container level and not the pod level.
```

Example:

Check what user used to execute sleep process within the pod ?

```bash
$ kubectl exec <pod-name> -- whoami
root
```

## Network Policies

Network policies, we can restrict/ deny traffic between pods within namespace or outside of the cluster.

Two types of network policies:

**1.Ingress:** traffic **entering** to system, network, or cluster (request coming into application from internet).

**2.Egress:** traffic **leaving** from system, network, or cluster (making **outgoing requests** to external APIs).

Suppose, i don’t want **Web Pod** directly send traffic to **DB Pod**, will create network policy for DB and associate it with DB Pod by using Labels and Selectors.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764330004135/c28c2234-5268-44e1-b1ce-a5fb8a35972d.png align="left")

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:                   # ip block suppose any backup server would recieve traffice from db pod
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - podSelector:
        matchLabels:
          role: api-pod        # In cluster any pod with label 'api-pod' can send traffic, if not specify namespace
      namespaceSelector:
        matchLabels:
          project: myproject
          kubernetes.io/metadata.name: prod     # selected prod env
    ports:
    - protocol: TCP
      port: 33o6
  egress:                                       # send out traffic to ecternal ip
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

List out network policies

```bash
$ kubectl get networkpolicy or kubectl get netpol
NAME             POD-SELECTOR   AGE
payroll-policy   name=payroll   2m42s
```

## **Kubectx Tool**

This tool, used to switch **context** between clusters in a multi cluster environment.

**What is Context?**

Context is set of access parameters that tells **kubectl,** ‘which cluster to talk, which user credentials to use and which namespace to operate in’.

Example

```yaml
contexts:                   #dev context
- name: dev
  context:
    cluster: dev-cluster
    user: dev-user
    namespace: dev
- name: prod              #prod context
  context:
    cluster: prod-cluster
    user: prod-user
    namespace: default
```

```bash
kubectx dev    #switch to dev context
kubectx prod   #switch to prod context
kubectx -      # switch back to previous context
kubectx -c     # current context
```

## Kubens Tool

Kubens used to switch between namespace quickly.

```bash
kubens <namespace-name>
kubens-  # swich back to previous ns
```