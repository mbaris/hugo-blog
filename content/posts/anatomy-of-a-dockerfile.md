---
title: "Docker Containers Part 1"
date: 2021-04-28T00:16:36+02:00
draft: true
tags: ["containers", "devops", "infrastructure"]
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
There are millions of images on [dockerhub](https://hub.docker.com/search?q=&type=image) that we can start using with a simple command.

``` bash
docker run -d -p 80:80 my-nginx service nginx start
```

However, for various reasons, we may want to customize these base images and there are multiple paths we can take to accomplish this.

### Modifying Images
Docker images are immutable, so we can't _exactly_ modify them. 

However, we can run a container using an existing image, make some changes on the container and then create a _new_ image with these modifications using the *commit* command.

For example, we can add a custom index.html file to a running container from the base nginx image and run the commit command to create a new image

``` bash
docker commit my-nginx
```

This method can be used, but it is easy to see how cumbersome it can be if we want to create new images for multiple applications frequently.

### Enter Dockerfiles
Dockerfiles are used to solve the same problem with a different approach. 
Instead of making the changes ourselves, we can write Dockerfiles with necessary steps to create these custom images.

``` dockerfile
# syntax=docker/dockerfile:1
FROM ruby:2.5
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
COPY ../compose /myapp

# Add a script to be executed every time the container starts.
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Configure the main process to run when running the image
CMD ["rails", "server", "-b", "0.0.0.0"]
```

Let's take a look at this file line by line

``` dockerfile
# syntax=docker/dockerfile:1
```
(Optional) *[syntax](https://docs.docker.com/engine/reference/builder/#syntax)* is only enabled if we are building the image with [BuildKit](https://docs.docker.com/engine/reference/builder/#buildkit)
In this line, we can inform the Dockerfile builder which syntax to use while parsing the Dockerfile

``` dockerfile
FROM ruby:2.5
```
*FROM* instruction is used to set the base image that we are going to use. This should always be the first instruction in a Dockerfile.

``` dockerfile
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
```
*RUN* instructions are used to 