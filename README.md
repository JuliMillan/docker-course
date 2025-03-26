# Docker Training

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

To run an [app](course-content\01-code) with docker, we have to mount it in a directory inside our container.

To create a container and run an app in it we type `docker run -v "<directory path in local machine>:<docker working directory>" <image> python /app/python-app.py`

Make sure to add the entire folder to the local directory and not a single file. This will copy all the contents in the folder.

- `docker run` will create a container from the specified image
- `-v` will create a volume where it will mount our local app in our working directory, which is the directory where the container will be running the application in.
- Inside the `""` we write first the local path to our application and after the `:` the working directory
- Then we add the command we will want to run, in this case we use `python` to run the app in the wd

Docker will look for the image locally and, if it's not there, it will download it from Docker Hub.

The final command would be somehting like: `docker run -v "C:\Users\...\course-content:/app/" python:3.8-slim python /app/python-app.py`

### Useful commands

- `docker p`s will return a list of all running containers
- `docker ps -a` will return a list of running and stopped container
- `docker rm` will delete specific containers if at the end we add the name or the container id
- To name the containers so the don't get assigned random names, we add the arg `--name` before the image. Naming our apps it's recommended since it will facilitate connections between containers later on
- To remove the container as soon as it stops running, we add `--rm` after `docker run`
- `docker images` returns a list of all the images
- `docker image rm <repository>:<tag>` or `docker image rm <image id>` will delete the specified image. We will need to delete all containers using that image first.

## [Building images from Dockerfiles](course-content\02-dockerfiles)

> *Remember*: `docker build` will create an image from a Dockerfile, and `docker run` will run a container from the specified image

**Dockerfiles** indicate how to create an image and run a container and look something like the following:

```docker
FROM python:3.8-slim

WORKDIR /app # Everything we do from here happens in the working directory

COPY python-app.py /app # or . instead of /app

CMD ["python", "python-app.py"]
```

- The first line will tell us what image to use
- The working directory is the path inside of the container where the application will run
- The third layer of the image is going to specify which application we are going to copy to the working directory to run. The second argument can be replaced by a dot. This dot means current directory, and we have defined that working directory is /app, so docker will move there automatically.
- The last line tells docker what commands to run for getting the application to work. Wherever there should be a space, we separate it into different blocks. Here we don't need to add the working directory because docker does it automatically.

> This Dockerfile will replace the very large command from the first part.

- We use `docker build` to create images from dockerfiles
- `t` to tag the image with a name
- and we use a dot `.` to say that we are currently in the same directory as the Dockerfile (we have to make sure we are)

The final command will look something like `docker build -t python-app .`

- Now that the image is built, we can run the container that has the app in it with `docker run <image name>`.

If we want to name the container or have it removed after it runs, we add the `--rm` and `--name` args.

So the final command will be `docker run --rm --name python-app-container python-app`

### Dockerignore
> A very useful way to create an image from a dockerfile, if we want to use all contents of a folder instead a single file, is to make sure the `COPY` command runs inside the current dir and copies everything in it to docker's working dir. For this to work, we are going to need a [`.dockerignore` file](course-content\02-dockerfiles\.dockerignore) so it will not copy the actual Dockerfile but everything else.

The Dockerfile will look like this:

```docker
FROM python:3.8-slim

WORKDIR /app

COPY . . # Copy everything from the current dir to docker's workdir

CMD ["python", "python-app.py"]
```

### Image Tags

If we don't tag images, the default tag is "latest". To tag an image we can add a `:` and the version after the image name when we build it, like `docker build -t python-app:0.0.1 .`

### Interactive shell inside a container

To understand what is happening inside our container, we can run it with an interactive shell that will allow us to move around inside its contents, by adding the arg `-it`, and the `/bin/sh` location.

We run `docker run -it --rm --name <container name> <image name> /bin/sh`

We type `exit` to leave the interactive mode.

## Creating complex images

### Running a flask app

To run a [flask app](course-content\03-flask-app), we will need to install dependencies by pip installing requirements during the image building stage, then we can either
- run the app directly with `python app.py` or
- we can set the `FLASK_APP` shell environment variable to the entry point of our web application and when calling the `flask run` command it knows to start running the flask app from the `app.py` file

#### For the first option, inside the Dockerfile we will have the following structure
```docker
## 1. What base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory
WORKDIR /app

## 3. -copy the project files into the working directory
COPY . .

## 4. Install dependencies
RUN pip install -r requirements.txt # during image building

## 5. Document and inform the developer that the application will use port 5000 of the container
EXPOSE 5000 

## 6. Define the command to run when the container starts
CMD ["python", "app.py"] # Will run after we create the container
```

> The `RUN` command executes during the **image building phase** and whatever is installed here will be installed **in** the image, instead `CMD` gives the container the commands it needs to **run the application**.

> The `EXPOSE` line won't actually expose the actual port, it will just let us know that the port should be exposed.

We navigate to the flask directory and there we build our image and run the containerized app: `docker build -t flask-demo .`

The container will be using the port 5000 but that is isolated from my local machine, so for those two to connect we are going to have to map our ports. So we run the container mapping a host machine port to the container's port with `docker run -p 5000:5000 --rm flask-demo` and open `localhost:5000` on the browser to see our app.

#### For the second way of running a flask app, we will set an environment shell variable

```docker
## 1. What base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory
WORKDIR /app

## 3. -copy the project files into the working directory
COPY . .

## 4. Install dependencies
RUN pip install -r requirements.txt # during image building

## 5. Document and inform the developer that the application will use port 5000 of the container
EXPOSE 5000 

## 6. Set the environment shell variable to the app
ENV FLASK_APP=app.py

## 7. Define the command to run when the container starts
CMD ["flask", "run", "--host=0.0.0.0"] # Will run after we create the container
```

The `--host=0.0.0.0` argument will allow the app to recieve requests from any available network interface.

From here, we run our app the same way as before.

> To remove every docker object we can run `docker system prune -a`