Hello,

I want deploy a 'prismagraphql/prisma:1.34' container using :
* docker-compose version (1.29.2)
* docker 20.10.9
* CentOS 8.4.2105

but when i use 'host.docker.internal' on host not working.
So i try to use :

```yml
extra_hosts:
        - "host.docker.internal:host-gateway"
```

My project three files : 


datamodel.prisma :
```graphql
type User {
  id: ID! @id
  name: String!
}
```

prisma.yml :
```yml
endpoint: http://localhost:4477
datamodel: datamodel.prisma
```


docker-compose.yml :
``` yml
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

I try many incidents to disable firewall add to /etc/hosts ... => NOT WORKING


Here i create a GitHub repo :  

