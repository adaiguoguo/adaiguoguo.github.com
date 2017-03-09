---
layout: post
title: "Docker ps hang issue"
description: can't use docker ps and other docker [commad] is ok
category: docker
tags: [centos,docker]
---
{% include JB/setup %}

---
## Results of environment info?
{% highlight html %}
[root@adct-eless-mesos-9 data]# docker version
Client:
 Version:      1.13.1
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   092cba3
 Built:        Wed Feb  8 06:38:28 2017
 OS/Arch:      linux/amd64
Server:
 Version:      1.13.1
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   092cba3
 Built:        Wed Feb  8 06:38:28 2017
 OS/Arch:      linux/amd64
 Experimental: false
[root@adct-eless-mesos-9 data]# uname -r
3.10.0-514.6.1.el7.x86_64
{% endhighlight %}

## What is the bug behavior?

{% highlight html %}
[root@adct-eless-mesos-9 ~]# docker ps
{% endhighlight %}
That action hang and container can't be stopped.

## How to find the bug root cause?

* 1: Find the dockerd status
{% highlight html %}
[root@adct-eless-mesos-9 data]# ps -ef|grep dockerd
root        1784       1  0 Feb21 ?        00:58:37 /usr/bin/dockerd --exec-opt native.cgroupdriver=cgroupfs --insecure-registry docker-registry.tools.elenet.me:5000 -g /data/docker -H tcp://127.0.0.1:4243 -H unix:///var/run/docker.sock --bip=10.99.5.129/28 --mtu=1450
[root@adct-eless-mesos-9 data]# lsof -p 1784
dockerd 1784 root   23u     unix 0xffff880aecb92c00       0t0   13743016 /var/run/docker.sock
dockerd 1784 root   24u     unix 0xffff880bfe629c00       0t0   13775439 /var/run/docker.sock
dockerd 1784 root   25u     unix 0xffff880f6d75f400       0t0   14510073 /var/run/docker.sock
dockerd 1784 root   26u     unix 0xffff880f6d758400       0t0   14526831 /var/run/docker.sock
dockerd 1784 root   27w      REG             253,17 283825669    4457362 /data/docker/containers/e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676/e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676-json.log
dockerd 1784 root   28r      REG             253,17 283825669    4457362 /data/docker/containers/e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676/e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676-json.log
dockerd 1784 root   29u     sock                0,7       0t0      17167 protocol: NETLINK
dockerd 1784 root   30u     unix 0xffff880f6d75ec00       0t0   14531849 /var/run/docker.sock
dockerd 1784 root   31u     unix 0xffff880ae8e18c00       0t0   14531856 /var/run/docker.sock
dockerd 1784 root   32u     unix 0xffff880fef600400       0t0   14534909 /var/run/docker.sock
{% endhighlight %}
Now we get the container e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676 both read and write.

* 2：Docker inspect

{% highlight html %}
[root@adct-eless-mesos-9 data]# docker inspect e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676
{% endhighlight %}
Now docker inspect id hang.

* 3：Find Pid by docker container id

{% highlight html %}
[root@adct-eless-mesos-9 data]# cat /data/docker/containers/e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676/config.v2.json |python -m json.tool|grep Pid
        "Pid": 7294,
[root@adct-eless-mesos-9 data]# ps -ef|grep 7294
root      661378  660472  0 16:04 pts/0    00:00:00 grep --color=auto 7294
{% endhighlight %}
The container process 7294 is disappeared.

* 4：Ensure the container when to exit.
{% highlight html %}
[root@adct-eless-mesos-9 data]# cat /run/docker/libcontainerd/containerd/events.log|grep e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676
{"id":"e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676","type":"start-container","timestamp":"2017-02-21T11:54:43.827455953+08:00"}
{"id":"e8e8a9e36e6ce633077467c7c7954b1b0cfff3df177a936d4d9b47b7b5e3a676","type":"exit","timestamp":"2017-03-08T19:54:27.877604017+08:00","pid":"init","status":137}
{% endhighlight %}
"status":137    128 + 9 = 137 (9 coming from SIGKILL)

## So I guess?
The container received a SIGKILL by Mesos.But for some reason, the exit operate was crashed.Then we docker ps,that operate will get info from container.But the container process is not found.

---
