---
layout: post
title:  "MongoDB on Docker"
sub_title: "docker container run..."
date:   2020-03-14 12:00:00 -0600
categories: mongodb docker
---
MongoDB on Docker - build and run mongodb-enterprise

1. [Build a MongoDB Docker Image](#build-a-mongodb-docker-image)
1. [Run the MongoDB Image](#run-the-mongodb-image)
1. [Storing data on the Host](#storing-data-on-the-host)
1. [Stop the MongoDB Container](#stop-the-mongodb-container)
1. [Access MongoDB](#access-mongodb)
1. [Requirements](#requirements)
1. [References](#references)

## Build a MongoDB Docker Image

The Dockerfile allows for building 2 types of images, one for [MongoDB Community](https://www.mongodb.com/download-center/community) and one for [MongoDB Enterprise](https://www.mongodb.com/download-center/enterprise).  This is configured by setting the MONGO_PACKAGE and MONGO_REPO variables as a build-arg.

Build the MongoDB image from files in [Docker Hub](https://hub.docker.com/_/mongo/) and configure it to use [MongoDB Enterprise](https://www.mongodb.com/download-center/enterprise).  Export the following build-time environment variables, download Docker assets and build a tagged image.

```bash
export DOCKER_USERNAME=corbsmartin
export MONGODB_PACKAGE=mongodb-enterprise
export MONGODB_REPO=repo.mongodb.com
export MONGODB_VERSION=4.2

curl -O --remote-name-all https://raw.githubusercontent.com/docker-library/mongo/master/$MONGODB_VERSION/{Dockerfile,docker-entrypoint.sh}
chmod 755 ./docker-entrypoint.sh
docker build \
  --build-arg MONGO_PACKAGE=$MONGODB_PACKAGE \
  --build-arg MONGO_REPO=$MONGODB_REPO \
  -t $DOCKER_USERNAME/$MONGODB_PACKAGE:$MONGODB_VERSION .

# output build process, laying the image
Sending build context to Docker daemon  18.43kB
...
Successfully built 4660c448dfe2
Successfully tagged corbsmartin/mongodb-enterprise:4.2
```

## Run the MongoDB Image

* Ensure the MongoDB image is listed in your local Docker repo
* View image details
* Run the image as a Container instance

```bash
docker image ls
REPOSITORY                     TAG  IMAGE ID     CREATED        SIZE
corbsmartin/mongodb-enterprise 4.2  4660c448dfe2 4 minutes ago  524MB

# detail info
docker image inspect corbsmartin/mongodb-enterprise:4.2

# run
docker run --name m0 -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=main_user \
  -e MONGO_INITDB_ROOT_PASSWORD=changeme \
  -itd corbsmartin/mongodb-enterprise:4.2

# list container
docker container ls
# inspect
docker container inspect m0
# view logs
docker container logs --tail 100 m0
# execute mongo command inside the container
docker exec -it m0 /usr/bin/mongo --eval "db.version()"
```

## Storing data on the Host

By default db data will be saved and managed by Docker and for dev environments this works quite well as long you don't remove/delete the container (stopping is ok).  If you'd like to isolate db files from the container lifeline then mount storage from the host into the container.

```bash
# create a spot for each mongod instance's data files
mkdir -p ~/docker-mounts/mongodb/m0
# mount as a volume inside the container
docker run --name m0 -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=main_user \
  -e MONGO_INITDB_ROOT_PASSWORD=changeme \
  -v ~/docker-mounts/mongodb/m0:/data/db \
  -itd corbsmartin/mongodb-enterprise:4.2
```

## Stop the MongoDB Container

Stopping the container will not destroy data, it will be saved in whatever state it was prior to stopping.  However removing the container from Docker will.

```bash
# stop
docker container stop m0
# remove (this will remove your data)
docker container rm m0
```

## Access MongoDB

The MongoDB standalone instance will be available from your host on port 27017 (if you ran as stated ^^^).  Connecting with the mongo shell is as easy as...

```bash
mongo --username main_user
# or
mongo mongodb://127.0.0.1:27017 --username main_user
```

You can also exec into an interactive shell using Docker.

```bash
# docker exec -it <NAME-OF-CONTAINER> sh
docker exec -it m0 sh
```

## Requirements

1. [Docker for your OS](https://www.docker.com/products/docker-desktop)
2. [An account on Docker Hub](https://hub.docker.com/)
3. [MongoDB Image](https://hub.docker.com/_/mongo/)

## References

1. [MongoDB on Docker Docs](https://docs.mongodb.com/manual/tutorial/install-mongodb-enterprise-with-docker/)
