---
title: Centos7安装nfs
date: 2020-04-08 10:07:25
tags:
- kubernetes
- nfs
categories:
- 笔记
---

Kubernetes 节点安装 nfs。

<!-- more -->

操作系统：CentOS Linux release 7.7.1908 (Core)
IP:192.168.7.8
   192.168.5.36
	 192.168.6.213

```bash
#192.168.7.8/192.168.5.36/192.168.6.213
#查看 rpcbind 状态
$ systemctl status rpcbind

● rpcbind.service - RPC bind service
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; vendor preset: enabled)
   Active: inactive (dead)

Apr 03 08:56:46 master systemd[1]: Dependency failed for RPC bind service.
Apr 03 08:56:46 master systemd[1]: Job rpcbind.service/start failed with result 'dependency'.

$ journalctl -xe  #查看日志

-- A session with the ID 101 has been terminated.
Apr 08 09:51:47 master polkitd[813]: Registered Authentication Agent for unix-process:18197:43531022 (system bus name :1.261493 [/usr/bin/pkttyagent --notify-fd 5 --fallback], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale en_US.UTF-8)
Apr 08 09:51:47 master systemd[1]: rpcbind.socket failed to listen on sockets: Address family not supported by protocol
Apr 08 09:51:47 master systemd[1]: Failed to listen on RPCbind Server Activation Socket.
-- Subject: Unit rpcbind.socket has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit rpcbind.socket has failed.

$ vim  /etc/systemd/system/sockets.target.wants/rpcbind.socket #rpcbind.service依赖rpcbind.socket

[Unit]
Description=RPCbind Server Activation Socket

[Socket]
ListenStream=/var/run/rpcbind.sock

# RPC netconfig can't handle ipv6/ipv4 dual sockets
BindIPv6Only=ipv6-only
ListenStream=0.0.0.0:111
ListenDatagram=0.0.0.0:111
#ListenStream=[::]:111    #注释监听的ipv6端口
#ListenDatagram=[::]:111

[Install]
WantedBy=sockets.target

$ systemctl restart rpcbind.socket #重启rpcbind.socket

Warning: rpcbind.socket changed on disk. Run 'systemctl daemon-reload' to reload units.
Job for rpcbind.socket failed. See "systemctl status rpcbind.socket" and "journalctl -xe" for details.

$ systemctl daemon-reload #systemd发现配置文件有改动，需要重载一下
$ systemctl restart rpcbind.socket
$ systemctl status rpcbind.socket

● rpcbind.socket - RPCbind Server Activation Socket
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.socket; enabled; vendor preset: enabled)
   Active: active (listening) since Wed 2020-04-08 10:06:06 CST; 14s ago
   Listen: /var/run/rpcbind.sock (Stream)
           0.0.0.0:111 (Stream)
           0.0.0.0:111 (Datagram)

Apr 08 10:06:06 master systemd[1]: Listening on RPCbind Server Activation Socket.

$ systemctl start rpcbind
$ systemctl status rpcbind

● rpcbind.service - RPC bind service
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-04-08 10:22:27 CST; 5s ago
  Process: 12694 ExecStart=/sbin/rpcbind -w $RPCBIND_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 12695 (rpcbind)
    Tasks: 1
   Memory: 1.2M
   CGroup: /system.slice/rpcbind.service
           └─12695 /sbin/rpcbind -w

Apr 08 10:22:27 master systemd[1]: Starting RPC bind service...
Apr 08 10:22:27 master systemd[1]: Started RPC bind service.

#192.168.7.8机器作为nfs server
$ vim /etc/exports

/data/nfs 192.168.7.0/22(rw,sync,no_root_squash) #写入挂载配置

$ systemctl start nfs
$ mkdir /data/nfs -p #创建挂载路径
$ chmod 777 /data/nfs
$ rpcinfo -p localhost

   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  46633  status
    100024    1   tcp  58541  status
    100005    1   udp  20048  mountd
    100005    1   tcp  20048  mountd
    100005    2   udp  20048  mountd
    100005    2   tcp  20048  mountd
    100005    3   udp  20048  mountd
    100005    3   tcp  20048  mountd
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    3   udp   2049  nfs_acl
    100021    1   udp  45040  nlockmgr
    100021    3   udp  45040  nlockmgr
    100021    4   udp  45040  nlockmgr
    100021    1   tcp  34461  nlockmgr
    100021    3   tcp  34461  nlockmgr
    100021    4   tcp  34461  nlockmgr

$ showmount -e localhost

Export list for localhost:
/data/nfs 192.168.7.0/22

#192.168.5.36/192.168.6.213
$ showmount -e 192.168.7.8

Export list for 192.168.7.8:
/data/nfs 192.168.7.0/22

$ mkdir -p /data/nfs #挂载目录
$ mount -t nfs 192.168.7.8:/data/nfs /data/nfs/

# 192.168.7.8
$ cd /data/nfs/
$ touch 1.x

# 192.168.5.36
$ cd /data/nfs
$ echo 5.36 >> 1.x

#192.168.6.213
$ cd /data/nfs
$ echo 6.213 >> 1.x

#192.168.7.8
$ echo 7.8 >> 1.x
$ cat 1.x

5.36
6.213
7.8

```
