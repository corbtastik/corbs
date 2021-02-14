---
layout:     post
title:      "Docker for Devs"
sub_title:  "one pager cheat sheet"
date:       2021-01-30 00:00:00 -0600
categories: docker containers development
---

A curated set of information on Docker for developers seeking a simple one pager reference.  This post serves as a "cheat sheet" on core Docker concepts and stops short of "container orchestration" which is a larger topic centered around the Kubernetes ecosystem.

## Docker Images

Docker images are portable applications that have gone through a source->to->image build process (aka "Dockerization").  Building Docker images is a deep topic unto itself and not covered in detail here.  We'll be sticking with configuring and running existing images.  That said there's a few things worth knowing about Docker Images before we dive in.

### Building

Docker Images are built using a `Dockerfile` which includes instructions to package source content into an Image.  Images are "layered", you start with a Base Image and add subsequent layers as part of the build process.

* Google has an excellent guide on best practices for building containers [here](https://cloud.google.com/solutions/best-practices-for-building-containers).

### Storing

Docker Images are inventoried in Registry such as the publicly available [Docker Hub](https://hub.docker.com).  When you install Docker it comes pre-baked with a local Repository which is used when we "pull" and "build" Images.

* Registries contain Repositories
* A Repository contains Images
* Images contain your application

Let's look at a short example to build and share an Image, such as the one for this blog.  We'll show two takes on building the Image.

* Pre-built static HTML to Image

* Jekyll Site Image


---

## Docker Run

Running containers with docker is something you'll likely be doing a lot of.  A development environment equipped with docker is powerful and supports "shift left" thinking where developers are challenged to do more and do it early.  Java became popular due to it's "write once, run anywhere" promise, similarly Docker provides the same promise yet on a broader scale.  Containers are becoming the defacto way to package and run "processes" that would otherwise be quite taxing for the developer to install, run and maintain locally as standalone packages.

---

### Running Containers

Docker **run** is a command that starts a Container on a "Docker Host" (aka your dev machine).  The first time you run a container its image is pulled from a registry (such as Docker Hub).  Images are pulled by **name:tag**, if tag isn't specified then the **latest** image is pulled by default.

```bash
# run the 'latest' image by default
docker run redis

# run a tagged image
docker run redis:latest
docker run redis:5.0.10
```

### Input/Output

Virtually any type of application can be containerized, shared and executed with Docker, including shell scripts, interactive cli(s), compiled programs and tasks.  Docker makes it possible for DevOps enthusiasts to deliver crafty automation written in BASH, Perl, Python, C, etc...in a portable format.  Typically such things require reading from stdin and writing to stdout/stderr as well as running in interactive mode.  All of which can be accomplish with **docker run**.

#### Commands and Arguments in Docker

How a Container behaves at runtime is dependent on how it's built, including how arguments are handled.  Docker essentially provides two ways to define how arguments are passed to the Container (excluding env-vars).

Often __CMD__ and __ENTRYPOINT__ are used together to achieve the intended startup effect.  The main thing to remember is how they're different.  Any command line arguments appended to the end of __docker run__ will override __CMD__ in the Dockerfile at runtime.

```bash
#--------------------------------------------------------------------------------------#
# Dockerfile snippet             docker run                  result                      
#--------------------------------------------------------------------------------------#
CMD ["sleep", "1"]               docker run sample           sleep 1s                    
CMD ["sleep", "1"]               docker run sample 3         exec error, 3 not a command 
CMD ["sleep", "1"]               docker run sample sleep 3   sleep 3s, but wonky cli     
CMD ["sleep", "1"]               docker run sample date      no sleep, prints date       
ENTRYPOINT ["sleep"]             docker run sample           usage error, no sleep arg   
ENTRYPOINT ["sleep"]             docker run sample 3         sleep 3s                    
ENTRYPOINT ["sleep"]             docker run sample sleep 3   usage error, invalid number 
ENTRYPOINT ["sleep"]             docker run sample date      usage error, invalid number 
CMD ["1"] ENTRYPOINT ["sleep"]   docker run sample           sleep 1s                    
CMD ["1"] ENTRYPOINT ["sleep"]   docker run sample 3         sleep 3s                    
CMD ["1"] ENTRYPOINT ["sleep"]   docker run sample sleep 3   usage error, invalid number 
CMD ["1"] ENTRYPOINT ["sleep"]   docker run sample date      usage error, invalid number 
#--------------------------------------------------------------------------------------#
```

---

## Networking

### Publishing Ports

Many applications expose functionality over ports, for example servlet containers, redis, kafka and others.  If you need to access the container over a network then run with "publishing".  Publishing a container port on the Docker Host can be bound to a specific interface or published "unbounded".  The former only allows access from the specific interface while the later is accessible from the outside.

```bash
# publish container port (8080) to a bound interface (127.0.0.1) and port (80) on the docker host
#   container only accessible from bound interface
docker run -p 127.0.0.1:80:8080 corbsmartin/todos-webui-embedded

# publish container port (8080) to an unbounded interface and port (80) on the docker host
#   container accessible from any interface on the docker host network
docker run -p 80:8080 corbsmartin/todos-webui-embedded
```

---



## References





