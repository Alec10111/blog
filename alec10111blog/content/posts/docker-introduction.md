---
title: "Docker Fundamentals"
date: 2023-02-28T15:38:07+01:00
---

## Introduction

Docker is an essential part of modern software development. This post serves as an introduction to the fundamentals of Docker, including a practical example.

## What is Docker?

Docker is an open-source platform written in Go that allows developers to create, deploy, and run applications in containers, providing a consistent and portable environment across different machines and operating systems.

Instead of emulating an entire OS like a virtual machines, Docker shares the host operating system kernel and only requires the necessary libraries and dependencies to run the application, resulting in lower resource utilisation.

With docker, the development environment can be easily reproduced. This ensures that everyone on the team is working with the same dependencies, configurations, and versions, hence reducing compatibility issues and errors.

Docker's fundamental blocks are containers and images.

## Containers

A container is a process that creates an isolated environment to run your software with its own file system, networking, and resources. This makes it possible to run multiple applications on the same server, without conflicts or interference between them.

Although they are isolated, containers may communicate with other containers using a standardised interface (Docker networks).

They can run on local machines, virtual machines, or cloud-based servers, and can be easily scaled up or down on traffic demand.

## Images

Docker images are templates from which containers are created, this is, a set of instructions on how to build and configure them. To create an image you need a Dockerfile, a file containing commands to install and configure the necessary software, copy the application code to the container, expose the appropriate ports for communication and so on.

Images can be stored in a remote registry, such as Docker Hub, accessible to other developers or team members. Docker images are immutable, meaning that they cannot be changed once they are created. This ensures consistency and reproducibility across different environments and deployments.

Any modifications or updates must be made by building a new image based on the original one. (the FROM command on the Dockerfile)

**Note:** Containers can be modified during runtime, but such changes are not reflected in the original image and are lost when the container is deleted or stopped.

## Practical example

To demonstrate how docker works, we will build a small FastApi server with Python running inside a docker container.
The only pre-requisite is that you have docker installed on your system. The easiest way is to get Docker desktop, which already features the docker cli.

Create app directory and get inside

```shell
mkdir fastapi-docker
cd fastapi-docker
```

Create a requirements file. Another way of doing this would be to manually install dependencies in a virtual env and use `pip freeze`. Although this second option requires you to have a development environment in your local system before you add docker, it is a common practice, sometimes beneficial if you have an IDE that can pick up on your dependencies and offer syntax highlighting or autocompletion.

```shell
touch requirements.txt
```

```txt
### fastapi-docker/requirements.txt ###

fastapi
uvicorn
```

Create the application file

```shell
touch main.py
```

```python
### fastapi-docker/main.py ###

from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

Create a Dockerfile. Its important that it is named exactly Dockerfile.

```shell
touch Dockerfile
```

```Dockerfile
### fastapi-docker/Dockerfile ###

# Base image that already has python and other stuff installed
FROM python:3.9

# Create working directory inside the container
WORKDIR /code

# Copy requirements file inside the container
COPY ./requirements.txt /code/requirements.txt

# Install necesary dependencies
RUN pip install -r /code/requirements.txt

# Copy application file to the container
COPY ./main.py .

# Run command to start the application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```

Build the image from the Dockerfile in the current directory and list available images.

```shell
docker build -t image-name .
docker image ls
```

Start a docker container from the given image and list running containers.

```shell
docker run --name fastapi-container -p 8000:80 fastapi
docker ps
```

The first command creates the Docker container from the designated image, maps port 80 inside the container to port 8000 on your local machine (remember containers have isolated network systems) and starts the FastAPI server using the command given on the CMD statement of the Dockerfile.
You can access the FastAPI server by visiting `http://localhost:8000` in your web browser or from the terminal using `curl`.

```shell
curl localhost:8000
```

Congrats, you now have a basic FastAPI server running inside a Docker container, which can be easily deployed to any platform that supports Docker.

-   You can stop and re-start the container using the commands `docker stop <container-name>` and `docker start <container-name>`
-   Spawn another container by re-running the previous `docker run` command.
-   Get inside the container using bash: `docker exec -it fastapi-container bash`
-   Remove the container using `docker rm <container-name>`
-   Remove the image using `docker rmi <image-id>`

## Further topics

There are still some more advanced concepts about containers and images that one could learn, but these are the basics to get going. Aside of those, if you want to dive more into docker, you ought to check the following:

-   **Volumes**: Docker volumes are a way to persist data generated by Docker containers. They can be used to share data between containers or to store data that needs to persist beyond the lifetime of a container.

-   **Networks**: An interface that allows containers to connect to each other and to other services outside the container environment, such as databases or APIs.

-   **Docker compose**: Tool for defining and running multi-container Docker applications. It allows you to define the services, networks, and volumes for your application in a single YAML file and then start the entire application with one command.

-   **Container Orchestration**: It is the process of managing and automating the deployment, scaling, and operation of multiple containers. Docker offers a native orchestration system called `docker swarm`, although the most popular option is `Kubernetes`.

## References

-   [Docker Tutorial: Quick, Easy & Effective Guide to Get Started Developing Go Apps](https://dev.to/docker/docker-tutorial-quick-easy-effective-guide-to-get-started-developing-go-apps-2b08)
-   [FastAPI in Containers - Docker](https://fastapi.tiangolo.com/deployment/docker/)
-   [Docker Mastery: with Kubernetes + Swarm from a Docker Captain](https://www.udemy.com/course/docker-mastery/)

## On the use of AI Language models

I used ChatGPT to generate some of the content for this blog post, which I then edited and refined to ensure it captures and highlights the topics I want to point out and remains original. The use of this AI language model allows me to quickly generate a starting point that I can edit and personalise to create the final post you see.
