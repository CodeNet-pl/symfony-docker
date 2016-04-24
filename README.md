Docker image for symfony applications
===

This is a Dockerfile to build a container image hosting symfony applications.
Purpose of this docker is not only to provide a docker image but also to bootstrap an entire docker driven environment hosting multiple symfony applications.

Installation
===

To install and use this docker image on an existing symfony application create a docker-compose.yml in the project root.
Sample docker-compose.yml file using symfony3 directory structure, MongoDB, GrayLog and MailHog:

```yaml
version: '2'
services:
  web:
    image: codenet/symfony
    hostname: web
    volumes:
      - .:/var/www
    links:
      - mongo
      - mailhog
  mongo:
    image: mongo
    hostname: mongo
    volumes:
      - ./var/mongo/db:/data/db
      - ./var/mongo/config:/data/configdb
  log:
    image: graylog2/allinone
    hostname: log
    volumes:
      - ./var/graylog/data:/var/opt/graylog/data
      - ./var/graylog/logs:/var/log/graylog
    environment:
      GRAYLOG_PASSWORD: your_password
  mailhog:
    image: mailhog/mailhog
```

Hostnames
---

Add a symfony container IP to your `/etc/hosts` file to ease access to symfony website.
To find out the container IPs you can issue command below in your project directory and add hosts to your /etc/hosts

```bash
docker ps -q | xargs docker inspect --format "{{ .NetworkSettings.Networks.${PWD##*/}_default.IPAddress }} {{ index .Config.Labels \"com.docker.compose.service\" }}.${PWD##*/}.lo" | sed s/web.//
```

Helper scripts
---

To ease of issuing symfony commands and running various commands inside a containers you can create two scripts inside `bin` directory:

bin/docker-exec (executes bash command inside a container)
```bash
#!/bin/bash

if [[ -z $1 ]]; then
    echo "Usage: docker-exec command [container]"
    exit 1
fi

args=$@
dir=${PWD##*/}
command=$1
container=web
if [[ ! -z $2 ]]; then
    container=$2
fi

docker exec -it ${dir}_${container}_1 $command
```

bin/docker-console (executes a symfony command inside a container)

```bash
#!/bin/bash

args=$@
dir=${PWD##*/}
docker exec -it ${dir}_web_1 bash -c "cd /var/www && php bin/console $args"
```

