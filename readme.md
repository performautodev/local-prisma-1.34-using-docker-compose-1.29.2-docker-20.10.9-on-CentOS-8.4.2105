#host.docker.internal not working on prismagraphql/prisma:1.34 on docker-compose (1.29.2) docker 20.10.9 CentOS 8.4.2105

issue refs :

https://stackoverflow.com/questions/69494586/host-docker-internal-not-working-on-prismagraphql-prisma1-34-on-docker-compose

https://github.com/docker/compose/issues/8771

https://github.com/moby/moby/issues/42919

https://github.com/prisma/prisma/issues/9677


```shell
docker-compose up -d
```
### Visit http://localhost:4477/


```shell
[dev@localhost ~]$ uname -a
Linux localhost.localdomain 4.18.0-305.19.1.el8_4.x86_64 #1 SMP Wed Sep 15 15:39:39 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux


[root@localhost dev]# cat /etc/centos-release
CentOS Linux release 8.4.2105
```


```shell
[root@localhost dev]# docker version
Client: Docker Engine - Community
 Version:           20.10.9
 API version:       1.41
 Go version:        go1.16.8
 Git commit:        c2ea9bc
 Built:             Mon Oct  4 16:08:25 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.9
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.8
  Git commit:       79ea9d3
  Built:            Mon Oct  4 16:06:48 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.11
  GitCommit:        5b46e404f6b9f661a205e28d59c982d3634148f8
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```


```shell
[root@localhost dev]# docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.6.3-docker)
  scan: Docker Scan (Docker Inc., v0.8.0)

Server:
 Containers: 2
  Running: 1
  Paused: 0
  Stopped: 1
 Images: 2
 Server Version: 20.10.9
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 5b46e404f6b9f661a205e28d59c982d3634148f8
 runc version: v1.0.2-0-g52b36a2
 init version: de40ad0
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 4.18.0-305.19.1.el8_4.x86_64
 Operating System: CentOS Linux 8
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 4.589GiB
 Name: localhost.localdomain
 ID: YOKS:77GU:YL3A:7ZDV:BGGO:6VAC:BPV3:O4AR:UZW7:GAZR:JGIX:B6TF
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```


```shell
[dev@localhost ~]$ docker-compose -v
docker-compose version 1.29.2, build 5becea4c
```


1. This not working :

```yml
version: '3'
services:
  prisma:
    image: prismagraphql/prisma:1.34
    restart: always
    ports:
    - "4477:4477"
    environment:
      PRISMA_CONFIG: |
        port: 4477
        # uncomment the next line and provide the env var PRISMA_MANAGEMENT_API_SECRET=my-secret to activate cluster security
        # managementApiSecret: my-secret
        databases:
          default:
            connector: postgres
            host: host.docker.internal
            database: postgres
            schema: public
            user: postgres
            password: yrt
            rawAccess: true
            port: '5432'
            migrations: true
```

==> Result :

```
This site can’t be reachedlocalhost refused to connect.
Try:

Checking the connection
Checking the proxy and the firewall
ERR_CONNECTION_REFUSED
```

===> i ping host.docker.internal

```shell
[dev@localhost ~]$ ping host.docker.internal
ping: host.docker.internal: Name or service not known
```


```shell
[dev@localhost ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

so i changed the `/etc/hosts`


```shell
[root@localhost dev]# echo "127.0.0.1   host.docker.internal" >> /etc/hosts
[root@localhost dev]#
[root@localhost dev]# cat /etc/hosts 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
127.0.0.1   host.docker.internal
```


```shell
[root@localhost dev]# ping host.docker.internal
PING host.docker.internal (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.057 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.058 ms
^C
--- host.docker.internal ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1042ms
rtt min/avg/max/mdev = 0.057/0.057/0.058/0.007 ms
[root@localhost dev]# ^C

```

==> result  NOT WORKING
```
This site can’t be reachedThe connection was reset.
Try:

Checking the connection
Checking the proxy and the firewall
ERR_CONNECTION_RESET
```


I try to Stop firewall on CentOS 8


#### get status 


```shell
[root@localhost dev]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor p>
   Active: active (running) since Fri 2021-10-08 03:40:54 EDT; 2h 7min ago
     Docs: man:firewalld(1)

```



#### stop it

```shell
[root@localhost dev]# service firewalld stop
Redirecting to /bin/systemctl stop firewalld.service

```


#### disable firewalld

```shell
[root@localhost dev]# systemctl disable firewalld
Removed /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```


#### get status 


```shell
[root@localhost dev]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor >
   Active: inactive (dead) since Fri 2021-10-08 05:49:02 EDT; 1min 59s ago
     Docs: man:firewalld(1)

```

##### restart docker


```shell
[root@localhost dev]# systemctl restart docker
```


==> NOT WORKING 



try to add 172.17.0.1 docker.host.internal

```shell
[root@localhost dev]# echo "172.17.0.1 docker.host.internal" >> /etc/hosts 
[root@localhost dev]# cat /etc/hosts 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
127.0.0.1   host.docker.internal
172.17.0.1 docker.host.internal
```

==> not working



2. This config not working :

```yml
version: '3'
services:
  prisma:
    image: prismagraphql/prisma:1.34
    restart: always
    ports:
    - "4477:4477"
    environment:
      PRISMA_CONFIG: |
        port: 4477
        # uncomment the next line and provide the env var PRISMA_MANAGEMENT_API_SECRET=my-secret to activate cluster security
        # managementApiSecret: my-secret
        databases:
          default:
            connector: postgres
            host: 172.17.0.1
            database: postgres
            schema: public
            user: postgres
            password: yrt
            rawAccess: true
            port: '5432'
            migrations: true
```

==> not working


2. This config not working :

```yml
version: '3'
services:
  prisma:
    image: prismagraphql/prisma:1.34
    restart: always
    ports:
    - "4477:4477"
    environment:
      PRISMA_CONFIG: |
        port: 4477
        # uncomment the next line and provide the env var PRISMA_MANAGEMENT_API_SECRET=my-secret to activate cluster security
        # managementApiSecret: my-secret
        databases:
          default:
            connector: postgres
            host: 172.19.0.1
            database: postgres
            schema: public
            user: postgres
            password: yrt
            rawAccess: true
            port: '5432'
            migrations: true
```

==> not working


3. 

```yml
version: '3'
services:
  prisma:
    image: prismagraphql/prisma:1.34
    restart: always
    extra_hosts:
        - "host.docker.internal:host-gateway"
    ports:
    - "4477:4477"
    environment:
      PRISMA_CONFIG: |
        port: 4477
        # uncomment the next line and provide the env var PRISMA_MANAGEMENT_API_SECRET=my-secret to activate cluster security
        # managementApiSecret: my-secret
        databases:
          default:
            connector: postgres
            host: host.docker.internal
            database: postgres
            schema: public
            user: postgres
            password: yrt
            rawAccess: true
            port: '5432'
            migrations: true
```
not working