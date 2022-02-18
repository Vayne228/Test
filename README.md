# Nordlys

## Kubernetes setup

Kubernetes is a container-orchestration system for automating computer application deployment, scaling, and management. It works with a variety of container runtimes such as Docker, Containerd, and CRI-O.

## Main functions

- Deploys to any environment that supports Kubernetes, locally or remotely, including environments like OpenShift.
- Supports running code written in any language, with some common runtimes prepackaged with the framework.
- Triggers the execution of code by many kinds of events—an HTTP endpoint, a queue message, or some other hook.

## Install Docker Engine

To install Docker Engine, you need the 64-bit version of one of these Ubuntu versions:

- Ubuntu Impish 21.10
- Ubuntu Hirsute 21.04
- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)

Docker Engine is supported on x86_64 (or amd64), armhf, arm64, and s390x architectures.

### Uninstall old versions

Older versions of Docker were called **docker**, **docker.io**, or **docker-engine**. If these are installed, uninstall them by entering the following commands in sequence:

```sh
sudo apt-get remove docker docker-engine docker.io containerd runc
```

It’s OK if **apt-get** reports that none of these packages are installed.

The contents of `/var/lib/docker/`, including images, containers, volumes, and networks, are preserved. If you do not need to save your existing data, and want to start with a clean installation, enter the following commands in sequence:

1. To uninstall the Docker Engine, CLI, and Containerd packages:

```sh
sudo apt-get purge docker-ce docker-ce-cli containerd.io
 ```
 
2. To delete all images, containers, and volumes:

```sh
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
 ```
 
## Installation methods

You can install Docker Engine in different ways, depending on your needs:

- Set up Docker’s repositories and install from them, for ease of installation and upgrade tasks.
- Download the DEB package and install it manually and manage upgrades completely manually. This is useful in situations such as installing Docker on air-gapped systems with no access to the internet.
- Use automated convenience scripts to install Docker.

### Install using the repository

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

#### Set up the repository

1. Update the **apt** package index and install packages to allow **apt** to use a repository over HTTPS:

```sh
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
 ```
 
2. Add Docker’s official GPG key:

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 ```

3. Use the following command to set up the stable repository. To add the nightly or test repository, add the word **nightly** or **test** (or both) after the word **stable** in the commands below.

```sh
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Install Docker Engine

1. Update the **apt** package index, and install the latest version of Docker Engine and containerd, or go to the next step to install a specific version:

```sh
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

> Got multiple Docker repositories?
If you have multiple Docker repositories enabled, installing or updating without specifying a version in the apt-get install or apt-get update command always installs the highest possible version, which may not be appropriate for your stability needs.

2. To install a specific version of Docker Engine, list the available versions in the repo, then select and install:

a. List the versions available in your repo:

```sh
apt-cache madison docker-ce
```

b. Install a specific version using the version string from the second column, for example, **5:18.09.1\~3-0\~ubuntu-xenial**.

```sh
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

3. Verify that Docker Engine is installed correctly by running the hello-world image.

```sh
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

Docker Engine is installed and running. The **docker** group is created but no users are added to it. You need to use **sudo** to run Docker commands. 

#### Upgrade Docker Engine

To upgrade Docker Engine, first run **sudo apt-get update**, then follow the [installation instructions](#set-up-the-repository), choosing the new version you want to install.

### Install from a package

If you cannot use Docker’s repository to install Docker Engine, you can download the **.deb** file for your release and install it manually. You need to download a new file each time you want to upgrade Docker.

1. Go to `https://download.docker.com/linux/ubuntu/dists/`, choose your Ubuntu version, then browse to `pool/stable/`, choose **amd64**, **armhf**, **arm64**, or **s390x**, and download the **.deb** file for the Docker Engine version you want to install.

> To install a nightly or test (pre-release) package, change the word stable in the above URL to nightly or test. Learn about nightly and test channels.

2. Install Docker Engine, changing the path below to the path where you downloaded the Docker package.

```sh
sudo dpkg -i /path/to/package.deb
```

The Docker daemon starts automatically.

3. Verify that Docker Engine is installed correctly by running the hello-world image.

```sh
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

Docker Engine is installed and running. The **docker** group is created but no users are added to it. You need to use **sudo** to run Docker commands. 

#### Upgrade Docker Engine

To upgrade Docker Engine, download the newer package file and repeat the [installation instructions](#set-up-the-repository), pointing to the new file.

## Launch a container with an application

First you need to start a container with an application in the Kubernetes cluster. For example, take ExampleApp, on port 8800 for accepting requests. 

### Create a structure

To create structure you need to create a directory and subdirectories. Run the following commands: 

```sh
mkdir quickstart_docker
mkdir quickstart_docker/application
mkdir quickstart_docker/docker
mkdir quickstart_docker/docker/application
```

You will get this structure:

quickstart_docker/ # Main project directory  
├──application/      #  Aplication sources   
└──docker/           # Docker files  
    └──docker/       # Put here Dockerfile for the application   

Everything will work correctly from one folder, but it is better to stick to the structure.

### Application deployment
To deploy an application, you need an example. Then you need to replace this with a real one. 

Make(Or edit if there is already one) the **application.py** file in the `quickstart_docker/application` with the following content: 

```
import http.server
import socketserver

PORT = 8800

Handler = http.server.SimpleHTTPRequestHandler

httpd = socketserver.TCPServer(("", PORT), Handler)

print("serving at port", PORT)
httpd.serve_forever()
```

### Connect dependencies 

The application needs an environment. At least, you need Python, and dependencies for it. All dependencies can be obtained from Docker Hub, take them from there.
Make (or edit if there is already one) the **Dockerfile** file in the `quickstart_docker/docker/application` with the following content:  

```
# Use base image from the registry
FROM python:3.5

# Set the working directory to /app
WORKDIR /app

# Copy the 'application' directory contents into the container at /app
COPY ./application /app

# Make port 8800 available to the world outside this container
EXPOSE 8800

# Execute 'python /app/application.py' when container launches
CMD ["python", "/app/application.py"]
```

### Make a build

The next step is to create a build. Run the following command:

```sh
docker build . -f docker/application/Dockerfile -t exampleapp
```

> Arguments:
. - working directory;  
-f docker/application/Dockerfile - docker-file;  
-t exampleapp - image tag to make it easier to find.  
Read more about [building images for Docker](https://docs.docker.com/engine/reference/builder/).

### Check the result

To make sure the image was created run the following command:

```sh
docker images
```

You will get result like this one:

```
| REPOSITORY | TAG    | IMAGE ID     | CREATED       | SIZE  |
|------------|--------|--------------|---------------|-------|
| exampleapp | latest | 83wse0edc28a | 2 seconds ago | 153MB |
| python     | 3.6    | 05sob8636w3f | 6 weeks ago   | 153MB |
```

Then, you need to push the image into the repository. Read about this in the next [article](#).
