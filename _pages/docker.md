---
title: Docker Setup
author: XXX
date: 01
layout: post
---

## Get Started 
Since there can be many different environments we need to 
setup for development in the AHA project, we use 
**Docker** to make thing easier.

Docker is a tool that allows developers to easily deploy their 
applications in a container to run on the host operating system. 
The key benefit of Docker is that it allows users to 
**package an application with all of its dependencies into a standardized unit**.

But first, you need to contact *Can Wang (canswang@stanford.edu) * to **setup your kiwi account** 
and **get added to the docker group**. 


> ##### WARNING
>
> You might see a `permission denied` error after running docker 
> command. This is because if you're on Linux, then you need to 
> prefix your docker commands with `sudo`. In this case, you should 
> make sure that *Can* had add your account into a docker group to 
> get the permission of using Docker.
{: .block-warning }


## Using docker
Here is a brief overview of **Docker Images** and **Docker Container**.


### Docker Image
A *image* is a **read-only blueprint** of our application which 
**form the basis of containers**. Often, an image is based on another 
image, with some additional customization. 

    docker images

We can use `docker images` command to list all images. For example, 
we can see `stanfordaha/garnet` is the name of the Docker image and 
we will build our containers on top of it. Typically, we would use 
the latest image version called `stanfordaha/garnet:latest`, so we 
can use `docker pull` command to pull docker image from docker hub.

    docker pull stanfordaha/garnet:latest


### Docker Container
Since *images* are just templates, you cannot start or run them. We 
could create a *container* from *image* and run the actual application. 
In other words, **a container is a runnable instance of an image**, 
where we can read, write and modify. 

You can create, start, stop, move, or delete a container using the Docker 
API or CLI.


#### List containers

    docker ps
    docker container ls

We can use `docker ps` command to show all containers that are currently 
running, which is exactly the same function as `docker container ls`.  


#### Create containers

And we can use `docker run` to create a container based on specific image. 
The `-it` flag specifies an interactive terminal which makes it easier to 
kill the container with `ctrl+c` (on windows) and the `--rm` flag 
automatically removes the container when it exits. The `-d` flag will 
detach our terminal so we can happily close your terminal and keep the 
container running. The `--name` flag corresponds to a name we want to give, 
while `<container-name>` is totally self-defined, typically we will use 
**first name + usage** when we create a new container. 

    docker run -it --rm -d -v /cad:/cad --name <container-name> stanfordaha/garnet:latest bash

After running above command, the new container must appear on the list if 
we call `docker ps` again. 


#### Attach, Detach container

To attach to Docker container after we create it, we use `docker attach` command.

    docker exec -it <container-name> bash

When we are currently attach to Docker container, we can use 
`ctrl+p ctrl+q` to detach from the Docker container.


#### Delete container
Now is the dangerous part. When we are currently attach to Docker container, 
we can use `ctrl+d` to delete the docker. When we are currently in the detach 
mode, we can use `docker stop` command to stop a running container. 
    
    docker stop <container-name>


## Update Docker
To get everything up to date, it is better to run `apt update`. Since `vim` is 
not installed in the docker yet, we can install vim to make things easier. 

    apt update
    apt install -y vim



### Other Useful Stuff
To check the history commands:

    history
    git log --graph --oneline --all


## Get the latest version
Best way to get the latest version is to create a new docker. To update an existing docker use:

    git submodule update --init --recursive





## Reference

[1] [https://docker-curriculum.com/](https://docker-curriculum.com/)

[2] [https://docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/)
