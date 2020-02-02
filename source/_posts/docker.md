---
title: Docker
date: 2019-03-14 11:31:00
tags: docker
---

Jenkins integration

<!-- more -->

Plugin: Yet Another Docker

daemon.json
```bash
$ sudo vim /etc/docker/daemon.json

{
    "insecure-registries": [Docker registry URL]
}
```

If /etc/default/docker does exist

```bash
$ sudo vim /etc/default/docker

DOCKER_OPTS="-H tcp://0.0.0.0:${Port} -H unix:///var/run/docker.sock"

$ sudo service docker restart
```

If /etc/default/docker is not found, edit /lib/systemd/system/docker.service as following:

```bash
$ sudo vim /lib/systemd/system/docker.service

ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock

$ sudo systemctl daemon-reload && sudo systemctl restart docker && sudo systemctl status docker
```
