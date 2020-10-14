---
layout: "post"
title:  "A Quick Introduction to Docker"
tags:   linux docker
---

> Docker simplifies application deployments that should run in "lightweight virtual-machines" within isolated environments.

* * *

I shall briefly describe Docker, focused on its principles and how it may simplify the way we distribute
our applications by using containers that package the application alongside its dependencies.

# Docker in a Nutshell

Docker is a technology that simplifies the distribution of the applications running "inside" isolated containers, or rather at least conceptually. 

Roughly speaking, we can consider containers as "lightweight virtual
machines". They offer a level of isolation for applications while sharing the underlying OS kernel. Therefore, they generally consume less computing resources than traditional virtual machines and are also quicker to spin up. However, there's the flip side, which is the lower level of isolation, and that may or may not be an issue for your project.

## Basic Concepts

Roughly speaking, a container is like an instance of an image, and an image is a recipe of how to build containers.

Images are "static", while containers are "dynamic". It's conceptually similar to Object
Oriented Programming, say an image is to a class what a container is to an object.

Interestingly, images are layered, that is, images are composed by aggregating of "check-points". For every command that you put
inside the image recipe, a layer is created. You can check out some past layer when necessary.
For example, suppose that the first layer downloads an Ubuntu image, and the second downloads python3, should the download of python3
have failed then you won't need to download Ubuntu again because its layer was already cached.

## Docker Client

_docker_ is the client which communicates with the Docker daemon (_dockerd_), responsible for executing commands on the client's behalf.

The docker client offers a variety of commands, let's try some of them.

To download the latest version of the Alpine OS image from the remote repository into your local cache:

```bash
docker pull alpine:latest
```

To verify all images that you have downloaded:

```bash
docker images
```

To run a container based on the downloaded `alpine:latest` image in interactive-mode:

```bash
docker run -it alpine:latest
```

To list all containers that are running:

```bash
docker ps
```

To stop a given container (you have to replace `$CONTAINER_ID` with the actual container Id obtained by `docker ps`):

```bash
docker stop $CONTAINER_ID
```

## Dockerfile

Now that we have seen some basic commands, let's create an image?

First off, we need to write a _Dockerfile_, that is, a receipt that the docker will consume to create an image based on it.

A _Dockerfile_ describes a sequence of commands, each with the following structure:

> INSTRUCTION param1 param2 … paramN

Where `INSTRUCTION` is a valid docker command that receives `_param1_`, `_param2_` , …, `_paramN_` parameters.

### Example

As an example, let's create a fairly simple Linux image with the Alpine OS and Python3 installed, and then run a
"Hello World" program written in Python when the container starts up.

Firstly, create a new directory, e.g. _/opt/dockertest_, and inside of it,
add a file _hello-docker.py_ with the following python script:

<script src="https://gist.github.com/rvarago/179cc1c69414347c1f368acd46bf2637.js"></script>

In the same directory, create a _Dockerfile_ with this content:

<script src="https://gist.github.com/rvarago/9ba1549057bfd7e09a956d770b9939f4.js"></script>

In the Dockerfile, every command generates a new layer inside the image.

To build the image and give it the name _alpine-python3-hello_:

```bash
docker build -t alpine-python3-hello .
```

Docker will execute the commands and generate a new image that will be available to instantiate into containers:

```bash
docker run alpine-python3-hello
```

Finally, you should see the output "Hello World from Docker!" on the standard output.

# Conclusion

We've discussed the basics of containers and how to use them
with Docker, then, we saw some commands followed by an example of how to
create an image.

Containers are becoming more and more popular in the software industry, for
example, in cloud-based environments where there is a variety of tools
emerging (Docker, Kubernetes etc).

I hope that with this quick introduction you should be able to start playing with
Docker, or other container technology, in your projects and profit from it.

### References

[1] [https://docs.docker.com/get-started](https://docs.docker.com/get-started/)

[2] [https://github.com/docker/labs](https://github.com/docker/labs)

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
