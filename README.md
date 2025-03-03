# Rebble - Guide to Setting up Pebble Development Environment Using Official Pebble SDK Docker Image

01 Mar 2025 - Vy Phan

Ubuntu 24.04

## Introduction
First thing first, this is not a Rebble official guide. This is just my personal note on how to set up the Pebble development environment. I am pretty sure I will forget it all in a week, so to prevent my goldfish memory from taking the wheel, I wrote down what I can remember from start to when I finally got the emulator up and running.

Most of the installation parts are written from my memory. I did not repeat the installation to confirm all the steps. A mistake/typo or more will be found as someone else reads and follows through these steps. 

I am just a noob trying to dabble into Pebble development, so please keep in mind that you are following these steps at your own risk.

I included most of the sites I have read in the *References* section at the very bottom of this guide. I did not come up with all these commands myself. It's more of putting together what has worked for me. I'm on Ubuntu 24.04. Not sure if these steps will work on other OS.

Hope this will help if anyone wants to use the official Pebble SDK Docker image.

Happy hacking!


## Docker
### 1. Download Docker
Download Docker Desktop from the official site. 

Instructions can be found at: https://docs.docker.com/desktop/setup/install/linux/ubuntu/
### 2. Pull Pebble SDK Docker Image
Official site: https://hub.docker.com/r/rebble/pebble-sdk
In terminal, run the following command to pull the docker image from Docker Hub
```console
docker pull rebble/pebble-sdk
```
*Note: may need to run with `sudo` depending on how Docker is originally installed*
```console
sudo docker pull rebble/pebble-sdk
```

### Note: Other Useful Docker Commands to Manage Images
**List all downloaded docker images**
```console
docker images
```
Output may look like this:
```console
$ docker images                                                                 
REPOSITORY          TAG       IMAGE ID       CREATED       SIZE
hello-world         latest    74cc54e27dc4   5 weeks ago   10.1kB
rebble/pebble-sdk   latest    6336205d0ed9   4 years ago   2.47GB
```

**Delete a downloaded docker images**
```console
docker rmi <image_name>
```

## Export Display from Container to Host
Set the environment variable for the display (needed for emulator) by running the following command in host terminal
```console
export DISPLAY=:0
```

## X11 Forwarding
This setup is needed for forwarding the display of the emulator from the container to host machine.

### 1. Modify X11 Forwarding in SSH daemon configuration file

1. Open the SSH daemon configuration file.
```console
sudo vi /etc/ssh/sshd_config
```
*Note: can also use `vim` and `nano`*

2. Uncomment or modify the lines with `ForwardingX11`. This line allows the graphical application to be forwarded over to your computer.
```console
ForwardingX11 yes
```

3. Save, then exit

### 2. Install X11-apps
In host terminal, run the following command
```
sudo apt-get -y install x11-apps
```
### 3. Enable X11 Forwarding
Before starting Docker container, allow X11 Forwarding by running the following command on host machine:
```console
xhost +
```

## Run the Pebble SDK Docker Container
### First Option - No Sharing Directories between Host and Container
To run the Pebble SDK Docker container, run the following command in the host terminal
```console
docker run --name=RebbleSDK -it \
    -e DISPLAY=$DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v ~/.Xauthority:/home/pebble/.Xauthority \
    rebble/pebble-sdk
```
After running the above command, Docker will open an interactive shell. Looks like this:
```console
root@2aef30c81e77:/opt/pebble-sdk-4.5-linux64# 
```
*Note: again, this `docker run` command will not link the project folder on host machine with the project folder in Docker container. If source code is on the host machine, it will need to be transferred to Docker container. See guides in "Copy files/folders between a container and the local file system" below.*

### Second Option - The Container Will Be Able to Access a Source Code Directory on Host Machine
The `docker run` function before has an extra flag `-v /HOST/PATH:/CONTAINER/PATH`. This flag allows host and container to share a directory on host machine. This is useful for development on host machine and using container to build and run emulator.
```console
docker run --name=RebbleSDK -it \
    -e DISPLAY=$DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v ~/.Xauthority:/home/pebble/.Xauthority \
    -v /HOST/PATH:/CONTAINER/PATH \
    rebble/pebble-sdk
```
`-v` option for basic volumn and mount operations. If the directory is not available on host machine, it will create a new one.

`/HOST/PATH:/CONTAINER/PATH` the first part before `:` is the path to the directory to be shared with the container. The second part is the path where the shared directory can be found inside the container.


### Note: Other Useful Docker Commands to Manage Docker Containers
**List all Docker Containers**
```console
docker ps -a
```
`-a` this option will list all the containers (running or not). It may looks like this:
```console
CONTAINER ID   IMAGE              COMMAND  CREATED  STATUS  PORTS  NAMES
2aef30c81e77   rebble/pebble-sdk  ...      ...      ...     ...    RebbleSDK
```

**Start a Docker Container**
```console
docker start <container_name>
```

**Stop a Docker Container**
```console
docker stop <container_name>
```
After stopping a container, all data in the container will be deleted. To start the container again, use the above `docker start` command. However, the container will start as new.

I usually just remove the container with the `docker rm` command below and re-run the `docker run` command after I exit it.

**Remove a Docker Container**
```console
docker rm <container_name>
```

## Create and Build a Sample Pebble Project
Official Pebble Tool Guide: https://developer.rebble.io/developer.pebble.com/guides/tools-and-resources/pebble-tool/index.html

### 1. Create a Sample Pebble Project
In the Docker interactive shell, run the following command to create a sample Pebble project
```console
pebble new-project my_rebble_project
```
Output may look like this:
```console
root@2aef30c81e77:/opt/pebble-sdk-4.5-linux64# pebble new-project my_rebble_project
Created new project my_rebble_project
root@2aef30c81e77:/opt/pebble-sdk-4.5-linux64# ls -la
total 36
drwxrwxrwx 1  501 staff 4096 Mar  2 05:36 .
drwxr-xr-x 1 root root  4096 Oct  5  2020 ..
drwxrwxrwx 1 root root  4096 Oct  5  2020 .env
drwxrwxrwx 1  501 staff 4096 Oct  5  2020 arm-cs-tools
drwxrwxrwx 1  501 staff 4096 Oct  5  2020 bin
drwxrwxrwx 1  501 staff 4096 Oct  5  2020 pebble-tool
-rwxrwxrwx 1  501 staff  371 Oct 12  2016 requirements.txt
drwxr-xr-x 3 root root  4096 Mar  2 05:36 my_rebble_project
```
The code of the project will be in the `my_rebble_project` folder. See Pebble Development Guide for more tutorials.

### Build a Pebble Project
Navigate into the project folder. For example above, `my_rebble_project`
```console
cd my_rebble_project
```
Build command must be run in the project folder
```console
pebble build
```
Output will be a `build` folder in the project folder:
```console
root@2aef30c81e77:/opt/pebble-sdk-4.5-linux64/sample# ls
build  package.json  src  wscript

```

### 2. Run the Emulator
Running the following command inside the project folder should trigger the QEMU emulator display to open

```console
pebble install --emulator <platform>
```
`<platform>` can be `aplite`, `basalt`, `chalk`, `diorite`, `emery`

Please check Pebble Hardware Information page for more details on the platforms: https://developer.rebble.io/developer.pebble.com/guides/tools-and-resources/hardware-information/index.html

Output may be like this:

```
root@2aef30c81e77:/opt/pebble-sdk-4.5-linux64/sample# pebble install --emulator basalt
Installing app...
App install succeeded.
```
Emulator can be closed and re-open with the above command.

### Note: Useful Commands to Clean Up Build
Perform the following command in the project folder
```
pebble clean
```
Run the `build` command above to rebuild a project when there is a code change.

## Copy files/folders between a container and the local file system

**Note: `CONTAINER` is the container's name**

**Note 2: Run the following commands on host machine**

### Copy a local file into container
```console
 docker cp ./some_file CONTAINER:/path/to/destination/folder/on/docker/container/
```

### Copy files from container to local path

```console
 docker cp CONTAINER:/path/to/destination/folder/on/docker/container/some_file ~/some_file
```

### Example: copy code from host to Pebble SDK Docker container

`./my_code.c` Host-machine path to the file to be transferred

`RebbleSDK` Container name (find this by running `docker ps -a`)

`/opt/pebble-sdk-4.5-linux64/my_pebble_project/src/c/` Docker-container path to the destination folder

``` console
docker cp ./my_code.c RebbleSDK:/opt/pebble-sdk-4.5-linux64/my_pebble_project/src/c/
```
Output may look like this:
``` console
docker cp ./my_code.c RebbleSDK:/opt/pebble-sdk-4.5-linux64/my_pebble_project/src/c/

Successfully copied 2.56kB to RebbleSDK:/opt/pebble-sdk-4.5-linux64/my_pebble_project/src/c/my_code.c
```

## Exit
#### Exit Docker Container
In the docker container interactive shell, run the following command
```console
exit
```
**WARNING: if you exit the interactive shell, this will stop the docker container as well. That means all the data populate from the point of container starting and exiting will be gone.**

**REMEMBER TO TRANSFER ANY FILES BEFORE EXITING**


## References
Docker Basic Commands Cheat Sheet: 
- https://docs.docker.com/get-started/docker_cheatsheet.pdf

Docker Setup for Pebble Development:
- https://github.com/pebble-dev/rebble-docker
- https://github.com/andyburris/pebble-setup
- https://github.com/bboehmke/docker-pebble-dev

X11 Forwarding:
- https://medium.com/@paliwalsamriddhi/gui-apps-within-a-docker-container-971681838fda
- https://tutorials.rc.nectar.org.au/x11forwarding/02-enable-x11-on-virtual-machine
- https://goteleport.com/blog/x11-forwarding/

Pebble Tool: 
- https://developer.rebble.io/developer.pebble.com/guides/tools-and-resources/pebble-tool/index.html

Pebble Hardware:
- https://developer.rebble.io/developer.pebble.com/guides/tools-and-resources/hardware-information/index.html
