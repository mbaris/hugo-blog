---
title: "Anatomy of a Dockerfile"
date: 2021-05-02T00:00:36+02:00
draft: false
tags: ["devops"]
description: "Dockerfile is a text document containing commands which can be run in sequence to assemble a docker image"
---

## What is a Dockerfile?
Dockerfile is a text document containing commands which can be run in sequence to assemble a docker image.

[A sample Dockerfile from the official docs](https://docs.docker.com/get-started/02_our_app/) looks like this

``` dockerfile
# syntax=docker/dockerfile:1
FROM node:12-alpine
RUN apk add --no-cache python g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

## Why do we need them?
There are millions of images on [dockerhub](https://hub.docker.com/search?q=&type=image) that we can directly start using with a command like this.
``` bash
docker run -it --rm -d -p 8080:80 --name web nginx
```

For various reasons, we may want to customize these base images. Docker images are immutable, so we can't _exactly_ modify them.
We can technically run a container using an existing image, make some changes on it and then create a _new_ image with these modifications using the *commit* command but there is a better way to accomplish this task 
However, before we start modifying images, we need to understand the concept of [layers in docker](https://docs.docker.com/storage/storagedriver/)

### Images, Layers and Containers
Each Docker container consists of a readable and writable layer on top of multiple read only layers.
These read only layers represent instructions in Dockerfiles, and they are deltas on previous layers(similar to git commits)

Multiple containers can share the underlying layers since they have their own writable/readable layer on top.
The readable and writable layer is a thin layer which has a lifespan associated with the container.

![Docker Layer Sharing](/images/dockerfile/layers.jpeg)
_Docker Layer Sharing from [About storage drivers](https://docs.docker.com/storage/storagedriver/)_
``` dockerfile
# syntax=docker/dockerfile:1
FROM tiangolo/uvicorn-gunicorn-fastapi:python3.7
LABEL maintainer="Mehmet Baris Kalkar"
LABEL version="1.1"
RUN addgroup api && adduser fast && adduser fast api 
USER fast:api
ENV GREETING="hola"
COPY ./app /project/app
WORKDIR /project
EXPOSE 8090
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8090"]
```

If we create a container from this same dockerfile, we will see a log similar to this:

``` bash
 => [1/4] FROM docker.io/tiangolo/uvicorn-gunicorn-fastapi:python3.7@sha256:a0e0188a485fd8c232d8774ae4680d3b834f95dd2deccdb0211ce71cfd778b97
 => [internal] load build context
 => => transferring context: 56B
 => [2/4] RUN addgroup api && adduser fast && adduser fast api
 => [3/4] COPY ./app /project/app 
 => [4/4] WORKDIR /project 
 => exporting to image 
 => => exporting layers 
 => => writing image sha256:3cef1a7b7ddc037fa375a1fb37daa907bc31031fedb4142b98e98e582c0bead5
 => => naming to docker.io/library/fastapi
```

One important thing to understand is [how these instructions are cached](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).
The result of some commands like FROM, COPY/ADD, RUN and WORKDIR can be cached.

Cached instructions are marked in the build command. If we build the same image by changing only the WORKDIR instruction to project2, we would see something like this.
``` bash
 => CACHED [1/4] FROM docker.io/tiangolo/uvicorn-gunicorn-fastapi:python3.7@sha256:a0e0188a485fd8c232d8774ae4680d3b834f95dd2deccdb0211ce71cfd778b97
 => [internal] load build context
 => => transferring context: 56B
 => CACHED [2/4] RUN addgroup api && adduser fast && adduser fast api
 => CACHED [3/4] COPY ./app /project/app
 => [4/4] WORKDIR /project2
 => exporting to image
 => => exporting layers
 => => writing image sha256:fe482845750cf79708d1a6cc107578e76bd843f92fb3092d636180547b32b897
 => => naming to docker.io/library/fastapi   
```

Let's take a look at this Dockerfile line by line

``` dockerfile
# syntax=docker/dockerfile:1
```
(Optional) *[syntax](https://docs.docker.com/engine/reference/builder/#syntax)* is only enabled if we are building the image with [BuildKit](https://docs.docker.com/engine/reference/builder/#buildkit)
In this line, we can inform the Dockerfile builder which syntax to use while parsing the Dockerfile

``` dockerfile
FROM tiangolo/uvicorn-gunicorn-fastapi:python3.7
```
*FROM* instruction is used to set the base image that we are going to use. 
This should always be the first instruction in a Dockerfile.

``` dockerfile
LABEL maintainer="Mehmet Baris Kalkar"

LABEL version="1.1"
```
*LABEL* instructions are used to add metadata to images.

Side note, There used to be a MAINTAINER instruction in the past, but it is deprecated now.

``` dockerfile
RUN addgroup api && adduser fast && adduser fast api 
```

*RUN* instruction is used to execute commands in a new layer on top of the current image and commit changes.
Following steps will use the new image.

``` dockerfile
USER fast:api
```
*USER* instruction sets the user and group for the following steps.

``` dockerfile
ENV GREETING="hola"
```

*ENV* is used to add environment variables to the container. This variable can be used in the following steps during build as well.
If we want to use a variable in only a single command and not in the image, we can define use the RUN command with a variable instead.
``` dockerfile
RUN LOCUST_LOCUSTFILE=custom_locustfile.py locust
```

``` dockerfile
COPY ./app /project/app
```
*COPY [--chown=<user>:<group>] <src>... <dest>* copies files from source and adds it to the file system of the container
Target path is always relative to the working directory.

*ADD* command also has a similar function, but it can also be used to fetch files from a remote URL or extract tar files. 

[It is preferred to use COPY](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy) 
over add because COPY is a more transparent and simple instruction.  

``` dockerfile
WORKDIR /project
```
*WORKDIR* Sets the working directory to run instructions like CMD, RUN, ENTRYPOINT and COPY after this step.

``` dockerfile
EXPOSE 8090
```
*EXPOSE* is an informational instruction. It does not actually publish any ports, but it is used as a documentation to let 
users know which ports should be published to use the image.

``` dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8090"]
```
*CMD* is the instruction to define the command you want to execute when run a container from an image. 
It is possible to override this command while actually running the image, so it acts as a default. 

*ENTRYPOINT* and *CMD* are similar commands, [the differences are explained here pretty well](https://www.ctl.io/developers/blog/post/dockerfile-entrypoint-vs-cmd/)

*VOLUME* command is used to create mounting points within the container. 
We can use these volumes to [share files between containers or the native host](https://docs.docker.com/storage/volumes/). 

