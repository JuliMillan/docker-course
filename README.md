# Docker Training: From Zero to Cloud

This repo includes notes from the Docker Training course from Udemy. The course's GitHub is [here](https://github.com/rslim087a/docker-course-remastered/tree/main).

I'll will be getting some concepts directly from [Docker's docs](https://docs.docker.com/get-started/docker-overview/).

## Intro

- **Docker** is a platform that allows us to run applications inside containers
- **Containers** include the application's source code and only the resources the application needs, making it faster and lighter than Virtual Machines, but still isolating the application from the local environment and facilitating reproducibility and consistency.

### Docker objects

- **Images**: Template for creating containers. It includes everything we need to install in them and configurations details.
- **Containers**: runnable instance of an image.
- **Volumes**: persistent data stores for containers. When we create a volume, it's stored within a directory on the Docker host. When we mount the volume into a container, this directory is what's mounted into the container. They are the preferred mechanism for persisting data generated and used by containers.

## Running applications inside Docker Containers

To run an app with docker, we have to mount it in a directory inside our container.

To create a container and run an app in it we type `docker run -v "<directory path in local machine>:<docker working directory>" <image> python /app/python-app.py`

Make sure to add the entire folder to the local directory and not a single file. This will copy all the contents in the folder.

- `docker run` will create a container from the specified image
- `-v` will create a volume where it will mount our local app in our working directory, which is the directory where the container will be running the application in.
- Inside the `""` we write first the local path to our application and after the `:` the working directory
- Then we add the command we will want to run, in this case we use `python` to run the app in the wd

Docker will look for the image locally and, if it's not there, it will download it from Docker Hub.

The final command would be somehting like: `docker run -v "C:\Users\...\course-content:/app/" python:3.8.19-slim python /app/python-app.py`

### Useful commands

- `docker p`s will return a list of all running containers
- `docker ps -a` will return a list of running and stopped container
- `docker rm` will delete specific containers if at the end we add the name or the container id
- To name the containers so the don't get assigned random names, we add the arg `--name` before the image. Naming our apps it's recommended since it will facilitate connections between containers later on
- To remove the container as soon as it stops running, we add `--rm` after `docker run`
- `docker images` returns a list of all the images
- `docker image rm <repository>:<tag>` or `docker image rm <image id>` will delete the specified image. We will need to delete all containers using that image first.