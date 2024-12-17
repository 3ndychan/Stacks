

Update the following variables in .env. DO NOT CHECK IN THE FILE IF THERE ARE SECRETS. There will be a way in the future to handle secrets through an encrypted vault that can be shuttled around and checked in with the rest, but to not expose exploits do ya due diligence.

```
# General settings
TZ=America/Eastern
PUID=1000
PGID=1000

# MongoDB settings
MONGO_USER=unifi
MONGO_PASS=
MONGO_DBNAME=unifi
MONGO_HOST=unifi-db
MONGO_PORT=27017
MONGO_AUTHSOURCE=admin
MONGO_TLS=false
MONGO_INITDB_ROOT_PASSWORD=

# Unifi specific settings
MEM_LIMIT=1024
MEM_STARTUP=1024
```

Otherwise this is pretty simple you need following:
software:
   - docker 
   - docker compose
The user that you will be running this as will(may) need to be a part docker group which will require you to logout and then back in before continuing.

This container stack has the following containers:
   -  unifi-network-application:
        image: lscr.io/linuxserver/unifi-network-application:latest
        volumes:
           - ./etc/unifi-controller:/config
        ports:
          - 8080:8080
          - 8443:8443
          - 3478:3478/udp
          - 10001:10001/udp
          - 5514:5514/udp
   - unifi-db:
        image: docker.io/mongo:6.0-jammy
        volumes:
          - ./data:/data/db
          - ./scripts/init-mongo.sh:/docker-entrypoint-initdb.d/init-mongo.sh:ro

The following script will be required as a part of the first time setup of the mongo db server that is pair with this. The way I understand it they have it so that it'll only run once to setup and create the unifi database.

scripts/init-mongo.sh
```
#!/bin/bash

if which mongosh > /dev/null 2>&1; then
  mongo_init_bin='mongosh'
else
  mongo_init_bin='mongo'
fi
"${mongo_init_bin}" <<EOF
use ${MONGO_AUTHSOURCE}
db.auth("${MONGO_INITDB_ROOT_USERNAME}", "${MONGO_INITDB_ROOT_PASSWORD}")
db.createUser({
  user: "${MONGO_USER}",
  pwd: "${MONGO_PASS}",
  roles: [
    { db: "${MONGO_DBNAME}", role: "dbOwner" },
    { db: "${MONGO_DBNAME}_stat", role: "dbOwner" }
  ]
})
EOF



Resources
https://docs.linuxserver.io/images/docker-unifi-network-application/#setting-up-your-external-database

https://gist.github.com/pdscomp/791bae497fd54661ac2d4b33f5cbd089
