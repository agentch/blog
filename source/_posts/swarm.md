---
title: Create docker swarm for ubuntu
date: 2019-03-15 11:34:56
tags: docker
---

Create docker swarm cluster.

<!-- more -->

## Get Docker CE for Ubuntu

```bash
$ sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update
$ sudo apt-get install -y docker-ce

$ sudo usermod -a -G docker ${user}
```

## Create a swarm

```bash
$ docker swarm init --advertise-addr ${MANAGER-IP}

$ docker swarm join --token ${token} ${MANAGER-IP}:${port}

$ docker info

Containers: 2
Running: 0
Paused: 0
Stopped: 2
  ...snip...
Swarm: active
  NodeID: dxn1zf6l61qsb1josjja83ngz
  Is Manager: true
  Managers: 1
  Nodes: 1
  ...snip...
  
$ docker node ls

ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader
  
```

If you don't have the command available, you can run the following command on a manager node to retrieve the join command for a worker:
```bash
$ docker swarm join-token worker
```

## Deploy a service to the swarm

```bash
$ docker service create --replicas 1 --name helloworld alpine ping docker.com
```
- The **docker service** create command creates the service.
- The **--name** flag names the service helloworld.
- The **--replicas** flag specifies the desired state of 1 running instance.
- The arguments **alpine ping docker.com** define the service as an Alpine Linux container that executes the command **ping docker.com**.

## Scale the service in the swarm

```bash
$ docker service scale ${SERVICE-ID}=${NUMBER-OF-TASKS}
```

## Delete the service running on the swarm

```bash
$ docker service rm ${SERVICE-ID}
```
Verify if the swarm manager removed the service
```bash
$ docker service inspect --pretty ${SERVICE-ID}
```