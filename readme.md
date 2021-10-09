✔️ Solved :

The answer is to use the output of this command :

```shell
ip route | grep docker0 | awk '{print $9}'
```

OR

```shell
[root@localhost dev]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:e6ff:fe93:a426  prefixlen 64  scopeid 0x20<link>
        ether 02:42:e6:93:a4:26  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1  bytes 90 (90.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

and use the line :

```shell
inet 172.17.0.1
```
so the yml file look like this :

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
            password: 'YOUR_PASS'
            rawAccess: true
            port: '5432'
            migrations: true
```

IMPORTANT:
my /etc/hosts is as default :

```shell
[root@localhost dev]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```


issue refs :

https://stackoverflow.com/questions/69494586/host-docker-internal-not-working-on-prismagraphql-prisma1-34-on-docker-compose

https://github.com/docker/compose/issues/8771

https://github.com/moby/moby/issues/42919

https://github.com/prisma/prisma/issues/9677
