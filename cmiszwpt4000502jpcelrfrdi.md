---
title: "Day 13: Storage in Kubernetes"
seoTitle: "Docker Storage, Kubernetes Volumes, PVC, Storage Class"
datePublished: Fri Dec 05 2025 15:04:18 GMT+0000 (Coordinated Universal Time)
cuid: cmiszwpt4000502jpcelrfrdi
slug: day-13-storage-in-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1764655694974/301de96f-443a-426f-9250-986c47116682.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1764947037525/df7f2e20-bc3c-4b60-9a05-9bafe6ca889c.png
tags: k8s-series

---

## Introduction

Before understanding the storage concept in container orchestration, such as Kubernetes, let’s examine how storage works in containers.

## Docker Storage

\-Docker storage had two types of plugins: 1. Storage Drivers, 2.Volume Drivers.

### Layered Architecture

Dockerfile for Python app.

```plaintext
FROM Ubuntu
RUN apt-get update && apt-get -y install python
RUN pip install flask flask.mysql
COPY . /opt.source-code
ENTRYPOINT FLASK_APP=/opt.source-code/app.py flask run
```

```bash
$ docker build Dockerfile -t prasanna/my-custom-app
```

**docker build** turns each step present in the file into layers. Once build is completed, we can’t modify content because it’s read-only; to make changes need to rebuild.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764667075219/b44a0330-e4bb-49af-a96f-6822cb8a4b61.png align="left")

**Docker run** will create a new **writable layer** called Container Layer based on top of the Image Layers.

```bash
$ Doker run prasanna/my-custom-app
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764824349720/59cb7d03-9d51-4e8f-b45b-e7d94f7ed8ff.png align="left")

**Writable Layer**: It stores information about logs, modified app details by the user, and any temporary files created by containers. life of this layer as long as the container is alive and same image is shared by all containers created using this image.

\-Log in to the created container using the COPY-ON-WRITE mechanism.

### COPY-ON-WRITE

\-When login to the container using certain methods ex: docker exec, it creates a temp file in Container Layer(read and write) which stores temp credentials, commands, and environment variables.

\-Application code present in the Image Layer, we can still edit details before saving file. A copy of details is stored in Container Layer by Docker, so a different version of file is present in the read-write layer. All future modifications to this file are handled through the COPY-ON-WRITE mechanism.

\-If we get rid of the container, the files present in the Container Layer get deleted.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764828099192/a5241c9a-5770-427e-b07c-b5929846f7f0.png align="left")

\-Persist the deleted data using Volumes.

### Volume Mount

\-It mounts volume from volume directory Ex: /var/lib/volumes

\-Suppose we are working with database and would like to preserve data created by a container we could add persistence to the container.

\-Create Docker Volume

```bash
$ docker volume create data_volume   # volumes and data_volume, folders get created under /var/lib/docker directory
```

\-Mount a Volume

```bash
$ docker run --mount type=bind,source=/var/lib/volumes,target=/var/lib/mysql mysql
```

\-This will create a new container(mysql-container) and mount data\_volume into /var/lib/mysql folder, all the data stored in the database is in fact stored in data\_volume on host, even if the container is destroyed, data is still active.

What if we did not create volume before mounting? Docker will create a volume automatically and mount the container.

```bash
$ docker run -v data_volume2:/var/lib/mysql mysql   #it creates docker_volume2
```

### Bind Mount

Mounts a directory from any location on the Docker host.

```bash
$ docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764837548768/834dc27d-edf8-449f-bd22-e09bc2bde476.png align="center")

**Storage Drivers**: The Entire process is taken care by storage layers.

Common storage drivers are AUFS, ZFS, BTRFS, Device Mapper, Overlay, and Overlay2. The selection of storage drivers depends on the OS, Ex, Ubuntu uses AUFS.

\-Docker automatically selects storage drivers based on the OS.

**Volume Drivers**: Volume Drivers are maintained by the volume plugins.

Plugins: Local, AzureFile storage, Convoy, DigitalOcean Block Storage, Flocker, gce-docker, GlusterFS, NetApp, RexRay, Portwork, VMware vSphere storage.

\-When we run a docker container, we can choose a specific volume driver.

```bash
$ docker run -it \
    --name mysql
    --volume-driver rexray/ebs
    --mount src=ebs-vol,target=/var/lib/mysql mysql
```

This will create a container and attach a volume from AWS EBC cloud, when a container exists, data safe in the cloud.

**Container Runtime Interface**: It defines how orchestration, such as Kubernetes, should communicate with containers like Docker. If any new containers are introduced, they can follow CRI standards to work with K8s.

**Container Networking Interface**: It offers different networking solutions.

**Container Storage Interface**: It supports multiple storage solutions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1764843553998/8c2ebb4d-3abc-4cc2-a6be-f65e2d8f4a8e.png align="center")

## Kubernetes Volumes and Volume Mount

**Volume:** Its actual storage location outside the container file system; it can live in different places depending on the type.

* **emptyDir**: stored on the Node(in RAM or Disk)
    
* **hostPath**: stored in a specific directory of a Node.
    
* **PVC / PersistentVolume**: stored in an external storage.
    
    * AWS EBC, NFC, GCE Persistence Disk, Ceph, SAN, Cloud Storage Backend.
        

**Volume Mount**: a directory inside the container filesystem, like a path app/data.

Let’s see an example with a Pod object.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:                 #node level 
      sizeLimit: 500Mi
      medium: Memory
# or
  volumes:
  - name: data-volume
    awsElasticBlockStorage:    #external storage 
       volumeId: <volume-id>
       fsType: ext4
```

## Persistent Volume and Volume Claims in K8s

**Persistent Volume**: It is a storage in k8s cluster that exists independently of Pods.

**Persistent Volume Claims**: a request for storage by a Pod that uses PVC to access PV.

1. The administrator creates a set of Persistent Volumes, and the user creates Pods to utilize the storage.
    
2. Kubernetes automatically binds PVC and PV based on the request, and it checks sufficient capacity, Access Modes, Volume Modes, and Storage Class.
    
3. If multiple matches for a single claim, and we would like to use a particular volume, use **Labels and Selectors** to bind to the correct volume.
    
4. A smaller claim can be bound to use a larger volume if all other criteria match, there is a one-to-one relation between claims and volumes, and no other claim can use the remaining volume.
    
5. If no other volumes are available, PVC is pending until new volumes are made available to the cluster. Once volumes are available, the claim would automatically be bound to the newly available volume.
    

### Persistent Volume

```yaml
apiVersion: V1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessmodes:                                  #Types: ReadWriteOnce, ReadOnlyMany, ReadWriteMany 
  - ReadWriteOnce
  capacity: 
    storage: 1Gi
awsElasticBlockStorage:
   volumeId: <volume-id>
   fsType: ext4
```

```bash
$ kubectl apply -f pv-definition.yaml
```

### Persistent Volume Claim

```yaml
apiVersion: V1
kind: PersistentVolumeClaim
metadata:
  name: my-vol-claim
spec:
  accessMode: 
  - ReadWriteOnce    #access should match with PV
  resources:
    request: 
      storage: 500mi
```

```bash
$ kubectl apply -f pvc-definition.yaml
$ kubectl get persistentvolumeclaim        #you should see binding details of PV and PVC
```

### Delete PVC’s

* When we delete PVC
    
* ```bash
      $ kubectl delete persistentvolumeclaim my-vol-claim
    ```
    
* PV associated with that deleted PVC set to Retain default until manually deleted by administrator.
    
* ```yaml
      persistentVolumeReclaimPolicy: Retain      #or, it can be deleted automatically.
    ```
    
    **StatefulSet:** we use it when pods need a stable identity and stable storage; using PVC/PV alone cannot guarantee it.
    

## Storage Class

So far, we have created PV, PVC, and used PVC in the Pod definition file as persistent volumes entire process called **static provisioning**.

**Dynamic Provisioning**

With Storage Class, we can define a provisioner, such as gcp, AWSEBS, or AzureDisk, which automatically creates PV and attaches it to a Pod when a claim is made.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage               # use this name in PVC define 'storageClassName=google-storage'
 provisioner: kubernetes.io/gce-pd 
 parameters:
   type: [pd-standerd | pd-ssd] 
   replication-type: [none  | regional-pd]
```