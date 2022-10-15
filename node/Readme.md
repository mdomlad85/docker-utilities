# Node Util

## Run npm command
docker-compose run --rm npm [any npm command]

## Linux issue

### The problem
In Linux, by default Docker runs as the "Root" user, so when we do a 
lot of the things that you are advocating for with Utility Containers 
the files that get written to the Bind Mount have ownership and 
permissions of the Linux Root user.  (On MacOS and Windows10, since 
Docker is being used from within a VM, the user mappings all happen 
automatically due to NFS mounts.)

### The solutions

#### Use  predefined "node" user (if you're lucky)

There is a lot of discussion out there in the docker community (devops) about security around running Docker as 
a non-privileged user.  
The Official Node.js Docker Container provides such a user that they call "node".

https://github.com/nodejs/docker-node/blob/master/Dockerfile-slim.template

FROM debian:name-slim

RUN groupadd --gid 1000 node \
&& useradd --uid 1000 --gid node --shell /bin/bash --create-home node

Luckily enough for me on my local Linux system, my "mitz" uid:gid 
is also 1000:1000 so, this happens to map nicely to the "node" user 
defined within the Official Node Docker Image.

So, in my case of using the Official Node Docker Container, all I need to do is make sure I specify that I want the container to run as a non-Root user that they make available.  To do that, I just add:

Dockerfile 
-----

FROM node:14-slim
USER node
WORKDIR /app

--------

If I rebuild my Utility Container in the normal way and re-run 
"npm init", the ownership of the package.json file is written as 
if "scott" wrote the file.

$ ls -la

total 12

drwxr-xr-x  2 mitz mitz 4096 Oct 31 16:23 ./

drwxr-xr-x 13 mitz mitz 4096 Oct 31 16:23 ../

-rw-r--r--  1 mitz mitz 204 Oct 31 16:23 package.json

#### Remove the predefined "node" user and add yourself as the user

However, if the Linux user that you are running as is not lucky to be mapped to 1000:1000, then you can modify the Utility Container Dockerfile to remove the predefined "node" user and add yourself as the user that the container will run as:

-------

FROM node:14-slim

RUN userdel -r node

ARG USER_ID

ARG GROUP_ID

RUN addgroup --gid $GROUP_ID user

RUN adduser --disabled-password --gecos '' --uid $USER_ID --gid $GROUP_ID user

USER user

WORKDIR /app

-------

And then build the Docker image using the following (which also gives you a nice use of ARG):

$ docker build -t node-util:cliuser --build-arg USER_ID=$(id -u) --build-arg GROUP_ID=$(id -g) .



And finally running it with:

$ docker run -it --rm -v $(pwd):/app node-util:cliuser npm init

$ ls -la 

total 12

drwxr-xr-x  2 mitz mitz 4096 Oct 31 16:54 ./

drwxr-xr-x 13 mitz mitz 4096 Oct 31 16:23 ../

-rw-r--r--  1 mitz mitz  202 Oct 31 16:54 package.json

Reference to Solution 2 above: https://vsupalov.com/docker-shared-permissions/