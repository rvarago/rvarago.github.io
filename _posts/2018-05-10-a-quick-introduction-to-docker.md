---
layout:	"post"
title:	"A Quick Introduction to Docker"
---

Docker is a technology that simplifies the deployment of applications by using
containers that run in isolation. Roughly speaking, you can think about a
container as a "lightweight virtual machine"…

* * *

Hello,

Today, I'm going to talk about one of the tools that are commonly applied to
DevOps practices: Docker.

I intend to describe its principles and how it can simplify the distribution
of your application by the use of isolated containers that can package
together the application and its dependencies.

In order to be a quick description, I'll omit a lot of concepts and commands,
but I advise you to search more about them, for example, in the references
section.

### What is Docker?

Docker is a technology that simplifies the distribution of the applications
based on containers that run in isolation.

Roughly speaking, you can think about a container as a "lightweight virtual
machine", it offers a degree of isolation for your applications. But a
container shares the OS kernel, so it can be more parsimonious regarding the
system's resources than a traditional virtual machine, but paying the price of
a lower level of isolation.

A container represents an abstraction at the application level that packages
together: the application and it's dependencies, each running as an isolated
process in the user space, sharing the OS kernel. Because it shares the OS
kernel, a container doesn't need to boot a kernel, thus it can run very
quickly and it's one of the most interesting advantages of containerization.

### Basic Concepts

A container is based on an image that describes what needs to be shipped
together, for example, compilers, interpreters, web servers, source code etc.

Basically, an image is a "receipt" of how to build containers, it's a "static
thing" while a container is the actual execution of an image, it's a "dynamic
thing". You can approximately compare the two concepts with the Object
Oriented Paradigm, saying that an image is for a class as a container is for
an instance of the class, an object!

An interesting aspect of images is that they are layered, that is, an image is
composed by the aggregation of "checkpoints". For every command that you put
inside the image, a layer is created, and you can checkout some past layer
when necessary. For example, suppose that the first layer is the download of
Ubuntu and the second is the download of Python3, if the download of Python3
has failed you won't be required to download Ubuntu again, because its layer
was already correctly generated.

Docker has a on-line server that acts as an image repository, where you can
distribute your images and download images developed by others. You can also
configure your own repository.

### The Docker Client

When you install Docker, you get access to the Docker's client ( _docker_ )
that communicates with the Docker's daemon ( _dockerd_ ), responsible for
actually execute the commands

The docker client offers a variety of commands, let's try some of them!

* * *

To download the latest version of the Alpine OS image from the remote
repository into your local cache:

> docker pull alpine:latest

Now, to verify the downloaded images:

> docker images

To run a container based on the downloaded alpine:latest image and getting
access to its terminal in interactive more:

> docker run -i -t alpine:latest

Have you seen how fast it was started? :)

To list the running containers, open a new tab in your terminal and type:

> docker ps

Now, to stop the container (you must replace CONTAINER_ID) with your actual
container Id obtained by the _docker ps_ :

> docker stop CONTAINER_ID

Docker has many more commands, so I recommend you to look at the references to
know more about them.

### Dockerfile

Now that you have some basic knowledge about Docker, how about create your own
image?

To create an image, you need to write a receipt that the docker client will
read and create the image based on it. This receipt has a name: _Dockerfile_.

A Dockerfile is composed by a sequence of commands with the following
structure:

> INSTRUCTION param1 param2 … paramN

Where _INSTRUCTION_ is a valid docker command that receives _param1_ ,
_param2_ , …, _paramN_ parameters.

### Example

To introduce the syntax of the Dockerfile, let's go to an example! The task is
to create an Linux image with the Alpine OS with Python3 installed and run a
simple "Hello World" program written in Python during the container startup.

Firstly, create a directory (in my case, _/opt/dockertest_ ). Inside it,
create a file _hello-docker.py_ and copy this python script into it:

Then, in the same directory, create a Dockerfile with this content:

In the Dockerfile, every command generates a layer inside the image.

Firstly, I've stated to docker that our image will be based on an existing
image for the Alpine OS. Then we named the maintainer. Updated the package
manager index and installed python3 in the image. After, we copied our script
_hello-docker.py_. Finally, we defined the default command after the container
startup will be to run python3 to execute _hello-docker.py_.

Now, to build the image and give it the name _alpine-python3-hello_ :

> docker build -t alpine-python3-hello .

Docker will execute the commands and generate a new image that will be
available to instantiate a new container:

> docker run alpine-python3-hello

And you'll see the output " _Hello World from Docker!_ " on your standard
output.

### Conclusion

In this article, we've discussed the basics of containers and how to use them
with Docker, then, some commands were showed, and finally an example of how to
create an image using the Dockerfile.

Containers are becoming more and more popular in software distribution, for
example in cloud computing environments and there are a variety of tools
emerging from this ecosystem (Docker, Kubernetes etc).

I hope that with this quick introduction, you can be able to start applying
Docker or other container technology in your projects and benefit from it.

In the future, I intend to write more about Docker and provide more
information about it, but now, I strongly recommend you to search more about
Docker, its usage, advanced features, and benefits. You can start by looking
at the references section.

### References

[1] [https://docs.docker.com/get-started](https://docs.docker.com/get-
started/)

[2] <https://github.com/docker/labs>


***
*Originally published at https://medium.com/@rvarago*
