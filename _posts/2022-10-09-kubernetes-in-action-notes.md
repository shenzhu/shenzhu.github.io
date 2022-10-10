---
title: "Kubernetes in Action阅读笔记"
tags:
  - 云计算
  - 读书笔记
---

# 1. Introducing Kubernetes

Kubernetes abstracts away the hardware infrastructure and exposes your whole data-center as a single enormous computational resource

## 1.1. 为什么需要Kubernetes类似的系统

在科技公司尤其是互联网公司中，将monolithic application转向microservice architecture是大势所趋，相较于单一的monolithic application来说，microservice在开发，部署和协调方面有一些问题需要解决，比如每个microservice对environment，dependency的需求不同，甚至会有冲突。这就需要provide a consistent environment to applications，保证开发环境和生产环境一致

## 1.2. Container Technologies

Kubernetes uses Linux container technologies to provide isolation of running applications

### 1.2.1. 理解什么是container

与container类似的一个概念是Virtual Machine(VM)，当deploy的程序比较大的时候，使用虚拟机是完全可以的，但在microservice architecture中，对每个microservice都提供一个VM会显得非常浪费。相较于VM，container更加轻量级，a container is nothing more than a single isolated process running in the host OS，中间没有虚拟出的OS一层，因此containers all perform system calls on the exact same kernel running in the host OS. Each VM runs its own set of system services, while containers don't, because they all run in the same OS

那么container是怎样做到在同一个OS上isolate process呢？答案是Linux提供的两种机制
- Linux Namespace: Makes sure each process sees its own personal view of the system(files, processes, network interfaces, hostnames, and so on)
- Linux Control Group(cgroups): Limits the amout of resources the process can consume(CPU, memory, network, bandwidth, and so on)

namespace的概念是针对不同的resource而言，一个process需要各种namespace来使用resource，比如Mount(mnt), Process ID(pid), Network(net)等等，each namespace kind is used to isolate a certain group of resources

### 1.2.2. Docker

Docker有三个主要的概念
- Images: A Docker-based container image is something you package your application and its environment into. Docker images是分层的
- Registries: —A Docker Registry is a repository that stores your Docker images and facilitates easy sharing of those images between different people and computers
- Containers: A Docker-based container is a regular Linux container created from a Docker-based container image

Different images can contain the exact same layers because every Docker image is built on top of another image and two different images can both use the same parent image as their base. 两个具有部分相同layer的docker container可以share文件，因为container image layers are read-only, when a container is run, a new writable layer is created on top of the layers in the image

因为没有提供虚拟的OS层，所以Docker对于系统环境有要求，并不是完全portable。最后需要强调的是，Docker本身并不提供process isolation，真正的container isolation是在Linux kernel level实现(Linux Namespaces and groups)，Docker只是让这些功能更加易于使用


## 1.3. Introducing Kubernetes

Kubernetes is a software system that allows you to easily deploy and manage containerized applications on top of it. It relies on the features of Linux containers to run heterogeneous applications without having to know any internal details of these applications and without having to manually deploy these applications on each host. Kubernetes can be thought as an operating system for the cluster

Kubernetes由两部分组成，control plane和worker notes
- Control Plane
    - Kubernetes API Server: Users and the other Control Plane components communicate with
    - Scheduler: Schedules your apps(assigns a worker node to each deployable component of your application)
    - Control Manager: Performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on
    - etcd: a reliable distributed data store that persistently stores the cluster configuration
- Nodes
    - Container runtime: Docker/rkt or others, runs your applications
    - Kubelet: Talks to the API server and manages containers on its node
    - Kubernetes Service Proxy(kube-proxy): Load-balances network traffic between application components

To run an application in Kubernetes, you first need to package it up into one or more container images, push those images to an image registry, and then post a description of your app to the Kubernetes API server. Kubernetes continuously makes sure that the deployed state of the application always matches the description you provided