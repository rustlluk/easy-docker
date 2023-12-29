# Easy-Docker
This repository is meant to make working with Docker easier when one wants to use graphical applications. The main 
script [deploy.py](#Docker/deploy.py) does basically similar things as docker-compose, but it is easier to use and
in general works better for graphical applications, as users does not have to take care about setting all the variables.

# Basic Descriptions
  - **Works only on GNU/Linux systems!!**
    - and yeah, I really mean Linux. This does not work on Mac or Windows. However, the problem is not in kernel, but 
      in docker-engine vs docker-desktop
    - see [Other systems](#other-systems) for more details
  - What it can do?
    - provides base Dockerfile which can be extended or used as "virtual environment"
    - [deploy.py](Docker/deploy.py) make thins easier and allows to build and run images
      - it automatically makes graphical applications working properly, even over SSH connection
# How to work with this repository?
- install [docker-engine](https://docs.docker.com/engine/install/ubuntu/)
  - **DO NOT INSTALL DOCKER-DESKTOP!**
    - docker-desktop is not the same as docker-engine, and it does not work the same way 
  - do not forget about [post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/)
  - (optional) install [nvidia-docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
    for GPU support
  - clone it, and either:
    1) extend the Dockerfile (install your libraries etc.), build it and use it
    2) just build the Dockerfile and work inside as in virtual box/environment
       1) you can install whatever you want inside

# Description
When using Docker, the most easy command to build is  `docker build -t IMAGE_NAME .` and you can run it later with
`docker run IMAGE_NAME`. Sound easy? It is, but then than comes the fun. You want to see host PC files in the docker?
You need to attach the volume. You want to have correct permissions? You need to set them in the Dockerfile and in
the run command. And the list go on. A lot of things can be fixed with docker-compose. However, it is meant more
for 'easier' configuration, not for easier usage of Docker for  more people. And it is basically why I created this 
repository.

The main functionality is defined in [deploy.py](Docker/deploy.py).
- **The script needs to be called from inside the [Docker](Docker) folder**
  - The reason is that Docker looks for `Dockerfile` and other files in the current directory. One can use the `-f`
    parameter to specify the path to the Dockerfile, but it is easier to just call it from the right folder if you 
    want to also include files etc.

Basic usage:
  - build image with container named "test_container", mounting path
    "/home/user/test_data" to docker: 
   
        ./deploy.py -b -p /home/user/test_data -c test_container
  - open the container again after it was closed (without build, without setting environment variables etc.)
   
        ./deploy.py -e -c test_container
  - open the container after it was closed, delete all changes in it and set all environment variables again

        ./deploy.py -c test_container
    - this is useful when you some variable change (usually $DISPLAY when connected over SSH) 
  - open the container, delete changes and allow GPU usage
    
        ./deploy.py -c test_container -nv
    - this cannot be done without deleting changes done in the container
  - change the path that is visible to the container
    
        ./deploy.py -c test_container -p /home/user/new_test_data
    - this cannot be done without deleting changes done in the container
  - open new terminal with the same docker session
    
        ./deploy.py -c test_container -t
  - build image with ubuntu 18 and cuda 12
  
        ./deploy.py -b -c ubuntu_18 -p /home/user/test_data -bi nvidia/cuda:12.0.0-devel-ubuntu18.04

## Parameters of deploy.py
- `-b` or `--build` for building the Dockerfile
  - default: False
- `-nv` or `--nvidia` when you want to use your Nvidia card
  - you have to use it when creating a new container
  - you need [nvidia-docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
  - default: False
- `-e` if you just want to run existing docker without building, without reloading environment variables etc.
  - default: False
- `-p` or `--path` to specify path that will be visible to the container
  - default: ""
- `c` or `--container` with desired name of the new, created container
  - default: my_new_docker
- `t` or `--terminal` to run new terminal in running docker session
  - default: False
- `-pv` or `--python-version` to specify additional python version to install
  - each system comes with default python version (e.g., Ubuntu 20 with Python3.8). With this, you can install
    additional one (e.g., pyhon3.11) if you want to use the advantages of newer versions
  - this parameters work only for python 3.7 and higher! Otherwise, you need to change line 21 in 
    [Dockerfile](Docker/Dockerfile)
  - default: 3.11
- `-pcv` or `--pycharm-version` to specify version of pycharm to use
  - delete lines 38-42 in [Dockerfile](Docker/Dockerfile) if you do not want to install pycharm
  - default: 2023.2.3
- `-bi` or `--base-image` to specify base image that will be used
  - default: nvidia/cuda:11.0.3-devel-ubuntu20.04
  - other can be found at hub.docker.com


# FAQ
  - **applications do not run on your screen** (or you have some strange screen
    related errors)
    - try to run the container again without `-e` parameters -> your $DISPLAY variable most probably changed, and
      you need to regenerate permission
  - **you get error of not being in sudo group when running image**
    - this happens when you try to run the container with different use id (`id -u`) than the one you used to build
      the image
    - you need to rebuild the image for this user
      - just run with `-b` switch. The user will be set automatically
  - **`sudo apt install something` does not work**
    - you need to run `sudo apt update` first after you run the container for the first time
  - **You want to use it over SSH**
    - use `ssh -X` to connect to the desired computer
    - run `deploy.py` script there
    - the $DISPLAY variable changes a lot when using SSH, so you may need to run the container without `-e` switch to
      load the correct one

# Other systems 
This script will work only on GNU/Linux systems. The reason is that it requires docker-engine. However, 
on Mac and Windows, only docker-desktop is available. It works differently, mainly in how system variables are parsed
and set in the Docker.
To make it work on docker desktop, you will have to set some variables set in the end of [deploy.py](Docker/deploy.py)
(all volume connections to docker, environment variables etc.) by hand in docker-desktop environment. And then maybe... 
**However, be clever and use Linux for programming and robotics stuff.**
