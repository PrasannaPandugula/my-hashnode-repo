---
title: "Docker  Made Easy"
seoTitle: "Docker vs VM: Simple Explanation with Examples"
datePublished: Sun Aug 24 2025 05:20:34 GMT+0000 (Coordinated Universal Time)
cuid: cmep8pahn000302l13h6uelqf
slug: docker-made-easy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760269761534/5a93f2a1-344c-4b09-9c3e-61f0bdc7e9e8.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1756012796575/4dbd2f42-556b-46f9-9f25-f7db921e27b8.png
tags: docker

---

### **What is a Container?**

A container is a lightweight, portable unit that packages an application and its dependencies together, ensuring it runs consistently across the different environments.

### Container vs Virtual Machine (VM)

**VM’s:** Run on a hypervisor, include a full guest OS → heavier, slower startup, more resource usage.

**Container:** Share the host OS Kernel, only package app + dependencies → lightweight, faster startup, less resource usage

👉 **Why containers are lightweight?**

Because they don’t need a full full operating system; they use the host OS kernel and isolate app at the process level.

### What is Docker?

Docker is a platform that simplifies building, shipping, and running containers using standard images.

### What is Docker Daemon?

The docker daemon (dockerd) this is the background service that listens Docker API requests and manages container life cycle (build, run, pull etc..)

**Docker execution flow:**

docker build → build an image from a docker file \[hostname/repositoryname:tag .\]

docker pull → downloads an image from docker hub(or registry)

docker run→ creates and starts a container from an image.

### Docker Architecture:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756012637444/77c925a5-6361-4426-a493-c933d921df33.png align="center")

**Install docker on EC2 Instance**

1\. connect to EC2 instance via Linux/ Mobaxterm

2\. Run - sudo apt update -y

3\. sudo apt install docker.io -y

4\. To check docker running/ start docker daemon → suto systemctl status docker

5\. check docker read to use → docker run hello-world (you will get permission denied)

→ python need to install using root user, now we have to add ubuntu(is VM) user to docker group.

Cmd: **sudo usermod -aG docker ubuntu** or **source ctrl+c**

→ restart VM or logout: sudo reboot & run - docker run hello-world

6\. clone sample python project in to EC2

Cmd: git clone [https://github.com/PrasannaPandugula/python-docker-hello](https://github.com/PrasannaPandugula/python-docker-hello)

7\. do docker build: **docker build -t prasannapandugula1/my-first-docker-image:latest .** \[prasannapandugula1 - my docker hub username\]

8\. execute image: **docker run -it prasannapandugula1/my-first-docker-image:latest** \[-it = interactive terminal\]

9\. push image to registry → login to user docker hub account  
docker login ; enter username & password.

10\. finally run → **docker push prasannapandugula1/my-first-docker-image:latest**

11\. check repository you should see latest image.

12\. docker images → list all images || docker pull {image id}

13. docker stop $(docker ps -q) \[to pass in the list of all running containers\]
    

Note: if you don’t want to use EC2, go for Docker Desktop (it creates VM) on your laptop/ desktop.

### Jenkins and Docker interaction:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756359618646/29631fe3-e663-4e56-addc-762c9ec7f60c.png align="center")