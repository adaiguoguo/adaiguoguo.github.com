---
layout: post
title: "Docker kill issue"
description: kill one docker task,but can't killed and the docker task can't be stop
category: kernel
tags: [centos,docker,kernel]
---
{% include JB/setup %}

---
## Results of environment info?
{% highlight html %}
[root@adca-mesos-20 ns]# docker version
Client:
 Version:      1.12.0-dev
 API version:  1.25
 Go version:   go1.6.2
 Git commit:   d27a038
 Built:
 OS/Arch:      linux/amd64
 Experimental: true
Server:
 Version:      1.12.0-dev
 API version:  1.25
 Go version:   go1.6.2
 Git commit:   d27a038
 Built:
 OS/Arch:      linux/amd64
 Experimental: true
[root@adca-mesos-20 ns]# uname -r
3.10.0-229.el7.x86_64
{% endhighlight %}

## What is the bug behavior?

{% highlight html %}
[root@adca-mesos-20 ~]# docker stop 2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601
{% endhighlight %}
That action hang and container can't be stopped.

## How to find the bug root cause?

* 1: Find the hang PID
{% highlight html %}
[root@adca-mesos-20 ~]# docker inspect 2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601 |grep Pid
            "Pid": 1489110,
            "PidMode": "",
            "PidsLimit": 0,
[root@adca-mesos-20 ~]# ps -ef|grep 1489110
root     1489110 1489091  0 Feb09 ?        00:00:00 [start.sh]
root     1489145 1489110  0 Feb09 ?        00:00:25 [java] <defunct>
{% endhighlight %}
Now we get the process parent PID 1489091

* 2：Have a look at parent PID

{% highlight html %}
[root@adca-mesos-20 ns]# ps -ef|grep 1489091
root     1489091    2471  0 Feb09 ?        00:00:00 docker-containerd-shim 2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601 /var/run/docker/libcontainerd/2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601 docker-runc
root     1489110 1489091  0 Feb09 ?        00:00:00 [start.sh]
{% endhighlight %}

{% highlight html %}
[root@adca-mesos-20 ns]# strace -f -p 1489091
Process 1489091 attached with 10 threads
[pid 1489142] futex(0xc8200c6508, FUTEX_WAIT, 0, NULL <unfinished ...>
[pid 1489141] futex(0xc82007c908, FUTEX_WAIT, 0, NULL <unfinished ...>
[pid 1489102] futex(0xc820126108, FUTEX_WAIT, 0, NULL <unfinished ...>
[pid 1489101] read(11,  <unfinished ...>
[pid 1489098] read(6,  <unfinished ...>
[pid 1489097] read(9,  <unfinished ...>
[pid 1489096] futex(0xc82004ad08, FUTEX_WAIT, 0, NULL <unfinished ...>
[pid 1489095] futex(0x8657c0, FUTEX_WAIT, 0, NULL <unfinished ...>
[pid 1489094] restart_syscall(<... resuming interrupted call ...> <unfinished ...>
[pid 1489091] futex(0x84b788, FUTEX_WAIT, 0, NULL
{% endhighlight %}

{% highlight html %}
[root@adca-mesos-20 ns]# lsof -p 1489091
COMMAND       PID USER   FD      TYPE DEVICE SIZE/OFF     NODE NAME
docker-co 1489091 root  cwd       DIR   0,18      160 56026342 /run/docker/libcontainerd/containerd/2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601/init
docker-co 1489091 root  rtd       DIR  253,2     4096        2 /
docker-co 1489091 root  txt       REG  253,2  2428640   666380 /usr/bin/docker-containerd-shim
docker-co 1489091 root  mem       REG  253,2  2112384   656862 /usr/lib64/libc-2.17.so
docker-co 1489091 root  mem       REG  253,2   142304   656888 /usr/lib64/libpthread-2.17.so
docker-co 1489091 root  mem       REG  253,2   164440   656855 /usr/lib64/ld-2.17.so
docker-co 1489091 root    0r      CHR    1,3      0t0     1028 /dev/null
docker-co 1489091 root    1w      CHR    1,3      0t0     1028 /dev/null
docker-co 1489091 root    2w      CHR    1,3      0t0     1028 /dev/null
docker-co 1489091 root    3u  a_inode    0,9        0     5855 [eventpoll]
docker-co 1489091 root    4w      REG   0,18        0 56034720 /run/docker/libcontainerd/containerd/2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601/init/shim-log.json
docker-co 1489091 root    5w     FIFO   0,18      0t0 56026344 /run/docker/libcontainerd/containerd/2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601/init/exit
docker-co 1489091 root    6u     FIFO   0,18      0t0 56026345 /run/docker/libcontainerd/containerd/2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601/init/control
docker-co 1489091 root    7r     FIFO    0,8      0t0 56024011 pipe
docker-co 1489091 root    9r     FIFO    0,8      0t0 56024012 pipe
docker-co 1489091 root   11r     FIFO    0,8      0t0 56024013 pipe
docker-co 1489091 root   13u     FIFO   0,18      0t0 56044767 /run/docker/libcontainerd/2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601/init-stdout
docker-co 1489091 root   14u     FIFO   0,18      0t0 56044768 /run/docker/libcontainerd/2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601/init-stderr
docker-co 1489091 root   15u  a_inode    0,9        0     5855 [eventfd]
docker-co 1489091 root   16r     FIFO   0,18      0t0 56044766 /run/docker/libcontainerd/2d73d951f0f765b2e32997ca9f801bc6a0278df48f75227782a00d378a021601/init-stdin
docker-co 1489091 root   17u  a_inode    0,9        0     5855 [eventfd]
docker-co 1489091 root   19u  a_inode    0,9        0     5855 [eventfd]
docker-co 1489091 root   49u  a_inode    0,9        0     5855 [eventfd]
{% endhighlight %}
We will find pipe can't be correct close in fd: 9,11.

* 3：Find the PID namespace problem.


{% highlight html %}
[root@adca-mesos-20 ns]# ls -al /proc/1489110/ns/
ls: cannot read symbolic link /proc/1489110/ns/net: No such file or directory
ls: cannot read symbolic link /proc/1489110/ns/uts: No such file or directory
ls: cannot read symbolic link /proc/1489110/ns/ipc: No such file or directory
ls: cannot read symbolic link /proc/1489110/ns/mnt: No such file or directory
total 0
dr-x--x--x 2 root root 0 Feb  9 00:54 .
dr-xr-xr-x 8 root root 0 Feb  9 00:54 ..
lrwxrwxrwx 1 root root 0 Feb  9 17:12 ipc
lrwxrwxrwx 1 root root 0 Feb  9 17:12 mnt
lrwxrwxrwx 1 root root 0 Feb  9 00:54 net
lrwxrwxrwx 1 root root 0 Feb  9 17:12 pid -> pid:[4026532278]
lrwxrwxrwx 1 root root 0 Feb  9 17:12 uts
{% endhighlight %}

{% highlight html %}
[root@adca-mesos-20 ns]# cat /proc/1489110/stack
[<ffffffff81074033>] do_wait+0x203/0x260
[<ffffffff81075150>] SyS_wait4+0x80/0x110
[<ffffffff810f1f88>] zap_pid_ns_processes+0x118/0x210
[<ffffffff81074dc9>] do_exit+0xa19/0xa60
[<ffffffff81074e8f>] do_group_exit+0x3f/0xa0
[<ffffffff810851e0>] get_signal_to_deliver+0x1d0/0x6e0
[<ffffffff81013467>] do_signal+0x57/0x6c0
[<ffffffff81013b39>] do_notify_resume+0x69/0xb0
[<ffffffff81614cdd>] int_signal+0x12/0x17
[<ffffffffffffffff>] 0xffffffffffffffff
{% endhighlight %}
zap_pid_ns_processes+0x118/0x210 is the root cause.

## How to find the code in kernel?
* 1：Install kernel-debuginfo
{% highlight html %}
[root@adca-mesos-20 ns]# wget http://debuginfo.centos.org/7/x86_64/kernel-debuginfo-3.10.0-229.el7.x86_64.rpm
[root@adca-mesos-20 ns]# wget http://debuginfo.centos.org/7/x86_64/kernel-debuginfo-common-x86_64-3.10.0-229.el7.x86_64.rpm
{% endhighlight %}

{% highlight html %}
[root@adca-mesos-20 ns]# rpm -hiv kernel-debuginfo-common-x86_64-3.10.0-229.el7.x86_64.rpm
[root@adca-mesos-20 ns]# rpm -hiv kernel-debuginfo-3.10.0-229.el7.x86_64.rpm
{% endhighlight %}

* 2：Find debug vmlinux
{% highlight html %}
[root@adca-mesos-20 ns]# rpm -q -l kernel-debuginfo
...
/usr/lib/debug/lib/modules/3.10.0-229.el7.x86_64/vmlinux
...
{% endhighlight %}

* 3：Find the code from stack
{% highlight html %}
root@adca-mesos-20 ns]# addr2line -e /usr/lib/debug/lib/modules/3.10.0-229.el7.x86_64/vmlinux ffffffff81074dc9
/usr/src/debug/kernel-3.10.0-229.el7/linux-3.10.0-229.el7.x86_64/kernel/exit.c:539
...
 530         if (unlikely(pid_ns->child_reaper == father)) {
 531                 write_unlock_irq(&tasklist_lock);
 532                 if (unlikely(pid_ns == &init_pid_ns)) {
 533                         panic("Attempted to kill init! exitcode=0x%08x\n",
 534                                 father->signal->group_exit_code ?:
 535                                         father->exit_code);
 536                 }
 537
 538                 zap_pid_ns_processes(pid_ns);
 539                 write_lock_irq(&tasklist_lock);
 540         }
...

{% endhighlight %}

{% highlight html %}
[root@adca-mesos-20 ns]# addr2line -e /usr/lib/debug/lib/modules/3.10.0-229.el7.x86_64/vmlinux ffffffff810f1f88
/usr/src/debug/kernel-3.10.0-229.el7/linux-3.10.0-229.el7.x86_64/kernel/pid_namespace.c:224 (discriminator 1)
...
 221         do {
 222                 clear_thread_flag(TIF_SIGPENDING);
 223                 rc = sys_wait4(-1, NULL, __WALL, NULL);
 224         } while (rc != -ECHILD);
...
{% endhighlight %}
sys_wait4 is the result why hang.





---
